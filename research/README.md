# WMS research

Our first customer profile is a small or medium Venezuelan merchant with a sales channel, a warehouse and potentially several stores. The [retail MVP](../MVP.md) defines the proposed pilot. We still need to choose a specific merchant and see how its staff receive, promise and hand over stock.

These notes combine published sources, proposed design choices, and questions for fieldwork. We have yet to observe a pilot site. Examples with quantities are hypothetical.

## What we have tested

| Experiment | Observed result | Boundary |
| --- | --- | --- |
| [Two connections reserve the last bag](architecture/race-conditions.md#native-postgresql-check-on-the-last-bag) | PostgreSQL held the second writer until the first finished. A first-writer commit left one promise for A. A rollback let B take it. | Two controlled schedules on one stock row |
| [PostgreSQL merchant isolation](architecture/tenant-isolation-trial.md) | Twelve assertions passed for merchant-scoped reads and writes, a composite foreign key, reuse after rollback and privileged-role behavior. | Two synthetic merchants in a disposable native PostgreSQL 16.15 cluster |
| [Offline observation capture](architecture/offline-capture-trial.md) | An aborted observation and command rolled back together. A repeated command retained its original quantity after a browser restart. | Local Chromium storage; separate merchant keys on one origin still require application authorization |
| [Floor-plan marker and list](experience/warehouse-map.md#browser-feasibility-check) | The marker remained aligned with its shelf at two screen sizes. Keyboard selection worked through the HTML list with the diagram removed. | A synthetic SVG fixture in headless Chromium |
| [HTTP contract comparison](architecture/server-tooling.md#observed-http-contract-on-two-frameworks) | Across 32 requests, the default validators differed on numeric input. With strict settings both frameworks agreed on all eight receipt cases. | Loopback HTTP and exact decimal-string validation, without database or latency measurements |
| [Order and webhook recovery](integrations/merchant-api.md#local-trial-of-the-order-and-its-webhook) | After four process interruptions and full replay, three orders produced three applied events. A missing event for one merchant did not delay another. | Local SQLite producer and consumer, without external event delivery |
| [Paging a changing inventory](integrations/inventory-catch-up.md#observed-on-native-sqlite) | A live offset scan duplicated one record, and even a stable key missed an earlier insertion. Reading a fixed snapshot and replaying changes reproduced the ledger at both checkpoints. | Three native SQLite connections and a synthetic inventory feed |
| [Commit order and stock-feed cursors](integrations/inventory-catch-up.md) | Checkpointing the highest visible sequence ID missed a five-bag receipt. A transactionally updated revision per merchant delivered all twelve bags and let a different merchant continue writing. | Two controlled PostgreSQL 16.15 schedules, without a throughput benchmark |
| [Enterprise export replay](architecture/tenant-export-example.md#replay-evidence-and-limits) | Five file hashes matched. Reimporting kept the two outstanding bags in transit, and a foreign merchant's attachment was rejected. | Synthetic in-memory import and a held delivery record |
| [GS1 scan capture](standards/barcode-capture-trial.md) | Removing one separator produced a valid but different lot and no expiry. An item requiring expiry needs a separate field check. | Eight synthetic scanner outputs through the official syntax engine, without physical image decoding |
| [Pilot stock walkthrough](operations/pilot-balance-replay.md) | Ten units reconciled across six distinct commands, including a sale and a partially received transfer. | A Python arithmetic model |

## Merchant discovery and local context

| Investigation | Decision it informs |
| --- | --- |
| [First warehouse and fieldwork](discovery/first-warehouse.md) | Who experiences the problem, who buys the product, and what evidence would justify a pilot? |
| [Local discovery in Venezuela](venezuela/local-discovery.md) | Which documents, business systems and operating conditions matter at the first site? |
| [Ice, farm and clothing businesses](discovery/three-merchant-scenarios.md) | Which stock operations travel across businesses, and which require measured weights, lot tracking or exact variants? |
| [Spanish merchant interview guide](discovery/merchant-conversations.md) | How can the first ice, farm and clothing conversations start with a recent sale and a real discrepancy? |
| [Existing alternatives](discovery/existing-alternatives.md) | Would a configuration change or a small integration solve the merchant's actual problem? |

## Warehouse and store operations

| Investigation | Decision it informs |
| --- | --- |
| [Shelf and bin addressing](operations/location-addressing.md) | What code should a worker see and scan at the same shelf from either end of an aisle? |
| [Custody and ownership](operations/custody-and-ownership.md) | How does moving merchant-owned goods between stores differ from handling another owner's goods? |
| [Manual picking and travel](operations/manual-picking.md) | Would better shelf identification, a different walking order or a two-order batch improve this site's work? |
| [Common warehouse problems](operations/common-problems.md) | What should a worker check first when a bin is empty, a transfer disappears or two customers are promised the same item? |
| [Cycle counting](operations/cycle-counting.md) | How can staff count while trading and avoid deducting a late sale twice? |
| [Warehouse operations and inventory](operations/stock-and-workflows.md) | What happened physically, who can act on it, and what can the next shift trust? |
| [Supplier receiving without an ERP](operations/supplier-receiving.md) | How can a small shop record a substitute or short delivery before anyone settles the supplier bill? |
| [Store replenishment](operations/store-replenishment.md) | How much should a store request when stock is scarce or a transfer is already on its way? |
| [Opening inventory and cutover](operations/opening-inventory.md) | How does a merchant begin with trustworthy balances while sales continue? |
| [Order routing and pickup](operations/order-routing.md) | Which site can fulfill the whole order and honor a customer's chosen pickup point? |
| [Returns and corrections](operations/returns-and-corrections.md) | What changes when an item is refunded, physically returned, exchanged or found after dispatch? |
| [Catalog and units](operations/catalog-and-units.md) | Can old and new pack sizes coexist and an old receipt still identify exactly what was sold? |
| [Permissions and stock history](operations/permissions-and-history.md) | Who may report, approve and post a stock change, and how will the next shift reconstruct it? |
| [Pilot balance replay](operations/pilot-balance-replay.md) | Can the initial ten units be reconciled across a sale, a short transfer, retries and a carrier handoff? |
| [Perishable traceability](operations/perishable-traceability.md) | Which goods can be traced, held or recalled, and what evidence would authorize release? |

## Floor and manager experience

| Investigation | Decision it informs |
| --- | --- |
| [Venezuelan quantity entry and dates](experience/venezuelan-entry-and-display.md) | How should a scale reading, decimal comma or late register sale appear in the right unit and business day? |
| [Warehouse maps](experience/warehouse-map.md) | How do we connect a physical bin to a searchable, accessible floor plan and decide whether SVG, Canvas or Leaflet fits? |
| [Merchant accounts and staff access](experience/merchant-account.md) | How does an owner invite workers, switch businesses, transfer ownership or exit with pending stock work? |
| [Floor workflows and accessibility](experience/floor-workflows.md) | Can a worker scan, correct a mistake and hand over a partly finished task on the site's equipment? |
| [Exception queues for managers](experience/manager-exceptions.md) | Which interruptions need a person now, and how does that responsibility pass to the next shift? |
| [Labels and printing](experience/labels-and-printing.md) | When does a supplier label suffice, and how do staff recover from a swapped or duplicate label? |

## Standards and integrations

| Investigation | Decision it informs |
| --- | --- |
| [Inventory snapshots and catch-up](integrations/inventory-catch-up.md) | Can a merchant rebuild current stock when lines change or disappear halfway through a paged read? |
| [First order over the API](integrations/first-order-walkthrough.md) | What does a checkout send, retry and read to track one pickup sale through an unknown SKU and collection? |
| [Identification and exchange standards](standards/identification-and-exchange.md) | Which identifiers, labels and messages must survive the first trading-partner handoff? |
| [Merchant APIs and webhooks](integrations/merchant-api.md) | How can a checkout submit a sale once, recover from a lost response and reconcile missed events? |
| [Shopify integration](integrations/shopify.md) | How do checkout, pickup and fulfillment affect stock in both systems? |
| [AROA POS integration](integrations/aroa-pos.md) | How can a completed sale and its paid check produce a single movement at the correct store? |
| [Venezuelan business systems](integrations/venezuelan-business-systems.md) | What do a2, Profit Plus and Saint document, and which line-level records could the first merchant actually supply? |
| [Venezuelan parcel carriers](integrations/venezuela-carriers.md) | What evidence distinguishes a printed guide, a booked pickup, carrier acceptance and delivery? |

## Architecture and technology

| Investigation | Decision it informs |
| --- | --- |
| [Tenant URLs and data isolation](architecture/tenant-isolation.md) | How does an ordinary merchant sign in, and what keeps its stock, files and integrations private? |
| [Customer-operated enterprise installations](architecture/enterprise-installation.md) | What must ship for a customer to control its domain, data and network while running the same release? |
| [Tenant export and restore example](architecture/tenant-export-example.md) | Can a merchant take an unfinished transfer and its evidence to another installation while preserving who owns every record? |
| [Race conditions](architecture/race-conditions.md) | Which decisions must be coordinated when a second order, cancellation or hold arrives concurrently? |
| [Replaceable modules](architecture/replaceable-modules.md) | Which decisions can vary independently and who owns each stock change? |
| [Web and mobile options for 2027](architecture/web-to-mobile.md) | Which parts of the floor application can travel from browser to native, and what would justify that investment? |
| [Code practices](architecture/code-practices.md) | How do importers and screens share stock rules, and how do new pack sizes preserve historical balances? |
| [Inventory verification](architecture/inventory-verification.md) | Can two devices reserve the last units, or a migration change a historic quantity? |
| [Server tools for 2027](architecture/server-tooling.md) | Which HTTP, schema and database libraries make stock transactions easiest for the team to verify? |

The [decisions for the retail pilot](../DECISIONS.md) record working assumptions and the evidence that could change them.

## Pilot verification issues

- Use a shared handheld to switch merchants with unposted work, including a revoked membership. [Issue #1](https://github.com/davideluque/wms/issues/1)
- Capture a real label and recover when an expiry field disappears from scanner output. [Issue #2](https://github.com/davideluque/wms/issues/2)
- Sync an offline POS sale that happened before the opening count, then a separate sale after cutover. [Issue #3](https://github.com/davideluque/wms/issues/3)
- Rehearse two clerks counting the same carton, then two distinct cartons carrying the same product. [Issue #4](https://github.com/davideluque/wms/issues/4)
- Verify the stock-feed cursor against writes that allocate IDs in one order and commit in another. [Issue #5](https://github.com/davideluque/wms/issues/5)

## In what order?

1. **Choose a site and observe a job.** Follow one routine movement and reconstruct a recent exception. Ask the next person in the chain to explain what they received and what they did with it. Collect the labels, quantities, decisions and elapsed time needed to replay both cases.
2. **Replay the evidence.** Use the standards notes to interpret the labels, the operations notes to reconcile quantities, and the architecture notes to locate the decision that changed stock. Record any partner rule or local requirement with its source and scope.
3. **Prototype one complete handoff.** A candidate is receiving a short delivery, putting away the accepted quantity, and reporting the shortage to the buyer. Pick the workflow after the visit. Have the people who do it try the ordinary case, a correction, and resuming on the next shift.

## Further topics worth investigating

| Topic | Evidence to gather | Decision it unlocks |
| --- | --- | --- |
| Master data and onboarding | One redacted item export, pack conversions, duplicate codes, opening balances and a count sheet | How to start with stock that staff can reconcile on day one |
| Floor devices and accessibility | Try the actual scanner, phone, printer and labels at the dock, using the team's language and working conditions | Which actions need scanning, manual entry, large controls or a printed fallback |
| Offline operation and recovery | Replay a lost acknowledgement, a device restart and two people claiming the last case | Which actions can finish offline, which remain pending, and how conflicts reach a person |
| System integrations and ownership | Follow one purchase line and its receipt into the customer's ERP, including a correction | Which system owns each fact, how to match records, and who resolves mismatches |
| Permissions, traceability and support | Ask who can release held stock, correct a shipment, inspect history and restore operations after losing a device | Minimum role boundaries, evidence to retain and recovery procedure |
| Commercial viability and rollout | Measure duplicate entry and time spent resolving discrepancies. Ask the buyer about hardware, onboarding, support costs and switching constraints | Whether a measurable improvement can outweigh the cost of changing systems |
| Existing alternatives | Replay the same short delivery and correction in the site's current tool and one candidate WMS. Compare steps, missing facts and setup effort | Where an existing product or integration suffices and where a Venezuelan operator would benefit from a new product |
| Sector and scale | Identify any customer handling third-party stock, lots, expiries, recalls or variable weight. Collect actual peak volumes and concurrent work | Which capabilities belong in the first release and which need a different customer profile |

Use the [product principles](../PRINCIPLES.md) to evaluate the proposed workflow. A particularly revealing check is whether someone on the next shift can explain a discrepancy and safely finish the job without calling the person who started it.
