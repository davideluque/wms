# Replenishing two or three stores

A transfer suggestion should answer “why this item, why this quantity, and where will it come from?” A manager can then approve the work without recomputing it on paper. A useful reference implementation is Odoo's per-location minimum and maximum, which can trigger resupply from another warehouse. It also documents order multiples and forecast horizons. Our pilot can test a narrower rule with one supplying warehouse. [Odoo reordering rules](https://www.odoo.com/documentation/19.0/applications/inventory_and_mrp/inventory/warehouses_storage/replenishment/reordering_rules.html) · [Odoo inter-warehouse replenishment](https://www.odoo.com/documentation/20.0/applications/inventory_and_mrp/inventory/warehouses_storage/replenishment/resupply_warehouses.html)

## A proposed first rule

Use one unit of measure per SKU for the calculation. For each store and SKU, start with saleable physical stock minus reservations, plus the outstanding quantity on timely, confirmed inbound transfers. Count an inbound line once as it moves from approved to in transit. For this rule all demand included has already been reserved. Show unreserved order shortages separately. An expected inbound unit can reduce a replenishment suggestion while still being ineligible for a customer pickup promise.

If the resulting position is below the minimum, propose enough to reach the target. Limit the quantity to eligible source stock after customer commitments and any agreed source floor. For sealed-case transfers, show the case multiple and the resulting overage or shortfall before approval. An open draft is a suggestion and reserves nothing. Recalculate and reserve atomically on approval because another store may have taken the last case.

| Situation | Outcome under the proposed rule |
| --- | --- |
| Store has 1 available, minimum 2, target 4. Source has 5 available. | Suggest 3. A second evaluation updates the existing suggestion. |
| Manager approves 3 and the goods are still at the warehouse. | Source has 2 available. Store has 1 available to customers, plus 3 expected for planning. |
| Source dispatches 3 and store receives 2. | Store has 3 physically available. One remains in transit. Show the discrepancy owner and expected resolution. |
| The remaining unit is declared lost after investigation. | Resolve the outstanding line with a documented loss. Planning position returns to 3. It is above minimum, so no new suggestion is raised. |
| Two stores request 3 each from the same 5 units. | After one approval, the other proposal shows at most 2 source units. Re-evaluate and request a decision on the shortfall. |

## Two cases for three stores

Three stores share one supplying warehouse and one truck that can reach all three before opening. Suppose the warehouse physically holds 17 units, including two on quality hold and three already promised to customers. The manager also keeps four uncommitted units for local sales. That leaves **8 eligible units in two sealed cases of four**. On this run the merchant ships whole cases, allows no store to exceed its target and has no other confirmed inbound. Every store has triggered its minimum.

| Store | Available before the truck arrives | Minimum | Target | Whole cases needed to reach minimum |
| --- | ---: | ---: | ---: | ---: |
| A | 0 | 6 | 8 | 2 |
| B | 1 | 4 | 5 | 1 |
| C | 2 | 5 | 6 | 1 |

The table below lists every way to place both cases within those limits. To reproduce it, assign zero, one or two cases to each store, keep triples that sum to two and discard any that exceed a store target. Add four units per case to its opening balance. These destination balances assume full, saleable receipt before opening.

| Cases to A / B / C | Destination balances after receipt | Stores at or above minimum |
| --- | --- | --- |
| 2 / 0 / 0 | 8 / 1 / 2 | A |
| 1 / 1 / 0 | 4 / 5 / 2 | B |
| 1 / 0 / 1 | 4 / 1 / 6 | C |
| 0 / 1 / 1 | 0 / 5 / 6 | B and C |

