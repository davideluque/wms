# Automated checks for stock integrity

Give the test runner a small warehouse it can reconcile by hand. Use a disposable database with the same engine, isolation level, constraints and application role as the proposed deployment. Exercise the normal receiving and reservation entry points. The examples assume fixed-bin, all-or-nothing reservations and a policy that rejects excess decimal places. A warehouse that permits partial allocation, rounding or negative stock needs its own expected results.

## An independent reference model

Represent stock as a map keyed by merchant, item or variant, location, lot when applicable, and disposition. Track reservations, observed quantities and command outcomes separately. Use integers at the item’s declared precision, e.g. 1,250 grams for 1.250 kg or one unit for a single shirt. A measured net weight and a crate count are independent quantities. Sum each dimension separately. Keep the reference arithmetic independent of the production parser, conversion helper and SQL. PostgreSQL distinguishes exact decimals from floating-point types and can round inputs to a declared column scale, so test precision at the entry point as well as in storage. [PostgreSQL numeric types](https://www.postgresql.org/docs/18/datatype-numeric.html)

Generate short sequences of receive, reserve, release, transfer, dispatch, hold, correct and replay commands. Include invalid commands deliberately, such as requesting more than the eligible balance. After each settled command compare the response, movement count, reservations and affected balances to the model. Check conservation across locations for transfers, account for every external receipt or issue, and trace corrections to prior movements. A rejected command may create a review task. Its stock effect must be zero. A model-based tool such as fast-check can generate and shrink these sequences. [fast-check model-based testing](https://fast-check.dev/docs/advanced/model-based-testing/)

## Scenario matrix

| Risk | Fixture or action | Result to assert |
| --- | --- | --- |
| Wrong quantity or precision | Receive two crates totaling 1.250 kg. Dispatch one weighing 0.375 kg. Try 0.0005 kg on an item limited to grams. | One crate and 0.875 kg remain. The excess-precision input is rejected under the example policy. Held or unweighed quantities keep their own status. |
| Oversubscription or partial transfer | Compete for the same lot as below. Force a rollback between the source debit and destination credit of one posting. | Accepted reservations fit eligible stock. Both sides of the failed posting retain their original quantities. |
| Replay after partial failure | Lose the response after committing a receipt, retry the identical command, then reuse its ID with a different quantity. Redeliver its event to the reporting consumer. | One movement, the same receipt identity on replay, a conflict on changed input, and one effect in the report once the consumer catches up. |
| Tenant isolation | Seed two fictional merchants with the same SKU, lot name and client command ID. Use the ordinary application role and each merchant’s authorized context for read, post and replay tests. Include an out-of-scope fixture reference as a denial case. | Each merchant sees its own balances and outcomes. Denied work reveals no other merchant’s record and leaves both ledgers intact. Exercise the deployed database role if row security is used. [PostgreSQL row security](https://www.postgresql.org/docs/18/ddl-rowsecurity.html) |
| Lost product identity | Receive two medium and four large shirts. Move one large to hold. Query availability by variant. | Two medium and three large are available. Four large remain on hand, including the held shirt. |
| Migration or adapter drift | Replay the dated case-pack fixtures and the equivalent client commands described below. | Quantities, relevant identifiers, holds, prior command outcomes and outstanding work retain their meaning. |

## A race small enough to audit

Start with one lot of loose produce, 1.250 kg eligible and zero reserved. A and B each request 0.750 kg. For a candidate using a conditional update under PostgreSQL Read Committed, run two database connections and control them with explicit barriers. The update must check availability against the stored balance while modifying it. PostgreSQL re-evaluates an update’s predicate after a competing updater commits. [PostgreSQL transaction isolation](https://www.postgresql.org/docs/18/transaction-iso.html)

| Step | Controlled action | Expected committed state |
| --- | --- | --- |
| 1 | A executes its guarded reservation update. Pause A before commit. | Other readers see 1.250 kg on hand and zero reserved. |
| 2 | B submits its guarded update on the same row. The harness observes B waiting for A’s lock, then releases A’s commit barrier. | A commits 0.750 kg reserved. |
| 3 | B resumes, finds only 0.500 kg eligible and returns insufficient stock. Drop A’s response and retry A with the same ID. | On hand 1.250 kg, reserved 0.750 kg, available 0.500 kg. Exactly one reservation exists, with A’s original ID. |

Run the mirror schedule with B first, then a third schedule where A rolls back at step 2. B must win in the rollback case. Synchronize on the database wait or an explicit test hook with a bounded diagnostic timeout. Sleeping for a few milliseconds makes the result depend on machine speed. A serializable implementation can legitimately abort a transaction and retry it in full, so its harness should assert the final business outcome and record the retry.

## Replay, shrink and compare

- **Migrations.** Seed an old-format receipt of four cases from a documented ten-bag pack. Add the eight-bag pack, migrate, then post a new four-case receipt. Expect 40 and 32 bags on the respective movements and 72 on hand if neither was issued. Re-run the resumable backfill and retry the original receipt with its original command ID. Compare the movements, on-hand total and command outcomes again. An unknown legacy pack goes to review with its entered quantity preserved. [Historical-unit migration example](code-practices.md)
- **Adapter parity.** Starting from fresh copies of the same fixture, submit the same receipt through a scanner adapter, an import and the merchant API. Check merchant scope, item identity, exact quantity, unit, lot, status and ledger effect. Preserve appropriate differences in source attribution. Assert matching domain outcomes for invalid or ambiguous input and for a lost reply. Pair these tests with request/response contract fixtures for supported client versions. [Pact on contract and functional testing](https://docs.pact.io/getting_started/what_is_pact_good_for)
- **Failure evidence.** Save the seed, generator and dependency versions, database configuration, initial fixture, ordered commands, transaction barrier order, and expected versus actual values. Shrink commands and quantities while retaining prerequisite receipts and both competing operations. Replay the minimized case against a fresh database and keep that explicit trace as a regression even if the generator changes. Run the short traces on each change and broader seeded searches separately. Report which scenarios ran and which still await an implementation or device trial.

## Optional catalog trial — an outfit competes for its components

Start each run with one shirt, one blue belt and one tan belt physically at the store. Outfit recipe v1 requires **one shirt and one blue belt**. The virtual outfit has no separately stocked unit. On 26 November 2026, two native SQLite 3.45.1 connections attempted these reservations against disposable fixtures. Physical stock stayed constant throughout.

| First writer reserves | Competing request after the first commits | Free components after both decisions: shirt / blue / tan |
| --- | --- | --- |
| Blue belt alone | Outfit v1 found the shirt, then failed on the belt. Its tentative shirt reservation was rolled back. | 1 / 0 / 1 |
| Outfit v1 | Blue belt alone was short. | 0 / 0 / 1 |
| Outfit v1 | A second outfit v1 was short. | 0 / 0 / 1 |

To repeat the schedule, hold the first writer’s `BEGIN IMMEDIATE` open after recording its component claims. Have the second connection attempt its transaction with zero lock timeout, verify `SQLITE_BUSY`, commit the first writer and retry the second. For each component in recipe order, guard the decrement with `free >= 1`. Roll back **all** tentative decrements and claim rows to a savepoint on shortage, then persist the command outcome. Each losing order ended with zero component claims.

Next publish v2, replacing the blue belt with the tan belt. Replaying the original outfit command with v1 returned its recorded success or shortage and left every balance intact. Reusing its command ID with v2 returned a conflict. A fresh, explicitly accepted v2 order succeeded only in the belt-first schedule, where both the shirt and tan belt remained free. This is a candidate behavior for a future bundle feature. SQLite’s writer lock made the three schedules deterministic. A production implementation must also verify its own transaction isolation and recipe-version contract.
