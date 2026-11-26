# AROA POS as a warehouse sales channel

AROA is a candidate for merchants already using its checkout. A bag of ice sold over the counter makes a good first example: one menu item could map to one stocked pack. A prepared meal with modifiers raises different questions about recipes, yields and ingredient stock. Begin with products the merchant can map unambiguously.

## What the source establishes

In the reviewed revision, an open check has a tenant, device, service type, status, version and modification history. Paying a held check writes a completed transaction and its link to the paid order in one local database transaction. Retrying that closure returns the existing transaction. A connector that imports the paid check and the completed sale independently could post one sale twice. The same revision also has a cash-sale test through the normal checkout. The pilot should exercise both checkout paths. [AROA order lifecycle](https://github.com/davideluque/aroa-pos/blob/9299ef0bf0803324a0d42c03276d956720abf950/src/features/orders/data.ts) · [Cash-sale test](https://github.com/davideluque/aroa-pos/blob/9299ef0bf0803324a0d42c03276d956720abf950/tests/e2e/cash-sale.spec.ts)

The transaction item records a menu item ID, quantity, sale-time name and price, with optional selected modifiers. The reviewed type lacks a separate immutable sale-line ID or warehouse location. The synced order carries a device ID and supports counter, dine-in and takeaway. Confirm physical handover and the store separately. A tablet moved between stores cannot reliably tell us the stock origin of an old sale from its current location. [AROA data types](https://github.com/davideluque/aroa-pos/blob/9299ef0bf0803324a0d42c03276d956720abf950/src/db/schema.ts) · [Order sync mapping](https://github.com/davideluque/aroa-pos/blob/9299ef0bf0803324a0d42c03276d956720abf950/src/sync/order-mappers.ts)

## Proposed first connection

| Fact to establish | Pilot decision |
| --- | --- |
| Which sale? | Scope transaction ID by installation and tenant. Attach the paid open-order ID when present. Keep transaction status changes linked to that same sale. |
| Which stock? | Let the merchant map menu item and any stock-relevant modifiers to SKU, pack size and effective dates. Preserve the mapping used on the imported sale. Send ambiguous or unmapped items to review. |
| Which store? | Capture the selling site at transaction time or supply a merchant-verified, time-bounded device-to-site assignment. Keep the original site when the device moves. |
| Which line? | Prefer source-issued stable line IDs for future edits or partial returns. For an immutable completed snapshot, store the original row positions and issue persistent WMS line IDs on first import. A changed or reordered snapshot needs reconciliation before another movement. |
| What physical event? | For a validated counter workflow where payment and handover coincide, import one handover. For advance payment, preserve demand and wait for collection. A void is a financial event. Require evidence of physical return before increasing saleable stock. |
| How does it reach the WMS? | Trial a merchant-authorized export with sample transactions, then assess a supported, replayable sales feed. Agree who owns publication, checkpoint and catch-up behavior. The reviewed source establishes a sync model, not an available public WMS connector. |

## Pilot proof

Run a direct sale, a held check amended then paid, and a cancelled open check. Sell two identical packs on separate lines. Take one transaction offline, close the shift, reassign the device to another store and reconnect. The WMS should post the originally confirmed site and quantity exactly once, preserve both source-line references, and show the age of the delayed sale. A stock promise made elsewhere while the device was offline may need review.

Void a previously handed-over sale with the goods still at the customer. Then process a separate physical return held for inspection. Staff should be able to explain the difference using the receipt and original movement. Validate against the deployed AROA version and merchant register practices. This note is a source review of revision `9299ef0`. No merchant installation, export or integration test was run.