Sending a case each to B and C gets two stores above minimum and leaves A unable to sell a single unit. A case each to A and B lets every store open with stock, and leaves C three units below minimum. A manager covering a large morning order at A might choose differently again. If that order becomes a firm customer commitment, include it explicitly and recalculate. Applying a rule to stores in database order would make a business choice without the merchant's knowledge. Oracle's distribution-planning documentation likewise distinguishes demand priority, sharing scarce supply and order multiples. [Oracle distribution planning and allocation](https://docs.oracle.com/cd/E18727_01/doc.121/e13358/T309464T471624.htm)

For the MVP, group suggestions that compete for the same source, item and departure. Put the units available to allocate above the store rows. Show each store's hours at zero, known upcoming demand, next feasible delivery and proposed balance. Let the manager change the case distribution and record the reason. Approving both cases reserves eight source units. The warehouse still physically holds 17 until dispatch. Until receipt, the shops can offer **0 / 1 / 2** even if the driver has already left. If a second store misses the cutoff or the source loses a case, show the changed offer for approval. [How to interview the store manager](../discovery/merchant-conversations.md) · [Open replenishment decision](../../DECISIONS.md)

## Receiving a transfer in parts

Give the transfer, each physical departure, and each receiving observation their own references. A transfer can remain open while one shipment is fully accounted for. Shopify documents partial receipts and lets an operator correct previously received quantities. Its **Reject** action records the rejection without updating location quantities, while **Cancel** can restore items never shipped to the origin. A connector needs a custody check before mapping either action to a WMS movement. [Shopify transfer receiving and corrections](https://help.shopify.com/en/manual/inventory-and-locations/inventory-transfers/receiving-and-managing-transfers)

Suppose the warehouse approves six shirts. It dispatches four in shipment S1. The store accepts two, holds one damaged shirt and cannot find the fourth. The source has two committed but still on its shelf. The destination has three in custody, of which two can be offered. The books carry one dispatched shirt with whereabouts under investigation. Each of the following events has a different stock effect.

| What happens next | What the clerk records | Effect on the original six shirts |
| --- | --- | --- |
| The device resends the three-shirt receipt | Same observation and command ID for S1 | No additional movement. Two remain at source, one unresolved in transit, three at store. |
| Staff find the fourth shirt in S1's bag at the store | A correction linked to S1 and the original count, with evidence of the item's identity and condition | If saleable, three are now available and one is held at store. The in-transit balance for S1 falls to zero. Source remains at two. |
| The source dispatches those two later | A distinct shipment S2 against the same transfer line | Two move from source commitment to transit. S1's counts stay intact. |
| The store accepts S2 | A new two-shirt observation referencing S2 | Six at destination: five available and one held. The transfer can finish while the damage review remains open with its own owner. |

If the store instead discovers it counted one shirt twice, preserve the mistaken receipt and link a downward correction to it. Check whether another order has committed that unit since the error. A later delivery of the same SKU cannot by itself settle which shipment was originally short. Ask the second-shift worker to explain the location and condition of all six units using the two shipment references. [Returns and dispatch corrections](returns-and-corrections.md) · [Worked export of a partially received transfer](../architecture/tenant-export-example.md)

**Two clerks can count the same physical carton.** Six shirts were dispatched in two cartons of three. Only carton A arrives. Two clerks independently record its three shirts, each with a different command ID. Both additions fit inside the six dispatched. Even a perfectly serialized balance update can leave the record at six received and zero in transit while only three shirts are present. If the cartons carry unique handling-unit IDs, claim the same carton once per receiving operation and let the second clerk inspect the existing receipt. A shared product barcode identifies the kind of shirt and can legitimately appear on both cartons. For unlabelled cartons, use one receiving session with an assigned counter and an explicit handoff, or physically separate and identify the work before concurrent counting. If two offline counters bring back overlapping evidence, keep the second observation pending until a person can establish whether a second carton arrived. The pilot should exercise both two counts of A and a genuine receipt of A followed by B.

In a local SQLite fixture, two distinct commands for A passed the quantity cap and posted six. Enforcing one completed carton receipt per shipment and carton kept that case at three, while distinct cartons A and B still posted six. The example assumes independently identifiable sealed cartons. For partial unloading, keep the carton receipt open and preserve its individual count observations. A single unique carton key cannot establish which loose pieces a worker has already counted.

## Notifications a manager can act on

**Test two clocks.** Suppose Store A has one bag at 09:00, minimum two and target four. A confirmed transfer of three is due at 10:00. At 09:05, the store can hand a walk-in customer one bag. Planning may count four units toward the replenishment target, while the checkout can promise only one now. If that customer takes the bag, available becomes zero and the timely incoming three still put the planning position above minimum. A waiting customer needs a truthful availability answer even while the replenishment rule is quiet.

At 10:30 the driver has still not arrived. The store's agreed overdue policy now excludes that supply from the near-term position. Tell the manager that three are already on the way, ask for a revised arrival time and show a proposed fallback. If a second transfer is approved and the original one later arrives, reconcile both instead of silently cancelling a physically dispatched shipment. The purchasing decision depends on what the manager knew at each time. Existing planning systems similarly make the forecast horizon explicit. [Odoo replenishment horizons](https://www.odoo.com/documentation/19.0/applications/inventory_and_mrp/inventory/warehouses_storage/replenishment/reordering_rules.html)

Keep one active shortage per store, SKU and rule while the condition persists. Show available now, incoming with its ETA, minimum, target and source capacity. An acknowledgement records who saw the issue. A snooze has an owner and expiry. Close or revise the suggestion when stock changes, and link an approved suggestion to the transfer so the next shift can follow the goods. If confirmed incoming is overdue, flag the late transfer and recompute using the planning horizon. [Ownership, escalation and handoff in the daily exception queue](../experience/manager-exceptions.md#one-case-from-report-to-handoff)

During the pilot, replay real sales and transfer history chronologically using only facts known at each step. Compare suggestions with managers' actual decisions. Record stockout time during selling hours, transfers per week, declined suggestions and units still in transit after the expected arrival. Ask the manager which direction each metric should move and what extra transfer workload the store can absorb.

For that replay, take one store and one fast seller. Collect the opening balance, sales with their actual occurrence and upload times, holds, transfer approval/departure/receipt times, and the arrival estimate known when each suggestion appeared. Record requests the store could not fill separately if the merchant tracks them. A day with zero completed sales during a stockout does not establish zero demand. Report elapsed selling hours without eligible stock, repeated suggestions for already covered supply and additional trips. Test the same history with approval required, with alternative lead times and with constrained source supply before proposing unattended transfers.

**Open decision.** Customer reservations, scarce stock distribution between stores, package multiples and supplier replenishment can each change an otherwise sensible answer. Begin with operator approval and make the suggested arithmetic visible. [How to evaluate thresholds against past sales](../discovery/first-warehouse.md) · [Retail pilot decisions](../../DECISIONS.md)

## When twelve free bags cannot fill a two-case transfer

A warehouse has three sealed six-bag cases and six loose bags, **24 bags on hand**. Store A’s draft asks for two intact cases. Store B approves one case before A. With the other two cases still intact, A’s request could still fit. A local Python 3.12.3 fixture then exercised two changes before A’s approval.

| Change after B’s approval | Source still free | Outcome when A approves the original two cases |
| --- | --- | --- |
| A customer reserves one remaining sealed case. | 12 bags: one free sealed case and six loose bags. | Send to review. A gains zero confirmed incoming. |
| A worker opens one remaining case and places all six bags in loose stock. | 18 bags: one free sealed case and twelve loose bags. | Same result, even though the loose bags alone could cover twelve. |

To repeat, label the three cases C1–C3, commit C1 to B, then either commit C2 to the customer or open C2. Retry A’s original approval twice. B should have six bags incoming and A zero. Show A’s manager “Requested 2 intact cases (12 bags). Available 1 intact case (6 bags).” Let the manager keep the full request pending or explicitly approve one case and see a six-bag shortfall against the original request. Reusing the previous approval ID for the changed quantity returned a conflict in the fixture. A new one-case approval, retried twice, committed C3 once. Both stores then had six incoming and the warehouse still held all 24 bags pending dispatch.

The two runs used named packs and sequential decisions in memory. In a live approval, recheck the destination need, source commitments and intact eligible packs together when recording the decision. If the revised offer changes again, show it for review. The exercise validates the distinction between bag count and eligible sealed cases. Database isolation and the merchant’s willingness to accept a smaller transfer still need a pilot test. [Intact packs and base units](catalog-and-units.md)
