# Picking efficiently in a small warehouse

Before adding batch picking, walk an ordinary order with the picker. Time the walk, the search for the right item, the quantity check, and the return to staging separately. If the delay is finding the correct shirt size, a better route will still take the worker quickly to a confusing shelf. If the location is clear but staff cross the same room all morning, route order or grouping adjacent orders is worth testing.

Petersen and Aase evaluated combinations of storage, routing and batching in a simulation of manual bin shelving. Small orders made batching especially influential in their modeled conditions. In a separate case at Ankor, researchers studied storage assignment together with routes and reported shorter walks after adapting both to that warehouse. Use these studies to choose what to measure. A Venezuelan shop with a short aisle, a narrow freezer door or customers waiting at a counter needs its own comparison. [Petersen and Aase, manual picking policies](https://www.sciencedirect.com/science/article/abs/pii/S0925527303002937) · [De Koster and colleagues, Ankor case](https://pubsonline.informs.org/doi/abs/10.1287/inte.1040.0083)

## A first version a picker can explain

For an assigned order, show the next location, exact variant or lot when required, quantity and unit, and the place to stage the completed order. Let the site assign a simple walk sequence to locations. Bin names are useful to staff but lexical sorting can put A-10 before A-2. A worker should be able to skip an inaccessible location with a recorded reason and come back to the open line. Preserve the order's reservation when a skip only changes the walk.

Confirm the item actually picked. At the end, show the staged package and the lines still outstanding. In a clothing shop, a different size is a different item. In food distribution, a lot already on hold is ineligible regardless of how convenient its bin is. A proposed replacement or alternate lot must be checked against the current reservation, item rules and customer commitment.

## When to test batching

In a hypothetical straight aisle, staging is at metre 0. Order A needs one item at metre 20, and order B needs another at metre 25. Separate return walks total 40 + 50 = 90 metres. One trip reaching metre 25 and returning is 50 metres. The 40-metre saving assumes the picker can carry both, both orders are ready to work, and the items stay separated. It says nothing about total fulfillment time until we include waiting, scanning and sorting.

Use separately identified totes or bags for the trial. Check each item against its order before marking either complete. Reverse the order of the trials across comparable work periods to reduce practice effects. Measure walking time, time from each order's release to readiness, wrong-order items, short picks and help calls. An urgent pickup may be ready sooner if staff start immediately rather than wait for a batch. A second order is a benefit only when the added coordination pays for itself.

## Evidence that changes the design

| Floor observation | Low-cost change to try | Result to inspect |
| --- | --- | --- |
| Staff search among neighboring variants | Improve shelf identification and show the distinguishing attribute in the pick step | Wrong-variant finds and time from reaching the bin to confirming the item |
| Most of the walk is repeat travel for joint-ready orders | Trial a two-order walk using separate totes | Per-order readiness, walking time and sorting errors |
| Fast sellers are far from staging | Trial a small pick face replenished from reserve stock | Walking saved versus extra replenishment trips, stockouts and count errors |
| The worker arrives at an empty pick face despite a balance | Capture actual find, site/bin, outstanding reservation and any alternate stock | Time to resolution and whether the next picker receives the same bad instruction |

Start with single-order picking in the pilot. The merchant can justify more complex routing with a measured problem, specific expected benefit and a trial that preserves each order's identity. [Warehouse operations](stock-and-workflows.md) · [Floor UX](../experience/floor-workflows.md)
