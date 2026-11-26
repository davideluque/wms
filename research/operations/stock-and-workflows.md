# Warehouse operations and inventory

Start with four questions at each handoff. What was expected? What physically happened? What is someone now allowed to do with the goods? Who still owes a decision? A single “complete” flag can discard information needed by the next shift.

## Follow the goods

| Operation | Facts worth distinguishing | Exception to replay |
| --- | --- | --- |
| Receive and inspect | Ordered item, actual item, quantity and unit, condition, source document | Partial delivery, unfamiliar barcode or unapproved substitute |
| Put away, replenish and transfer | Origin, destination, quantity picked up and quantity confirmed at destination | The move was interrupted with goods still on the cart |
| Reserve and pick | Demand, stock eligible for this order, quantity committed and quantity found | Two orders compete for the last case or the assigned bin is empty |
| Pack and dispatch | What each package contains, what is staged and what left with the carrier | One package misses collection after a shipment was prepared |
| Return and release | Original shipment if known, goods returned, condition and disposition | A return arrives and needs inspection before resale |
| Count and correct | Count scope and time, amount observed, intervening movements and approved adjustment | Someone picks from the bin while a count awaits review |

A reservation can exist at warehouse level before a specific location or pallet is assigned. A cycle count can also remain open while picking continues. Dynamics documents both behaviors. For a pilot, ask which commitments staff make early and how they detect movement during a count. [Dynamics reservation hierarchies](https://learn.microsoft.com/dynamics365/supply-chain/warehousing/reservations-in-warehouse-management) · [Dynamics cycle counting](https://learn.microsoft.com/en-us/dynamics365/supply-chain/warehousing/cycle-counting)

## Worked receipt

Suppose an order calls for **10 cases of 12 units**. Nine cases arrive, including one damaged case held for inspection. Assume the other eight pass inspection immediately, with no prior reservations. The reporter needs several different answers.

| Question | Answer at this point |
| --- | --- |
| What did the supplier owe on the original order? | 120 units |
| What arrived physically? | 108 units |
| How much is missing from this delivery? | 12 units, pending the buyer's decision on the remaining order quantity |
| What can be allocated under this site's policy? | 96 units |
| What is present but held? | 12 units, with the inspection outcome pending |

If inspection later releases the held case, eligible stock rises to 108 and the original receipt still records 108. If the receiver entered nine twice by mistake, record a correction linked to the mistaken entry. A device retry of the original submission should produce a single receipt. An intentional scan of a second identical case should count both. Use the package mapping to interpret the label, then run both actions on the floor. [Catalog identities and unit conversions](catalog-and-units.md) · [Repeated scans and save retries on the handheld](../experience/floor-workflows.md#a-second-scan-and-a-second-attempt-to-save)

## Find the boundary with the ERP

Ask who owns the purchase order, item and pack definitions, physical receipt, supplier claim, invoice and financial posting. In one documented arrangement, the WMS sends a shipment receipt to the ERP, which can use it for invoicing. The systems reconcile stock at an agreed instant and level of detail. Microsoft's example also illustrates the risk of counting one receipt again via a second inventory feed. [Dynamics warehouse management with an external ERP](https://learn.microsoft.com/en-us/dynamics365/supply-chain/warehousing/wms-only-mode-external-erp)

For the pilot, replay the example across that boundary. Match order and line references, item and unit, accepted and held quantities, the time of the physical observation and the time of acknowledgement. Interrupt the exchange after the ERP accepts the receipt and before the WMS sees the reply. A buyer should be able to determine whether the receipt posted and retry safely. [Who owns stock and acknowledgment at a module boundary](../architecture/replaceable-modules.md) · [Warehouse and accounting handoffs in Venezuela](../venezuela/local-discovery.md#dispatch-and-accounting)

## Questions that change the model

- Does “on hand” include receiving, inspection, staging or another owner's goods? Ask which number purchasing, picking and accounting each use.
- If a replacement has a different item code or pack size, who approves it and which original line remains traceable?
- Is a sealed case always interchangeable with twelve loose units? Does the partner accept broken cases, differing pack sizes or variable weights?
- Which actions can staff reverse on the floor? Once goods leave the building, who records a later return or documentation error?
