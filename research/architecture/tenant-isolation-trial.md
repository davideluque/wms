# Native PostgreSQL merchant-isolation trial

On 26 November 2026, a disposable native PostgreSQL 16.15 cluster ran **12 passing assertions**. Two merchants had orders with the identical external ID `412` and location code `MAIN`. The ordinary application login saw five bags under merchant A and eleven under B. Reusing the same backend after a rollback leaked neither the previous context nor the rolled-back write.

## Observed results

| Case | Observed |
| --- | --- |
| Fresh runtime login with no merchant set | Zero visible receipts. An attempted insert for A returned SQLSTATE `42501`. |
| A context, including a B write | Receipt 412 contained only A’s five bags. An insert specifying B returned `42501`. |
| A receipt using a B-only location | A’s row passed its tenant policy, then the composite foreign key rejected the parent reference with `23503`. |
| B transaction rolls back, then the connection is reused | B saw eleven bags, inserted a synthetic receipt and rolled back. The same backend saw zero rows without context and only A’s original receipt after setting A again. A privileged count at the end verified two original rows totaling sixteen bags. |
| Effective table owner before and after `FORCE ROW LEVEL SECURITY` | With missing context, the owner saw two receipts before FORCE and zero after. A distinct test role with `BYPASSRLS` and read-only table grants still saw both afterward. |

