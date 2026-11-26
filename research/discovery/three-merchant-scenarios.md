# Could an ice seller, a farm and a clothing retailer share one WMS?

**Yes, if the job starts when identifiable goods enter stock and ends when someone takes custody of them.** The same system can receive an item, put it somewhere, reserve it, record what actually left and explain a difference at count. A bag, a weighed crate and a shirt size need different definitions of “one.” That choice affects the stock record and the receiving screen.

All three examples are hypothetical. The merchants’ products, facilities, sales channels, equipment and procedures are unknown. Start each interview with a recent transaction and ask the merchant to show how it was recorded.

## Ice seller — count a bag or weigh the ice?

Suppose a freezer receives 60 sealed bags labeled 5 kg. A shop receives 18, and two split bags are discarded. The freezer should show **40 bags available**. Record the two damaged bags and the reason for disposal. If they are still physically in the freezer awaiting inspection, show 42 on hand, of which two are on hold and 40 are available. The 5 kg label identifies the product. Counting the bags provides no independent measurement of the weight actually received or lost.

If customers instead buy loose ice by weight, a dispatch of 18.4 kg needs a quantity in kg at the agreed precision. If the merchant both counts sacks and bills by measured weight, preserve both observations and specify when weighing occurs. Treating every sack as its nominal weight would misstate that transaction. A warehouse implementation of this dual-unit model records both the handling unit and measured weight. [Dynamics 365 catch-weight processing](https://learn.microsoft.com/en-us/dynamics365/supply-chain/warehousing/catch-weight-processing)

**Ask the seller.** “Show me yesterday’s delivery and the last time ice melted or a customer rejected a bag. At which point did it leave your stock? Does anyone weigh it? Is stock loaded onto a truck still yours?” If the truck holds saleable inventory, it may warrant a stock location. Route optimization, freezer telemetry and the process of making ice are separate product choices. Finished bags could enter through a receipt from production without scheduling a freezing run.

## Farm — trace a crate from receipt to customer

Suppose a farm sells packed tomatoes. A stockroom receives four reusable crates under lot H7. Their measured net weights are 11.0, 11.4, 12.2 and 10.9 kg, totaling **45.5 kg**. Two crates weighing 11.4 and 12.2 kg go to customer A. The WMS links the dispatch of **23.6 kg** to H7 and leaves two crates containing **21.9 kg**. If H7 is later held, staff can locate the remainder and identify the shipment to A. Returning an empty crate changes the container balance only if the merchant tracks returnables.

Lot tracking, container tracking and weighing answer different questions. A farm selling sealed, fixed-count boxes might need only boxes and a lot. A packhouse that combines harvest lots and relabels the output needs input-to-output lot links to answer the same trace query. GS1’s produce guidance distinguishes growing, packing and shipping and illustrates links between packing runs and shipments. [GS1 fresh fruit and vegetable traceability guideline](https://ref.gs1.org/guidelines/fruit-veg/)

**Ask the farmer.** “What was the last product you shipped? Show the label or note that connects it to its source. If I bring it back, how would you find other affected customers? When are weights, grades and losses known?” Establish whether the warehouse receives harvested goods or finished packs. Field planning, irrigation, crop inputs and estimating future yield serve additional farm-management jobs. Repacking with lot genealogy is a separate decision if the WMS must cover the packhouse.

## Clothing retailer — stock the exact sellable variant

Suppose an order contains six navy shirts, three medium and three large. The receiving worker counts two medium and four large. Record those actual quantities against distinct variant identities and flag both differences even though the carton total is correct. After one large sells, large on hand is **three**. If the customer returns that shirt with a stain, large on hand becomes **four**, with **three available** and **one awaiting inspection**. A style total alone could hide the receiving mistake and permit a sale in the wrong size.

An internal variant identifier is enough to try this workflow. If the merchant uses GS1 identifiers, each size and color combination has its own GTIN. Map an existing barcode to the variant and carry the style as a reporting group. RFID, assortment packs and marketplace availability become useful only if the merchant can show a job they address. [GS1 guidance on size and color variants](https://support.gs1.org/support/solutions/articles/43000734083-how-many-gs1-gtins-do-i-need-when-i-have-a-product-with-many-sizes-and-colours-)

**Ask the retailer.** “Show two variants people confuse, a recent exchange and a sale made while a transfer was in progress. Which channel promised the item, and when did you make the returned garment sellable?” A point-of-sale or store platform can supply the sale and return references. Checkout, payment refunds and buying a seasonal assortment can stay with the systems that already handle them.

## Choices for the product owner

| Decision | Evidence to collect | Consequence |
| --- | --- | --- |
| Which merchant and workflow lead the pilot? | Observe one receipt, one issue and one discrepancy. Compare frequency and cost of today’s errors. | Pilot a complete job in one operation. Reuse its stock commands in a second scenario before expanding coverage. |
| Which dimensions are required per item? | Inspect real labels, units, scales, lot or variant identifiers and the precision used on paperwork. | Configure count, measured quantity, lot and variant per item. A bag counter should see a bag count, while a weighed dispatch prompts for kg. |
| Where does the WMS take custody? | Trace a production or harvest handoff, vehicle load, retail sale or return to a named person and record. | Decide which external references are enough and whether weighing, packing genealogy, vehicle stock or sales-channel synchronization is required at launch. |
| What evidence establishes a shared product? | Reconcile each walkthrough to physical stock by item, location, unit and any relevant lot, variant or hold. Ask the merchant to correct one realistic mistake. | Promote a capability into the common workflow when another operation uses the same meaning and transaction. Keep sector-specific steps optional and examine any proposed production or farm-planning expansion separately. |
