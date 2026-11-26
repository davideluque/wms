# TypeScript server and data tools for a 2027 WMS

**Trial Fastify, JSON Schema and node-postgres first** if the team can review SQL comfortably. This combination exposes the exact transaction that changes stock. **Compare Hono, Zod and Drizzle** if maintaining TypeScript definitions across a growing catalog and many endpoints is more costly. Both can run on Node.js and use PostgreSQL through the same driver, so the comparison can hold the database schema and business contract constant. These are implementation candidates. No benchmark has been run.

## What each combination buys

| Candidate | HTTP validation and OpenAPI | Data access and maintenance tradeoff |
| --- | --- | --- |
| Fastify + JSON Schema/Ajv + node-postgres (`pg`) | Declare request and response schemas on the route and use `@fastify/swagger` to generate OpenAPI. Test Fastify’s body coercion settings explicitly. Converting a numeric input into a string can defeat a decimal-string contract. [Fastify validation and serialization](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/) · [Fastify Swagger plugin](https://github.com/fastify/fastify-swagger) | Parameterized SQL and explicit transaction control make locks, predicates and rollback easy to inspect. The team owns query-result mapping, migration SQL and consistency between TypeScript and route schemas. Use the same checked-out client throughout a transaction and release it on every exit path. [node-postgres transactions](https://node-postgres.com/features/transactions) |
| Hono + Zod OpenAPI + Drizzle over `pg` | Hono’s Node adapter serves routes. `@hono/zod-openapi` uses Zod schemas to validate inputs and describe operations. Document runtime refinements and business errors that a generated schema cannot express. [Hono Node adapter](https://hono.dev/docs/getting-started/nodejs) · [Hono Zod OpenAPI example](https://hono.dev/examples/zod-openapi) | Drizzle supplies typed queries and a transaction callback with isolation settings. Its SQL template allows a guarded update to remain explicit. Inspect generated SQL and confirm every write uses the callback’s transaction handle. The Node adapter, validation bridge, ORM and migration tooling each introduce a compatibility check. [Drizzle PostgreSQL drivers](https://orm.drizzle.team/docs/get-started-postgresql) · [Drizzle transactions](https://orm.drizzle.team/docs/transactions) · [Drizzle SQL templates](https://orm.drizzle.team/docs/sql) |

## Database contract for both candidates

For the comparison, a receipt line identifies an authenticated merchant, command, item, lot if tracked, location, unit and exact quantity. Accept positive finite decimal text with an item-specific maximum scale. Store whole items as integral counts and measured quantities as exact decimals or scaled integers. Keep two crates and their 1.250 kg net weight as separate observations. Validate precision before casting to a constrained `numeric` column, since PostgreSQL can round on assignment. Return the documented string representation through both clients. [PostgreSQL numeric types](https://www.postgresql.org/docs/18/datatype-numeric.html)

For a client submission, use non-null fields and `UNIQUE (merchant_id, installation_id, operation_type, client_key)`. A manual entry uses the merchant’s internal source namespace. Authorize the current caller for the site and action before returning a stored outcome. Keep external order identity unique by merchant, installation and external order ID. Give a posted movement its own WMS-generated ID unique within the merchant. Scope references to a merchant with composite foreign keys. Protect every balance with a unique stock identity including location, variant, lot and disposition where applicable. If lot is optional, define how a missing lot participates in uniqueness, for example with `NULLS NOT DISTINCT`. Apply non-null and row-local check constraints to eligible and reserved quantities under the agreed stock policy. Cross-row invariants need transaction logic. [PostgreSQL constraints](https://www.postgresql.org/docs/18/ddl-constraints.html)

In one receipt transaction claim the command, record its normalized input and result, write the movement, update the balance and append any required outbox row. A repeat with identical input returns the original movement. A changed payload using the same key conflicts. A single-balance reservation can use the following parameterized statement inside the transaction, after checking quantity and eligibility. This example assumes that placing units on hold removes them from the eligible bucket under the same balance lock. If eligibility depends on a separate lot record, hold, reserve and dispatch must lock that record in a consistent order and recheck its disposition in the posting transaction. A zero-row update requires a domain outcome before reporting success.

```sql
UPDATE stock_balance
SET reserved = reserved + $3
WHERE merchant_id = $1 AND balance_id = $2
  AND on_hand - reserved >= $3
RETURNING reserved;
```

For multi-row business rules that use serializable isolation, retry the entire transaction after a serialization failure and preserve the caller’s command ID. Keep external delivery outside that transaction. [PostgreSQL transaction isolation](https://www.postgresql.org/docs/18/transaction-iso.html)

## A comparative implementation trial

Build the same two operations, `POST /receipts` and `POST /reservations`, in disposable branches. Feed both the same warehouse fixtures, PostgreSQL version, indexes, pool size, Node version and offered workload. Use the same OpenAPI dialect supported by both generators and the target merchant client. Publish an artifact locally, validate it, then generate a client that imports neither candidate’s server code. Check decimal strings, required fields, command identity, error responses and a replay from an earlier client. [OpenAPI specification](https://spec.openapis.org/oas/v3.0.3)

| Trial | Observable pass condition |
| --- | --- |
| Receipt and HTTP serialization | Receive two crates totaling 1.250 kg. Assert both quantities and their units on the wire, in the ledger and in the generated client. Reject numeric JSON input for a string field and 0.0005 kg for an item limited to grams. |
| Crash and replay | Fail after writing the movement but before commit, then retry. Separately lose the response after commit. In each case the final stock reflects one posting and retry returns the settled movement ID. |
| Contention and database serialization | Two sessions each reserve 0.750 kg from 1.250 kg. Coordinate execution with barriers. One reservation settles and 0.500 kg remains available. For a serializable multi-row case, seed two bins with one bag each and a trial rule that one bag must remain free across the site. Both sessions read both bins before claiming different bags. After the serialization retry, one reservation settles and one bag remains free. [Deterministic stock verification examples](inventory-verification.md) |
| Hold races | Apply a hold concurrently with a reservation, then with dispatch of a staged parcel. When the hold settles first, block new reservations and dispatch. If a reservation already settled, flag it and stop the pick or handover. If dispatch settled first, preserve the movement and include its recipient in the trace. |
| Upgrade and data compatibility | Add a new eight-bag pack alongside a historical ten-bag pack. Run both implementations against supported old receipt payloads, migrate twice and verify original unit evidence and quantities. [Historical unit migration](code-practices.md) |

Only compare speed after stock and contract checks pass. Record attempted and committed commands per second separately, median and 95th-percentile end-to-end latency, query count, lock waits and retry count for uncontended and hot-bin workloads. Repeat matched runs in alternating order. Have another developer change a receipt field and inspect the migration and generated client. Record the time, files touched and defects each reviewer caught. These are proposed measurements, with no target until a warehouse supplies its actual peak and response-time needs.

## Support check before selection

The documentation was checked on 26 November 2026. At selection, record the actual mutually supported Node, framework, validator, OpenAPI generator, driver and database versions. Review Fastify’s LTS and Swagger plugin compatibility tables. For Hono, verify the Node adapter and Zod OpenAPI peer dependencies together, and run the selected Drizzle release and migration-tool version against the chosen PostgreSQL major. Re-run the fixture suite after a representative dependency upgrade. Choose on demonstrated behavior and the team’s ability to maintain the combination through the intended product window. 2027 package availability and compatibility remain unverified. [Node release schedule](https://nodejs.org/en/about/previous-releases) · [Fastify LTS policy](https://fastify.dev/docs/latest/Reference/LTS/) · [Fastify Swagger compatibility](https://github.com/fastify/fastify-swagger)
