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

## When a pickup is overdue or a bag is missing

Shopify POS describes setting goods aside, notifying the customer and marking pickup after collection. Shopify’s cancellation workflow separately offers refund choices and an inventory restock option. Agree which system updates sellable stock before enabling a connector. A channel’s automatic restock can otherwise offer an item that staff are still trying to find. [Shopify POS pickup workflow](https://help.shopify.com/en/manual/sell-in-person/shopify-pos/order-management/pickup-in-store-for-online-orders) · [Shopify cancellation, refund and restock options](https://help.shopify.com/en/manual/fulfillment/managing-orders/canceling-orders)

**Proposed pilot policy.** The merchant chooses the pickup window, the event that starts it, store hours and time zone, who may extend it, and who contacts the customer. Save the promised deadline on the order. If the window starts at successful notification, a failed or unconfirmed send goes to staff for follow-up. The sales channel owns deposits, refunds and commercial cancellation. The WMS owns the short count, physical handover and put-back evidence.

| At the counter | Operator decision | Stock and customer outcome |
| --- | --- | --- |
| Deadline passes with goods still staged | Open one overdue case for the shift lead. Verify both bags, check the last customer contact and record an extension or send the order to the channel for cancellation under merchant policy. | Keep the staged units protected while the decision is pending. An extension updates the deadline and reminder once. Replaying the timer changes no stock. |
| Customer presents an old “listo para retirar” message | Look up the current order and revision. If preparation, location or promise changed, explain the current status and have the assigned clerk contact the customer. Suppress queued notifications for superseded revisions. | A previously sent message remains in the history. A pickup after cancellation or put-back needs a newly authorized promise and a fresh availability check. |
| Only one of two labeled bags can be found | Scan the bag found, mark the other as unlocated and search the staging and handover records. Suspend the ready status and assign a discrepancy. | Keep both units unavailable pending resolution. If the merchant and customer agree to partial collection, confirm only the actual bag and track the outstanding line separately. |
| Channel confirms cancellation while staff are putting goods away | Claim the task against the current order revision. Scan each recovered bag, verify condition, and record the actual destination bin. Hold damaged goods for review. | Release each verified sellable quantity once. Reconcile any channel restock already posted. If handover won a concurrent update, follow the return workflow for goods that left. |

**Two-bag check.** Start with eight shirts at Store A. Staff stage two shirts in separately labeled bags for order 514, leaving **6 available + 2 staged**. After a missed pickup, only bag 1 can be found. When the channel confirms cancellation, staff verify bag 1 is sellable and return it to its bin. The ledger now shows **7 available + 1 unlocated**, with zero handed over. A refund or repeated cancellation leaves that balance intact. Finding bag 2 and scanning it back to the bin restores eight available. If an authorized count instead confirms a loss, close at seven available with a linked one-unit adjustment. [Stock permissions and corrections](permissions-and-history.md) · [Pickup labels and multi-bag checks](../experience/labels-and-printing.md)

## Evidence for a pilot

Replay past orders in their original arrival order, including cancellations and late register uploads. Compare proposed source with the actual dispatch site. For each difference, ask about opening hours, service areas, pickup preference, store stock reserved for walk-ins, cold handling and staff capacity. These should become configurable constraints only where they change real routing decisions. For ice or fresh produce, a normal parcel service requires an explicit suitability check with the carrier and merchant before offering delivery.

Measure orders reassigned after release, promises the site could not fulfill, unnecessary transfers, and the interval between order and physical handover. Separate store pickup from shipped orders. Validate that an employee can explain why each order was routed and why an apparently available unit was excluded. [Retail MVP](../../MVP.md) · [Shopify connector](../integrations/shopify.md)
