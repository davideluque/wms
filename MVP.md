# Retail warehouse MVP

A small or medium Venezuelan merchant sells through a checkout, an online shop, or both. Staff should be able to identify the order that created demand, the location that committed stock and the person or carrier who received it. The pilot should prove this across one warehouse and a small number of stores. An ice business, a farm selling packed produce, and a clothing shop are candidate discovery partners. Their goods, units and workflows will test how broadly the same inventory model applies.

## What the merchant can do

| Job | First usable version |
| --- | --- |
| Start with believable stock | Import products, external SKU mappings, units and opening counts by location. Preview duplicates and unknown mappings. Reconcile totals with the merchant before enabling promises. |
| Receive and correct goods | Record the actual quantity received, put damaged units on hold, count a discrepancy and post a traceable correction. Show available, reserved, held and in-transit quantities with their location and meaning. |
| Capture a sale once | Import an order from one pilot sales channel. Support a simple manual order for a shop without an integration. Keep the originating channel, order and line references. A retry of the same sale must be safe. |
| Choose a source and reserve | Use the selected pickup store or a merchant-defined dispatch priority. Reserve at one eligible location when the whole order fits. Route shortages to a person. Record overrides and release cancelled commitments. |
| Hand over an order | Let staff pick and confirm quantities, then either mark the order ready for store pickup or request a carrier pickup. Record actual handover, recipient or carrier reference, and time. A carrier booking and a physical dispatch are separate events. |
| Replenish a store | When available stock falls below a configured minimum, suggest a transfer toward a target level, accounting for confirmed incoming supply and stock at the source. An operator approves, dispatches and receives the transfer. Keep shortages visible between sites. |
| Act on exceptions | Give the responsible person one actionable notification for a low-stock condition, short pick or failed handoff. Link it to the order or transfer. Acknowledging the notice and resolving the underlying work are distinct actions. |
| Connect another system | Offer a documented inventory read, an idempotent order submission, order status lookup and signed stock or fulfillment webhooks. Include examples for duplicates, delayed delivery and reconciliation. |

## Which system owns which fact?

The sales channel owns the commercial order and payment. The WMS records stock movements, reservations and physical handoffs. A carrier supplies collection or delivery evidence where its service supports it. The billing system owns fiscal documents. Keep a common reference across them so staff can explain a sale end to end. Confirm these boundaries with the first merchant, particularly a POS that already deducts stock.

## Pilot scope

Start with one merchant, countable goods with known units, one real sales integration and one delivery-company handoff. Shopify and AROA POS are candidate channels. Select the first from merchant usage and access to test transactions. Validate one carrier's booking or manifest flow and provide manual entry of a collection reference when its system is unavailable. Pickup and store transfers should still work for a merchant with only physical stores.

For launch, staff approve replenishment, choose how to handle an order that cannot ship from one location, and review returns before releasing stock. Carry split shipments, predictive purchasing, route optimization, marketplace connectors and sector-specific traceability into later releases when a pilot demonstrates demand. A merchant whose goods require unsupported controls calls for a different pilot scope.

## Acceptance walkthrough

1. Open with eight saleable units in the warehouse and two in Store A. An online order for three routes to the warehouse. Warehouse available falls to five and physical stock remains eight. Import the sale twice and check that only three units are reserved.
2. A customer buys one unit at Store A and takes it home. The POS sale and handover reduce store stock to one. With a minimum of two and a target of four, the system suggests transferring three. The manager approves once. Two remaining warehouse units are available after both commitments.
3. Dispatch the transfer, restart the receiving device, then receive only two units at the store. Store A now has three. One remains in transit pending investigation. Keep the transfer open and explain every unit at shift change.
4. Book the online order with the carrier. Retry after a lost response. Confirm one collection and dispatch the three reserved units once. A customer-delivery claim needs evidence from the carrier or an authorized manual confirmation.
5. Run a separate order for store pickup. Verify that stock promised to the customer is protected from another sale, that “ready” survives shift change and that cancellation before collection releases it. Replay the case with a short pick and a returned item awaiting inspection.

Capture the merchant's current inventory discrepancies, time spent re-entering sales, and time from sale to ready or dispatch. Agree pilot targets after measuring the baseline. Ask a second-shift worker to explain each remaining balance and open exception using only the records.

[Research notes and open investigations](research/README.md) · [Product principles](PRINCIPLES.md)
