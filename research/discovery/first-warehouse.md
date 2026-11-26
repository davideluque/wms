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
