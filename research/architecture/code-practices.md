# Code practices for maintaining a small WMS

A receiving clerk, a spreadsheet import and a merchant API should reach the same stock-posting code. Otherwise a fix for the mobile screen can leave the importer counting cases as individual bags. The examples below assume one deployable application and a transactional database. The TypeScript-style names illustrate responsibilities that also apply in other languages.

## Organize files around work people recognize

```text
catalog/
  public.ts                 item, variant and dated pack definitions
inventory/
  public.ts                 postReceipt, moveStock, correctMovement
  post-receipt.ts           validate and commit a stock receipt
  movement-store.ts         queries and writes owned by inventory
receiving/
  record-observation.ts     what arrived, who saw it, what needs review
integrations/
  merchant-receipts.ts     translate the partner’s receipt format
```

A developer changing a receipt starts in `receiving`. The import `receiving → inventory/public` is allowed. An import of `inventory/movement-store` from a route, report or partner adapter would bypass the stock rules. Scope an import check to consumers and cover aliases and relative paths. In an ESLint project, restricted import paths and patterns can enforce those static imports. Also review direct SQL against inventory-owned tables, which an import check cannot see. Export a second operation when another caller has a genuinely different job. [ESLint restricted imports](https://eslint.org/docs/latest/rules/no-restricted-imports)

Keep parsing and calculation in small functions. A scanner decodes a barcode, the catalog resolves it to an item or variant, and inventory checks whether that item may be posted in the supplied unit. A bag uses a whole count. Produce can carry an independent measured weight alongside crate count. Two shirt sizes have distinct stock identities. These are item properties that different merchants can share. A merchant name is a poor switch for choosing arithmetic. Use exact decimal quantities or integer multiples of a declared unit, and specify the precision and rounding of each measurement. [PostgreSQL numeric types](https://www.postgresql.org/docs/18/datatype-numeric.html)

## Give a command a testable contract

For example, `postReceipt` takes an authenticated merchant scope, a command ID, an observation reference, a location and one or more actual item lines. Each line carries the item or variant ID and a quantity with its unit. It can carry a lot, measured net weight or a pack-definition revision when the item calls for one. A successful result identifies the posted movement. Replaying the same command returns that movement. A different payload with the same scoped ID returns a conflict. An unrecognized unit or item produces a reviewable observation and leaves on-hand stock untouched.

Persist the observation before attempting a posting that could need human review. Give that observation and its pending command a durable link so a restart can resume the attempt. Inside the inventory transaction, claim the command ID under a unique constraint, validate and apply the stock changes, and save the result. If a notification is required, write an outbox entry in the transaction and deliver it later. Keep the transaction short. A scale reading, a worker’s decision or a partner’s acknowledgement can arrive minutes later. Record it as a subsequent step linked to the original observation. [PostgreSQL transactions](https://www.postgresql.org/docs/18/tutorial-transactions.html) · [AWS transactional outbox](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)

## Change a pack size without changing history

Suppose an ice distributor records four cases received when each case held ten bags. A later supplier pack contains eight. The historical receipt represents **40 bags**. A new receipt of four of the smaller cases represents **32 bags**. The old observation also says “four cases,” which an operator may need when discussing the original delivery.

1. **Expand the schema.** Add fields for entered quantity and unit, stock quantity and unit, and the pack definition used. Preserve the old data. Deploy readers that understand the previous representation and new writers that capture the conversion used at posting. While old writers remain active, retain a compatible input path and include their late writes in reconciliation.
2. **Backfill using evidence available for that movement.** Use the actual receipt’s pack identifier, recorded conversion or evidence that the received pack matched a dated purchase line. A purchase line alone establishes what was ordered. If substitutions or mixed packs leave the delivered quantity uncertain, preserve the source and route it to review. If an old row says only “4” and its unit is unknown, retain the original value and flag the row for investigation. A nominal crate weight cannot recover the farm’s measured historical kg. A style-level total cannot establish which shirt sizes were received.
3. **Reconcile before switching reads.** Re-run the backfill in resumable batches, compare movement and balance totals by item, location and unit, and check historical reports and open work. Resolve ambiguous records individually or retain them in the legacy representation with their uncertainty visible. Preserve the original source and correction trail.
4. **Retire the legacy path after its consumers finish.** Test an offline device created under the old pack and an older merchant client. Resolve a delayed command against its original item/pack identity or ask for review if ambiguous. Changing the meaning of an existing `quantity` field breaks the contract even when its type stays the same. Introduce explicit units or a new operation version. Enable new quantities that older clients cannot represent only after those clients have an appropriate path. [GitLab migration practices](https://docs.gitlab.com/development/database/avoiding_downtime_in_migrations/) · [Google API compatibility guidance](https://google.aip.dev/180)

## Put the most expensive mistakes under test first

| Failure cost | Smallest useful test | Evidence it should produce |
| --- | --- | --- |
| Stock invented or lost | Two real database connections reserve the final bag. Force failure after one leg of a stock transfer. Retry a receipt whose committed reply was lost. | At most one reservation wins, the failed transfer changes neither balance, and the replay yields the original receipt ID and a single movement. |
| Old stock or traceability silently rewritten | Migrate fixtures with ten-bag and eight-bag cases, ambiguous historical units, and a measured farm lot. Run the migration twice. | Old and new receipts remain 40 and 32 bags, ambiguity remains visible, and recorded weights and lot links survive both runs. |
| Item dispatched under the wrong identity | Run the public receiving and return commands on mixed shirt sizes, with a damaged large returned on hold. | Counts reconcile by size and condition. Three sellable large shirts plus one held large produce four on hand and three available. |
| Real clients disagree after a release | Send captured, de-identified payload shapes from a supported older client and a current one. Exercise one receive-and-resume path from the real scanner UI. | Both clients agree on stock units, settled command ID and review state. The operator can distinguish local save from committed receipt. |

Use focused examples for known failures. If releases keep exposing mistakes across sequences of receive, move, hold and correct, generate sequences and compare final balances with a simple independent model. Stateful testing tools can shrink a failing sequence into a reproducible case for the next developer. Run the same transaction failures against each candidate server stack before comparing latency. [Stock checks for the server-tooling trial](server-tooling.md#a-comparative-implementation-trial) · [Hypothesis stateful testing](https://hypothesis.readthedocs.io/en/latest/stateful.html)
