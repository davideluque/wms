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

## Notifications a manager can act on

Keep one active shortage per store, SKU and rule while the condition persists. Show available now, incoming with its ETA, minimum, target and source capacity. An acknowledgement records who saw the issue. A snooze has an owner and expiry. Close or revise the suggestion when stock changes, and link an approved suggestion to the transfer so the next shift can follow the goods. If confirmed incoming is overdue, flag the late transfer and recompute using the planning horizon.

During the pilot, replay real sales and transfer history chronologically using only facts known at each step. Compare suggestions with managers' actual decisions. Record stockout time during selling hours, transfers per week, declined suggestions and units still in transit after the expected arrival. Ask the manager which direction each metric should move and what extra transfer workload the store can absorb.

**Open decision.** Customer reservations, scarce stock distribution between stores, package multiples and supplier replenishment can each change an otherwise sensible answer. Begin with operator approval and make the suggested arithmetic visible. [Retail pilot decisions](../../DECISIONS.md)
