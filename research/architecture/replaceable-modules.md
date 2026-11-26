# Replaceable WMS modules and stock ownership

Two workers can both see six units in a bin. If their orders each need four, one component must decide which commitments the warehouse can honor. The picking algorithm, handheld client and database can each be replaced if their contracts preserve that decision and let operators recover when the physical count disagrees.

## Where to draw the boundaries

This candidate design assumes reservations against eligible, physically present units at a known bin and lot. Expected receipts, reservations made at warehouse level and catch-weight items would change the model. Dynamics 365 illustrates how inventory dimensions and the level at which a reservation is made affect the interpretation of an on-hand total. [Dynamics 365 inventory on-hand list](https://learn.microsoft.com/en-us/dynamics365/supply-chain/inventory/inventory-on-hand-list)

| Candidate module | Owns | Contract with the rest of the WMS |
| --- | --- | --- |
| Inventory | Physical movements, balance by stock identity, holds, reservation lifecycle and approved corrections | Receives commands to reserve, release, move or adjust. Returns the committed result and its revision. A bin transfer posts both legs together. |
| Allocation policy | Ranking eligible stock, e.g. earliest expiry first | Returns candidate bins, lots, quantities and policy version from a supplied snapshot. Inventory rechecks eligibility and availability when committing. |
| Warehouse work | Receiving, pick and count tasks, operator observations, discrepancies and handoffs | Persists the operator's observation and task progress. Links each resulting stock command and its outcome to that task. |
| Devices and integrations | Scan capture, delivery state, partner identifiers and protocol translation | Translate a scan or partner request into the same business command used by other clients. Preserve its identity across retries. |
| Reporting | Searchable stock and work views, refresh position | Reads committed changes. Shows its latest applied revision or refresh time so an operator knows how fresh the view is. |

These are ownership boundaries that can share a deployment. Keeping a module's tables private lets a caller depend on operations and outcomes while the module changes its storage. Using business language at each boundary also accommodates different meanings of “complete” in picking, shipping and inventory. [Microsoft guidance on domain analysis](https://learn.microsoft.com/en-us/azure/architecture/microservices/model/domain-analysis)

An atomic posting can represent one physical step. If a cart leaves bin A before anyone confirms arrival at bin B, the first posting moves stock from A to an in-transit location. A later posting moves it from there to B. An interrupted trip then leaves the next shift an accurate place to start looking.

## Two deployment options

| Approach | Where it fits | Cost to demonstrate |
| --- | --- | --- |
| Modular application with one transactional store | A small team often changes work and inventory together. Local calls and transactions simplify a bin transfer. | Verify module imports and data access boundaries. Replace the allocation implementation behind the same interface and run both against the same fixtures. Releases remain coordinated. |
| Separately deployable inventory or integration service | Inventory has a distinct owner or workload, or a partner integration needs an independent release or failure boundary. | Measure command latency and contention, exercise service timeout and recovery, and establish the age of each downstream projection. A durable workflow must resume across partial success. |

Independent storage buys deployment autonomy at the price of propagating changes and reconciling in-flight work. A frequently repeated synchronous round trip between modules is evidence to revisit the split. [Microsoft guidance on service data ownership](https://learn.microsoft.com/azure/architecture/microservices/design/data-considerations)

## Worked example — two reservations and a lost reply

1. Bin A holds six eligible units of lot L and zero reservations. Orders X and Y concurrently request four units each, all or nothing. Each sends its own stable command ID with its order line, stock identity, quantity and unit.
2. Inventory evaluates availability and writes the reservation under one transaction. An atomic conditional update can guard a single balance. A multi-bin rule needs protection across every affected balance, for example a serializable transaction with a full retry on serialization failure. If X wins, its reservation is four, available is two, and physical stock remains six. Y receives “insufficient stock.” [PostgreSQL transaction isolation](https://www.postgresql.org/docs/18/transaction-iso.html)
3. X loses the reply. Its retry returns the same reservation ID from a command record committed atomically with the reservation. Reusing that key with a changed quantity returns a conflict. Define the key scope, retention period and treatment of late device replays in the contract. For this example, repeating Y's original command returns its original decision. A deliberate later attempt uses a new ID. [Amazon Builders’ Library on idempotent APIs](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
4. If other modules subscribe, commit the event to an outbox with the stock change. A relay may deliver twice after a crash. A consumer records the event ID together with its local update, detects gaps using the stream revision and replays missing changes. [AWS transactional outbox guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)
5. The picker finds only three units for X. Save “three picked, one short” against the task, post the three-unit move to staging, resolve the outstanding reservation under the short-pick policy and assign the bin discrepancy for investigation. A later approved correction references the observation and original movement. A physical observation can thus survive even if a proposed posting needs review. [Product principles](../../PRINCIPLES.md)

## What disconnected work can promise

A handheld can durably save scans with the command ID, task, quantity, observed time and last known revision. Show “saved on this device” until the server confirms. A second device can resume from the last synchronized step. If the first device still holds unsent observations, handoff must surface that gap. On reconnect, preserve contradictory observations for a named reviewer and link the resolution to the original work.

If the aisle must authorize a new reservation while disconnected, evaluate exclusive preassigned work or quantity rights per device or site. With four rights granted to the aisle and two retained centrally, each can spend only its own share. The safety cost is stranded capacity when a device disappears. Recovery must account for old-device writes before reissuing its rights. Quantity rights alone do not settle lot recalls or conflicting claims to the same serial number. The bounded-counter research supplies the numeric mechanism. The warehouse eligibility and recovery rules still need validation. [Balegas et al., *Extending Eventually Consistent Cloud Databases for Enforcing Numeric Invariants*](https://arxiv.org/abs/1503.09052)

## How to demonstrate a replacement

- **For a policy change**, run both implementations on eligible, held, expired and tied candidates. A changed ranking can be intentional. Check that the proposed quantities, units and eligibility satisfy the contract, and that inventory rejects stale choices at commit.
- **For a stateful replacement**, verify the consumer interactions plus stock invariants, concurrent attempts, late retries and interrupted transfers against both providers. Message-shape checks cover only one part of that evidence. [Pact guidance on contract and functional testing](https://docs.pact.io/getting_started/what_is_pact_good_for)
- **At cutover**, import balances, open reservations, corrections, command outcomes and outstanding event positions into the candidate. Replay through a recorded position with external side effects disabled. Compare quantities by stock identity and reservation by order line. Fence the former writer, drain in-flight commands, reconcile the final position, then switch routing. A rollback after new writes requires replaying them into the previous implementation and rechecking the same balances and identifiers. [Microsoft guidance on incremental replacement](https://learn.microsoft.com/en-us/azure/architecture/patterns/strangler-fig)

## Questions for the warehouse

- At what point does a bin, lot, owner or serial become binding, and can an order be partially allocated?
- Which steps must a worker physically complete during an outage? How long can a device remain offline, and how is its unsent work recovered at shift change?
- Who may release a short-picked remainder, approve a count variance or place a lot on hold while other workers are picking it?
- What measured stocking contention, maximum projection lag or independent release requirement would justify splitting inventory into a separate service?
