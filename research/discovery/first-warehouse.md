# Choosing and learning from the first warehouse

**Decision to make.** Pick a customer reachable for repeated observation, with a recurring warehouse problem, an operator willing to test changes and a buyer who can explain the cost. A Venezuelan distributor managing its own stock is one candidate. A third-party warehouse managing inventory for multiple owners would introduce a different question immediately: whose goods can this order consume?

## Follow one shipment

Ask a receiver to show a recent ordinary delivery from its first document to the moment somebody else relied on the stock. Ask them to replay the last delivery that went wrong using the records they normally consult. Observe at the point of work where possible. Research in familiar surroundings reveals how people use their own tools and handle interruptions. [GOV.UK guidance on contextual research](https://www.gov.uk/service-manual/user-research/contextual-research-and-observation)

| Person | Ask them to show | Capture |
| --- | --- | --- |
| Receiver | “Which mark tells you this is one case of twelve? What do you do if the mark is missing?” | Package, label, unit, device and fallback |
| Stock controller | “Show me the last time the screen and shelf disagreed. What could still move while you investigated?” | Count time, concurrent work, correction and approval |
| Picker or dispatcher | “When could you promise this item? When could you physically take it?” | Reservation, release conditions, location and stale information |
| Buyer or administrator | “Show me how you learned that the delivery was short. Which reference let you follow up?” | Supplier document, re-entry, open quantity, acknowledgement and closure |
| Site manager or owner | “Who pays for these discrepancies today? Which improvement would justify switching?” | Cost, frequency, person with budget, setup constraints and success measure |

Request redacted examples of the expected order, the label and the resulting receipt. Keep their common line reference, units, timestamps and quantities consistent across examples. Record local terms in the speaker's Spanish and ask a second role to explain them. Two people may use “disponible” for different decisions.

## Leave with a replayable case

Use one row per observed step: **who acted → what they saw → what they did → what changed physically → what the system recorded → who used the result**. Include manual work. Ask the operator to correct the reconstruction. Mark separately what someone remembers, what a document establishes and what remains unclear.

For the example of a short delivery, measure the time from physical receipt to stock becoming usable, and from finding the shortage to the buyer having enough information to act. Record lines handled and how many needed re-entry so that a busier day is interpretable. Agree on a target with the buyer after seeing the baseline.

An initial prototype is worth building when the team can replay the ordinary case and its exception, name the next person affected, and identify a measurable improvement. If uncertainty remains over who can release stock or who owns discrepancies, another conversation with that role is likely to teach more than a broader feature survey.

## Can last month’s sales justify a replenishment threshold?

Pick one store, SKU and route. Mark when the shelf was empty, when each register sale became visible centrally, and when transfer requests had to reach dispatch. A retailer that sold its last four bags by noon has evidence of four sales. The register alone cannot reveal how many afternoon customers left or bought a substitute. Record denied requests separately and label the remaining demand unknown. Research on vending-machine stockouts found that substitution can also inflate sales of the alternative item. [Anupindi, Dada and Gupta on stockouts and substitution](https://doi.org/10.1287/mksc.17.4.406)

Rehearse the actual decision time. Suppose a Friday 09:55 sale synced at 10:20, after a 10:00 route cutoff. A replay that gives the manager that sale at 09:55 invents an opportunity to load Friday’s truck. Use only information then visible, including any documented phone call. Check the next actual truck and the store’s receiving hours to establish when stock could become usable. [a2 on delayed branch synchronization](https://a2.com.ve/producto/13-manager-punto-de-ventas/) · [Dynamics planning with shipping and receiving calendars](https://learn.microsoft.com/en-us/dynamics365/supply-chain/master-planning/supplier-requested-confirmed-dates)

Compare candidate minimums on later, held-out shifts. Show actual stockout hours, trips and units held alongside the simulated outcomes for each minimum. Try a range of plausible unmet requests during empty-shelf periods. If the preferred threshold changes across that range, use manager-approved suggestions and collect another route cycle of evidence before automating it. [Store replenishment rule and trial](../operations/store-replenishment.md)

## How to tell whether the pilot helped

Agree on the window, denominator and evidence for each measure with the merchant before comparing results. Record a baseline on the same kinds of shifts and report the number of observations beside every percentage. A first week with more reported discrepancies can mean that staff finally have somewhere to record them. Inspect the cases before attributing a cause.

| Operational question | Proposed pilot measure | Interpretation to check |
| --- | --- | --- |
| Can the picker trust a location? | Counted variant–bin–condition records matching the reconciled physical count divided by all records counted. Include unexpected items and explicit zero counts. | A correct site total can conceal stock in the wrong place. Report count scope, date and disputed movements. |
| Can the store keep its promises? | Original order lines fulfilled in full by the agreed service cutoff divided by lines due in that window. Keep customer-approved substitutions, cancellations and partials identifiable. | APQC's order-line fill rate measures complete lines. The cutoff is our proposed pilot addition. Publish raw line counts and separate pickup from shipping. [APQC order-line fill rate](https://www.apqc.org/what-we-do/benchmarking/open-standards-benchmarking/measures/order-line-fill-rate) |
| Did automation save the employee work? | Minutes of active handling and repeated manual entries per completed sale or transfer. Also record elapsed time to resolution for exceptions. | A faster happy path can shift work to the next person. Measure the buyer, store clerk or dispatcher who actually closes the exception. |
| Does the next shift inherit a usable record? | Ask incoming staff to identify the unresolved quantity, physical custody, next action and authorized owner from the WMS. Record successful cases and time taken. | Include at least one short receipt and one delayed digital acknowledgment. A green notification cannot establish physical handover. |

**A misleading total.** The ledger shows four medium shirts in bin A and one in B. A reconciled count finds two in A and three in B. The site total is five on both sides, yet zero of two bin records match and a picker sent to A for three shirts will be short. Two misplaced shirts account for both differences. Summing the absolute bin variances gives four units of discrepancy across records, which would overstate how many physical shirts need moving. Follow the movement evidence before posting a loss. [Cycle counting](../operations/cycle-counting.md)