The database binary, client and libpq came from Ubuntu’s arm64 PostgreSQL packages, version `16.15-0ubuntu0.24.04.1`, extracted in a private temporary directory. The cluster accepted only a Unix socket inside that directory. Database and role creation occurred in the disposable cluster. No repository dependencies were added. [PostgreSQL 16 row security](https://www.postgresql.org/docs/16/ddl-rowsecurity.html) · [Transaction-local settings](https://www.postgresql.org/docs/16/sql-set.html) · [Composite foreign keys](https://www.postgresql.org/docs/16/ddl-constraints.html)

## Repeat on a disposable cluster

Save the SQL block below as `trial.sql`. On a Unix machine with compatible native `initdb`, `pg_ctl` and `psql` on the path, run as an unprivileged OS user. This example creates fresh data and socket directories. The selected port identifies the private socket. The temporary cluster trusts local clients able to reach that socket, so keep its parent private.

```sh
set -eu
trial_dir=$(mktemp -d)
chmod 700 "$trial_dir"
mkdir "$trial_dir/socket"
initdb -D "$trial_dir/data" -U trial_bootstrap --no-locale --encoding=UTF8 \
  --auth-local=trust --auth-host=reject
pg_ctl -D "$trial_dir/data" -l "$trial_dir/server.log" \
  -o "-c listen_addresses='' -c unix_socket_directories='$trial_dir/socket' -p 54832" -w start
trap 'pg_ctl -D "$trial_dir/data" -m fast -w stop && rm -rf -- "$trial_dir"' EXIT
psql -X -h "$trial_dir/socket" -p 54832 -U trial_bootstrap -d postgres -f trial.sql
```

Each expected database error is caught by its SQL condition. An unexpected success or a failed assertion stops `psql` with a nonzero exit.

```sql
\set ON_ERROR_STOP on
SELECT current_user AS bootstrap_role \gset
SELECT version();
CREATE ROLE trial_owner NOLOGIN;
CREATE ROLE trial_app LOGIN NOSUPERUSER NOBYPASSRLS;
CREATE ROLE trial_auditor NOLOGIN BYPASSRLS;
CREATE SCHEMA trial AUTHORIZATION trial_owner;
SET ROLE trial_owner;
CREATE TABLE trial.location (
  merchant text NOT NULL, code text NOT NULL,
  PRIMARY KEY (merchant, code)
);
CREATE TABLE trial.receipt (
  merchant text NOT NULL, external_id text NOT NULL,
  location text NOT NULL, bags integer NOT NULL CHECK (bags > 0),
  PRIMARY KEY (merchant, external_id),
  FOREIGN KEY (merchant, location) REFERENCES trial.location (merchant, code)
);
INSERT INTO trial.location VALUES ('A','MAIN'), ('B','MAIN'), ('B','B-ONLY');
INSERT INTO trial.receipt VALUES ('A','412','MAIN',5), ('B','412','MAIN',11);
ALTER TABLE trial.location ENABLE ROW LEVEL SECURITY;
ALTER TABLE trial.receipt ENABLE ROW LEVEL SECURITY;
CREATE POLICY merchant_rows ON trial.location
  USING (merchant = nullif(current_setting('trial.merchant', true), ''))
  WITH CHECK (merchant = nullif(current_setting('trial.merchant', true), ''));
CREATE POLICY merchant_rows ON trial.receipt
  USING (merchant = nullif(current_setting('trial.merchant', true), ''))
  WITH CHECK (merchant = nullif(current_setting('trial.merchant', true), ''));
CREATE FUNCTION trial.expect(ok boolean, label text) RETURNS void
LANGUAGE plpgsql AS $$
BEGIN
  IF ok IS DISTINCT FROM true THEN RAISE EXCEPTION 'FAIL %', label; END IF;
  RAISE NOTICE 'PASS %', label;
END $$;
GRANT USAGE ON SCHEMA trial TO trial_app, trial_auditor;
GRANT SELECT ON ALL TABLES IN SCHEMA trial TO trial_app, trial_auditor;
GRANT INSERT ON trial.receipt TO trial_app;
RESET ROLE;

\connect - trial_app
SELECT pg_backend_pid() AS backend \gset
SELECT trial.expect(current_user = 'trial_app' AND session_user = current_user,
                    'ordinary runtime login');
SELECT trial.expect((SELECT count(*) FROM trial.receipt) = 0, 'fresh context sees zero');
DO $$
BEGIN
  BEGIN
    INSERT INTO trial.receipt VALUES ('A','NO-CONTEXT','MAIN',1);
    RAISE EXCEPTION 'unexpected insert';
  EXCEPTION WHEN insufficient_privilege THEN RAISE NOTICE 'PASS missing-context write 42501';
  END;
END $$;
BEGIN;
SET LOCAL trial.merchant = 'A';
SELECT trial.expect((SELECT count(*) = 1 AND min(bags) = 5 FROM trial.receipt
                     WHERE external_id = '412'), 'A sees only its five bags');
DO $$
BEGIN
  BEGIN
    INSERT INTO trial.receipt VALUES ('B','WRONG-TENANT','MAIN',1);
    RAISE EXCEPTION 'unexpected insert';
  EXCEPTION WHEN insufficient_privilege THEN RAISE NOTICE 'PASS wrong-tenant write 42501';
  END;
  BEGIN
    INSERT INTO trial.receipt VALUES ('A','BAD-PARENT','B-ONLY',1);
    RAISE EXCEPTION 'unexpected insert';
  EXCEPTION WHEN foreign_key_violation THEN RAISE NOTICE 'PASS composite foreign key 23503';
  END;
END $$;
COMMIT;
BEGIN;
SET LOCAL trial.merchant = 'B';
SELECT trial.expect((SELECT count(*) = 1 AND min(bags) = 11 FROM trial.receipt
                     WHERE external_id = '412'), 'B sees only its eleven bags');
INSERT INTO trial.receipt VALUES ('B','ROLLED-BACK','MAIN',2);
ROLLBACK;
SELECT trial.expect(pg_backend_pid() = :backend AND
                    (SELECT count(*) FROM trial.receipt) = 0,
                    'same connection after rollback sees zero without context');
BEGIN;
SET LOCAL trial.merchant = 'A';
SELECT trial.expect((SELECT count(*) = 1 AND min(bags) = 5 FROM trial.receipt),
                    'same connection reused for A');
COMMIT;

\connect - :bootstrap_role
SET ROLE trial_owner;
SELECT trial.expect((SELECT count(*) FROM trial.receipt) = 2, 'owner sees two before FORCE');
ALTER TABLE trial.receipt FORCE ROW LEVEL SECURITY;
SELECT trial.expect((SELECT count(*) FROM trial.receipt) = 0, 'owner sees zero after FORCE');
RESET ROLE;
SET ROLE trial_auditor;
SELECT trial.expect((SELECT count(*) = 2 AND sum(bags) = 16 FROM trial.receipt),
                    'read-only BYPASSRLS role sees both original rows after FORCE');
RESET ROLE;
```

## What this establishes

PostgreSQL enforced these two tables’ policies and foreign key for the tested login, isolation context and actions. The trial reused one actual backend, a deterministic stand-in for sequential checkout from a pool of one. The setting is supplied by trusted application code. Authentication, merchant selection and site permissions still require independent application tests. The test did not exercise PgBouncer, the WMS HTTP layer, a job queue, object storage, privileged functions, or customer hardware. Re-run it with the supported database major, actual pool mode, migrations and deployment roles before a 2027 rollout. [Proposed tenant architecture](tenant-isolation.md) · [Separate two-connection stock experiment](race-conditions.md)
