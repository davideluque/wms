# Catching up inventory while the warehouse keeps working

**A page cursor tells the client where to read next. A snapshot also fixes which version it will read.** For the proposed [merchant API](merchant-api.md), issue a snapshot token and its corresponding change cursor from one consistent database view. All pages under that token must describe the same stock position. After installing the snapshot, the client can apply later stock revisions and explicit removals.

## Observed on native SQLite

On 26 November 2026, Python 3.12.3 and SQLite 3.45.1 ran a local trial on three connections with WAL enabled. The fixture represents one merchant, one site and quantities in bags. Letters are immutable stock-record IDs ordered alphabetically. At cursor 4, the first page read B=2 and D=4. A writer then committed five changes together: insert A=1, update D=40, remove F, insert G=7 and update H=80. [SQLite snapshot isolation](https://sqlite.org/isolation.html)

| Paging method, two records per page | Observed enumeration | Consequence |
| --- | --- | --- |
| Live offset | B=2, D=4, D=40, G=7, H=80 | D appeared twice. A sorted before the offset and was missed. |
| Live keyset using immutable ID | B=2, D=4, G=7, H=80 | No duplicate. A was still missed, D remained stale and H came from a newer state. |
| Keyset within one read snapshot | B=2, D=4, F=6, H=8 | All four original rows and their original quantities were preserved at cursor 4. |

The client then replayed events **5 through 9** onto the snapshot. It repeated the first change page and processed F’s deletion marker. During catch-up the writer changed D again, to 41 at event 10. The client first matched the saved state at 9, including D=40 and F absent. The next round applied event 10 and matched the live ledger exactly: **A=1, B=2, D=41, G=7, H=80**.

## Proposed pilot contract

- Bind the snapshot token to its authorized merchant, site/filter, ordering, revision and expiry. Use immutable stock-position keys that distinguish variant, location and other tracked dimensions. Resolve access again on resume. A renamed SKU keeps its key. A removed position has a tombstone containing its prior key. Zero quantity can remain a valid position.
- Page a materialized export or a versioned view as of the snapshot revision. A maximum ID captured at the start bounds inserts beyond that ID, but cannot preserve older values or deleted rows. The trial held a read transaction open briefly. A long-lived browser download needs a bounded server-side snapshot mechanism and expiry.
- For each catch-up round, pin a visible upper position and return ordered changes strictly after the saved cursor through that position. A position must cover every committed change up to that point. A PostgreSQL sequence value allocated inside an uncommitted transaction alone cannot guarantee this ordering. A client that commits a cursor past a late-committing event could permanently miss it. [PostgreSQL sequence behavior](https://www.postgresql.org/docs/18/functions-sequence.html)
- Stage and verify every snapshot page before atomically replacing the matching local scope. Apply each change page and advance the local checkpoint in one durable transaction. Retain deletion or filter-exit events until the cursor expiry policy permits retirement. On expiry, rebuild from a fresh snapshot with its own cursor. Verify a merchant-scoped export and independently changing sites against the production implementation.

<details>
<summary>Repeat the local trial with Python’s standard library</summary>

Save as `trial.py` and run `python3 trial.py`. The trial uses a fresh temporary database, a stable reader, a live reader and a writer whose commits occur between pages. It checks the results above with assertions. Its in-memory consumer and SQLite’s serialized writes leave crash recovery, token expiry, tenant authorization and PostgreSQL commit ordering for separate trials.

```python
import sqlite3
from pathlib import Path
from tempfile import TemporaryDirectory

def run(path):
    writer, stable, live = [sqlite3.connect(path, isolation_level=None) for _ in range(3)]
    writer.execute("PRAGMA journal_mode=WAL")
    writer.executescript("""
      CREATE TABLE stock(id TEXT PRIMARY KEY, bags INTEGER NOT NULL);
      CREATE TABLE changes(seq INTEGER PRIMARY KEY, id TEXT NOT NULL, bags INTEGER);
    """)
    def change(rows):
        with writer:
            writer.execute("BEGIN IMMEDIATE")
            for key, bags in rows:
                if bags is None:
                    writer.execute("DELETE FROM stock WHERE id=?", (key,))
                else:
                    writer.execute("""INSERT INTO stock VALUES (?,?) ON CONFLICT(id)
                                      DO UPDATE SET bags=excluded.bags""", (key, bags))
                writer.execute("INSERT INTO changes(id,bags) VALUES (?,?)", (key, bags))

    def page(db, after=""):
        return db.execute("SELECT id,bags FROM stock WHERE id>? ORDER BY id LIMIT 2",
                          (after,)).fetchall()

    def log_page(after, upper):
        return live.execute("""SELECT seq,id,bags FROM changes WHERE seq>? AND seq<=?
                               ORDER BY seq LIMIT 2""", (after, upper)).fetchall()

    def watermark(db):
        return db.execute("SELECT max(seq) FROM changes").fetchone()[0]

    opening = [("B",2), ("D",4), ("F",6), ("H",8)]
    change(opening)
    stable.execute("BEGIN")
    start = watermark(stable)  # Establish this reader's snapshot and its change cursor.
    first = page(stable)
    assert start == 4 and first == page(live) == opening[:2]
    change([("A",1), ("D",40), ("F",None), ("G",7), ("H",80)])
    offset = first.copy()
    for n in (2, 4):
        offset += live.execute("SELECT id,bags FROM stock ORDER BY id LIMIT 2 OFFSET ?",
                               (n,)).fetchall()
    keyset = first + page(live, "D")
    snapshot = first + page(stable, "D")
    stable.execute("COMMIT")
    assert offset == [("B",2), ("D",4), ("D",40), ("G",7), ("H",80)]
    assert keyset == [("B",2), ("D",4), ("G",7), ("H",80)]
    assert snapshot == opening

    local = dict(snapshot)
    cursor, upper = start, watermark(live)
    assert upper == 9
    expected_at_upper = dict(live.execute("SELECT id,bags FROM stock"))
    def apply(batch):
        nonlocal cursor
        for seq, key, bags in batch:
            if seq <= cursor:
                continue
            if bags is None:
                local.pop(key, None)
            else:
                local[key] = bags
            cursor = seq
    first_changes = log_page(cursor, upper)
    apply(first_changes)
    apply(first_changes)  # Replay a delivered page.
    change([("D",41)])    # Arrives while the consumer is still reading through 9.
    while batch := log_page(cursor, upper):
        apply(batch)
    assert cursor == upper and local == expected_at_upper and "F" not in local
    apply(log_page(cursor, watermark(live)))
    assert cursor == 10 and local == dict(live.execute("SELECT id,bags FROM stock"))
    print(f"start={start}, upper={upper}, final={cursor}, stock={sorted(local.items())}")
    for db in (live, stable, writer):
        db.close()

with TemporaryDirectory() as directory:
    run(Path(directory) / "trial.db")
```

</details>

## Native PostgreSQL trial — can cursor 2 hide receipt 1?

**Yes.** On 26 November 2026 two application connections on PostgreSQL 16.15 wrote separate receipts for merchant A at Read Committed. R1 contained five bags and R2 seven. Each transaction wrote its receipt and event together. A separate consumer read only committed events. Both designs finished with two physical receipt records totaling twelve bags.

| Revision source | What the consumer saw while R1 was open | What happened after R1 committed |
| --- | --- | --- |
| PostgreSQL sequence | R1 had allocated 1. R2 allocated 2 and committed first. The reader saw only `2:R2` and checkpointed 2. | The reader’s `rev > 2` query returned no rows. A full reread showed `1:R1, 2:R2`. Incremental delivery accounted for seven of twelve bags. |
| Transactionally updated row per merchant | R1 updated A’s head from 0 to 1. R2 waited on R1’s row lock. An independent B transaction issued and committed its own revision 1 for nine bags. B’s reader saw it immediately. A’s reader saw no events and its committed head remained 0. | R1 committed, R2 resumed and committed revision 2. Starting from 0 returned both A events in order. The consumer accounted for all twelve bags. |

The harness released R1 only after confirming that R2 had committed in the sequence case or that `pg_blocking_pids` identified R1 as R2’s blocker in the per-merchant case. There was no timing assumption about which writer would finish first. [PostgreSQL sequence behavior](https://www.postgresql.org/docs/16/functions-sequence.html) · [Row locks and transaction lifetimes](https://www.postgresql.org/docs/16/explicit-locking.html)

**Contract implication.** Keep the revision row update, business write and event insert in the same transaction. Every publisher for that merchant stream must participate. Use the committed head and matching snapshot view for an upper bound. Serialize writers only within the ordering scope promised to consumers. The trial shows B made progress during this specific A lock. It did not measure throughput, hot-merchant wait times, parallel tenant migration or recovery from an expired cursor. A multi-merchant transaction needs a consistent lock order if it updates multiple heads.

<details>
<summary>Repeat the controlled schedules in a fresh PostgreSQL database</summary>

Use the [private disposable cluster procedure](../architecture/tenant-isolation-trial.md). Start with a fresh database and run this setup under its bootstrap role. Open the writer and reader sessions as `trial_worker`.

```sql
CREATE SEQUENCE allocated;
CREATE TABLE heads (m text PRIMARY KEY, rev integer NOT NULL);
INSERT INTO heads VALUES ('A',0),('B',0);
CREATE TABLE receipts (mode text, m text, k text, bags integer, PRIMARY KEY(mode,m,k));
CREATE TABLE events (mode text, m text, rev bigint, k text, PRIMARY KEY(mode,m,rev));
CREATE FUNCTION post(p_mode text, merchant text, receipt text, qty integer) RETURNS bigint
LANGUAGE plpgsql AS $$
DECLARE n bigint;
BEGIN
  IF p_mode = 'seq' THEN
    n := nextval('allocated');
  ELSIF p_mode = 'row' THEN
    UPDATE heads SET rev=rev+1 WHERE m=merchant RETURNING rev INTO STRICT n;
  ELSE
    RAISE EXCEPTION 'invalid mode';
  END IF;
  INSERT INTO receipts VALUES (p_mode,merchant,receipt,qty);
  INSERT INTO events VALUES (p_mode,merchant,n,receipt);
  RETURN n;
END $$;
CREATE ROLE trial_worker LOGIN;
GRANT SELECT,INSERT ON receipts,events TO trial_worker;
GRANT SELECT,UPDATE ON heads TO trial_worker;
GRANT USAGE ON SEQUENCE allocated TO trial_worker;
```

1. In session 1 begin a Read Committed transaction, run `SELECT post('seq','A','R1',5)` and leave it open. In session 2 run `SELECT post('seq','A','R2',7)` with autocommit on. In a fresh reader statement fetch `SELECT rev,k FROM events WHERE mode='seq' AND m='A' AND rev>0 ORDER BY rev`. Save the last visible revision (2). Commit session 1 and repeat the read with `rev>2`.
2. In session 1 begin a fresh transaction and run `SELECT post('row','A','R1',5)`. Record its backend PID. In session 2 issue the equivalent R2 posting with mode `row` and quantity 7. From a separate observer confirm the first PID appears in `pg_blocking_pids(second_pid)`. While session 2 is waiting, run `SELECT post('row','B','R1',9)` in a third autocommit session. Check B’s event, A’s empty feed, and A’s committed head of 0. Commit session 1, wait for session 2 to finish, then read A’s feed with `mode='row'` and `rev>0`.
3. For both modes compare the incremental result with `SELECT mode,count(*),sum(bags) FROM receipts WHERE m='A' GROUP BY mode` and a full event enumeration. Run this only on an isolated fixture. The worker grants exercise transaction behavior. Merchant authentication and RLS are covered by the separate isolation trial.

</details>
