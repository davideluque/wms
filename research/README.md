# WMS research

Our first customer profile is a small or medium Venezuelan merchant with a sales channel, a warehouse and potentially several stores. The [retail MVP](../MVP.md) defines the proposed pilot. We still need to choose a specific merchant and see how its staff receive, promise and hand over stock.

These notes combine published sources, proposed design choices, and questions for fieldwork. We have yet to observe a pilot site. Examples with quantities are hypothetical.

## Investigations

| Investigation | Decision it informs |
| --- | --- |
| [First warehouse and fieldwork](discovery/first-warehouse.md) | Who experiences the problem, who buys the product, and what evidence would justify a pilot? |
| [Warehouse operations and inventory](operations/stock-and-workflows.md) | What happened physically, who can act on it, and what can the next shift trust? |
| [Identification and exchange standards](standards/identification-and-exchange.md) | Which identifiers, labels and messages must survive the first trading-partner handoff? |
| [Replaceable modules](architecture/replaceable-modules.md) | Which decisions can vary independently and who owns each stock change? |
| [Local discovery in Venezuela](venezuela/local-discovery.md) | Which documents, business systems and operating conditions matter at the first site? |
| [Store replenishment](operations/store-replenishment.md) | How much should a store request when stock is scarce or a transfer is already on its way? |
| [Merchant APIs and webhooks](integrations/merchant-api.md) | How can a checkout submit a sale once, recover from a lost response and reconcile missed events? |
| [Shopify integration](integrations/shopify.md) | How do checkout, pickup and fulfillment affect stock in both systems? |
| [AROA POS integration](integrations/aroa-pos.md) | How can a completed sale and its paid check produce a single movement at the correct store? |
| [Floor workflows and accessibility](experience/floor-workflows.md) | Can a worker scan, correct a mistake and hand over a partly finished task on the site's equipment? |
| [Web and mobile options for 2027](architecture/web-to-mobile.md) | Which parts of the floor application can travel from browser to native, and what would justify that investment? |
| [Ice, farm and clothing businesses](discovery/three-merchant-scenarios.md) | Which stock operations travel across businesses, and which require measured weights, lot tracking or exact variants? |
| [Opening inventory and cutover](operations/opening-inventory.md) | How does a merchant begin with trustworthy balances while sales continue? |
| [Venezuelan parcel carriers](integrations/venezuela-carriers.md) | What evidence distinguishes a printed guide, a booked pickup, carrier acceptance and delivery? |
| [Spanish merchant interview guide](discovery/merchant-conversations.md) | How can the first ice, farm and clothing conversations start with a recent sale and a real discrepancy? |
| [Existing alternatives](discovery/existing-alternatives.md) | Would a configuration change or a small integration solve the merchant's actual problem? |
| [Order routing and pickup](operations/order-routing.md) | Which site can fulfill the whole order and honor a customer's chosen pickup point? |
| [Returns and corrections](operations/returns-and-corrections.md) | What changes when an item is refunded, physically returned, exchanged or found after dispatch? |
| [Code practices](architecture/code-practices.md) | How do importers and screens share stock rules, and how do new pack sizes preserve historical balances? |

The [decisions for the retail pilot](../DECISIONS.md) record working assumptions and the evidence that could change them.

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
