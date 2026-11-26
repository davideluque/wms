# One outfit on the receipt, several items on the shelf

Ask the merchant to show how staff pick a set and what they do when a customer returns one piece. “Combo” on a sales receipt can mean a price promotion on items scanned separately, an outfit assembled to order, or a sealed box that already exists in stock. The same shirt can appear in all three arrangements. The physical workflow determines which balance is available to promise.

| What the picker finds | Inventory treatment to rehearse |
| --- | --- |
| A sealed six-pack of identical bags | Use the verified pack factor for one variant. If the merchant promises an unopened pack, check how many intact packs remain. [Catalog packages and units](catalog-and-units.md) |
| Separate garments grouped at checkout for a discount | Pick and reserve the component lines with the order. Keep the promotion reference for customer service. The sales channel determines prices and credit. |
| An outfit assembled on demand from one blue M shirt and one belt | Keep one commercial group with its exact component variants and quantities. Reserve the components together at the selected location. One outfit label can help staff look up the pick. Scanning that label cannot establish which pieces entered the bag. |
| A sealed, stocked gift box containing a shirt and a belt | Count an intact box as one stocked presentation. Keep its contents out of loose availability. If a merchant assembles a box from loose stock, record the assembly, the consumed components and the resulting box together. Opening one needs a recorded unpack and verification of contents and condition before loose units become available again. |

Odoo documents a kit that appears as one sales line with separate components on the delivery order. Shopify also distinguishes the group presented to the customer from individual items in fulfillment and returns. Confirm the merchant's channel and physical process before choosing a mapping. [Odoo kits](https://www.odoo.com/documentation/19.0/applications/inventory_and_mrp/manufacturing/advanced_configuration/kit_shipping.html) · [Shopify bundle inventory and returns](https://help.shopify.com/en/manual/products/bundles/eligibility-and-considerations)

## Does the pickup store have a complete outfit?

For a fixed set, at each eligible site divide each component's available quantity by its required quantity, round down and take the smallest result. Combine repeated requirements for the same variant before dividing. Exclude stock on hold or already promised. A count of buildable outfits is an estimate at that instant. The shirt is still physically the same unit that a customer could buy separately.

| Available blue M shirts / belts | Store A | Store B | Complete outfits for immediate pickup |
| --- | --- | --- | --- |
| Stock split across stores | 2 / 0 | 0 / 2 | Zero at either store. Adding chain-wide quantities incorrectly suggests two. |
| One belt physically transferred from B and received at A | 2 / 1 | 0 / 1 | One at A. During transit, neither store could offer a complete outfit. |
| Store A reserves the outfit, then sells its other shirt separately | 0 / 0 available | 0 / 1 available | The reserved shirt and belt remain at A until handover. Each later component handover is posted once. |

At pack time, show the shirt's colour and size and the exact belt variant. If the belt is missing, keep the case open and show the missing line. The customer can wait or agree to a changed order through the channel. Keep the original promise, the agreed change and what actually left. If the catalog later substitutes a different belt in new outfits, the old order and any return still refer to the original belt.

**Three listings can compete for a single belt.** Put one blue shirt, one red shirt and one belt in a store. A blue outfit, a red outfit and a loose belt each show one available when viewed separately. Enumerating the eight baskets that request zero or one of each listing gives four feasible baskets, including the empty basket. Every two-listing combination asks for the same belt twice. Recheck all component claims together on reservation. If a channel independently admits orders against earlier listing counts, surface the resulting shortage and assign the affected order. [Concurrent component reservation trial](../architecture/inventory-verification.md) · [Shopify fixed bundle checks](../integrations/shopify.md#can-this-store-pick-and-return-a-fixed-bundle) · [GS1 identification for sets and packs](../standards/identification-and-exchange.md#physical-assortments-virtual-sets-and-packs-of-one-sku)

## When only the belt comes back

After A hands over the outfit, its shelf has no shirts and no belts. B still has one belt. The customer returns only the belt at B. Receive the identified belt into hold at B under its original fulfilled component and order group. B now has **one belt available and one held**. The shirt remains with the customer. A full refund from the channel, if the merchant permits it, adds zero shirts or belts. If inspection releases the returned belt, B has two belts available and still cannot promise an outfit. Replaying the return updates no balance a second time. If the merchant instead supplies an unopened stocked box, inspect it under the merchant's sealed-box procedure before offering it again. [Return custody and commercial decisions](returns-and-corrections.md)

**Pilot boundary.** A merchant selling countable items and packs of one variant can use the existing catalog proposal. Ask a merchant who sells sets for one checkout receipt, its component pick, the location of each piece, and a partial return. If the checkout sends both a group and its components, preserve their relationship and pick the components once. If only a parent code arrives and staff cannot establish what that order contained, put it into review before reserving any units. Selecting automated kit support, assembly, substitutions or partial fulfillment needs evidence from that merchant. [MVP scope](../../MVP.md) · [Shopify connector checks](../integrations/shopify.md)
