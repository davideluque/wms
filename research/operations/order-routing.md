# Choosing a dispatch location and protecting a pickup

“Which store has one?” and “Which store can fulfill this sale?” can have different answers. A store may have a shirt on a return shelf, a bag of ice already reserved for someone, or a parcel ready for a carrier that cannot serve this address. The customer also may have chosen a specific pickup store. The pilot needs an explainable decision and a way for staff to change it safely.

Shopify lets merchants configure location ranking and rules to minimize split fulfillments. It distinguishes stock at the location of a POS sale from locations enabled for online fulfillment. A connected checkout can therefore make its own assignment before the WMS receives the order. Establish which system owns routing for each channel. [Shopify order routing](https://help.shopify.com/en/manual/fulfillment/setup/order-routing/understanding-order-routing) · [Shopify fulfillment by location](https://help.shopify.com/en/manual/inventory-and-locations/locations/fulfillment)

## Candidate policy for a small merchant

| Sale type | Source and action | Exception with an owner |
| --- | --- | --- |
| Counter sale, item handed over | Use the actual selling location. Import the handover once, even after the register reconnects. | If the WMS already promised those units, record the sale and raise the conflicting commitment for the store lead. Preserve the historical handover. |
| Customer selects store pickup | Restrict to that store. Reserve confirmed eligible units there. Tell staff to stage the order, then notify the customer after the complete order is physically ready. | If stock is missing, ask the store to recount or arrange a transfer. Renegotiating the store or promised time needs customer follow-up. |
| Shipping, channel has assigned a site | Check the assignment, item mapping, site capability and available quantity. Link the channel's fulfillment line to the WMS reservation. | If the assigned site cannot honor the order, record the reason and coordinate one reassignment with the channel. Withhold the new pick until its authority is clear. |
| Shipping, WMS owns the assignment | Filter to merchant-enabled sites that can serve the destination, handle the item and fulfill every line from eligible stock. Use an explicit site priority. Recheck quantities together when reserving. | When no single site fits, present the candidate stock and reason. Staff may arrange a transfer, get approval for a partial shipment or cancel the unmet demand. |

Use a stable tie-breaker within a configured priority. Show the accepted rule, policy version, item and site availability at decision time, plus any reason for rejecting a higher-priority site. For a manual reassignment, verify the first site's picking and carrier state. Release and replace a reservation together when the WMS owns both. If a package is already on a truck, start a recovery or return workflow.

## A two-line counterexample

Suppose an online order contains one blue medium shirt and one belt. The warehouse has three eligible shirts and zero belts. Store A has one of each and offers shipping. Store B has two belts and handles pickup only.

1. For shipping, Store A can fulfill the entire order. A rule that chooses the highest shirt count first would choose a site that cannot finish the job.
2. If another buyer reserves Store A's only belt after ranking, the stock transaction must recheck both lines. For the proposed all-or-nothing pilot, a failed attempt leaves neither line reserved by this order. The order enters the exception queue with the failed item and site.
3. If the customer chose pickup at the warehouse, Store A's stock is a possible transfer source. It cannot make the order ready at the warehouse. Reserve or protect agreed supply, then receive, stage and verify the complete order before sending a ready message.
4. Replay the same case with one site holding returns or a stale offline register. Display what is known about stock freshness, which order is exposed, and who can resolve the discrepancy. A central transaction can prevent competing WMS commitments. It cannot retract goods that an independent offline checkout physically handed over.

## Evidence for a pilot

Replay past orders in their original arrival order, including cancellations and late register uploads. Compare proposed source with the actual dispatch site. For each difference, ask about opening hours, service areas, pickup preference, store stock reserved for walk-ins, cold handling and staff capacity. These should become configurable constraints only where they change real routing decisions. For ice or fresh produce, a normal parcel service requires an explicit suitability check with the carrier and merchant before offering delivery.

Measure orders reassigned after release, promises the site could not fulfill, unnecessary transfers, and the interval between order and physical handover. Separate store pickup from shipped orders. Validate that an employee can explain why each order was routed and why an apparently available unit was excluded. [Retail MVP](../../MVP.md) · [Shopify connector](../integrations/shopify.md)
