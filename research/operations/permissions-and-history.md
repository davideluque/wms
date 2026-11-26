# Who may change stock, and how the next shift can tell

A clerk at Store A can report finding a damaged item even if only the owner can write it off. Preserving that observation lets staff record reality while an approval is pending. Design access around a merchant, permitted locations, action and current work state. The same checks belong at every entry point, including a scanner, an import and a partner API. [OWASP authorization guidance](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)

## Starting responsibilities to test

| Work | Who might do it at a small business | Evidence to keep |
| --- | --- | --- |
| Receive, pick or count at the assigned site | Store or warehouse worker | Own identity, observed quantity and unit, task and site, observation and posting times |
| Quarantine a damaged item | Person who found it, where merchant policy permits | Actual location and quantity, condition and reason. The hold must affect allocation immediately once accepted. |
| Release held goods or approve an adjustment | Named role authorized for that item, reason and location | Decision, discrepancy it resolves, quantity, approving actor and policy applied |
| Dispatch or receive a transfer | Source worker confirms departure. Destination worker confirms arrival. | Each actor confirms its own site, timestamp and quantities. A short receipt creates unresolved in-transit stock. |
| Change a pack conversion or approve opening balances | Catalog or stock owner | Prior definition, new effective revision, sample that establishes the conversion and affected open work |
| Connect a checkout or carrier | Merchant administrator grants a scoped integration identity | Installation and permitted sites/actions, changes in access, last successful use and revocation |

These are responsibilities, so one owner-operated shop can assign several to one person. Some merchants may choose a second approver for high-value losses or regulated goods. Ask the owner how often those cases arise and what happens when they are alone. Shopify provides an example of POS actions that can be allowed, denied or approved by an authorized manager at supported locations. [Shopify POS permissions](https://help.shopify.com/en/manual/your-account/users/roles/permissions/pos-permissions)

## Reconstruct one correction

At 09:15 a receiver says twelve shirts arrived. At 10:20 a picker finds eleven and records the observed bin, size and count. The supervisor checks for a movement between the two times. Suppose a checkout sold one shirt at 09:45 and later synced. The original receipt stays at twelve. Link the late sale and the review outcome to the discrepancy. If the supervisor instead verifies an eleven-shirt receipt and no intervening movement, post a linked adjustment of minus one with the observation and approver. A physical count alone cannot establish which explanation is correct.

Give staff a business history they can read by order, movement or item. Keep the source reference, actor, relevant site, command ID, effective business time, recorded time, prior decision, resulting quantity and unit. Choose which roles can see a delivery recipient or contact detail. An inventory reviewer usually needs a handover reference. Technical support can investigate delivery attempts using correlation IDs. Keep secrets and unnecessary customer details out of diagnostic logs, and control access to records containing personal data. [OWASP logging guidance](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)

## Shared-device rehearsal

Have a receiver record a shortage on a shared handheld. Switch users while offline and confirm the queued observation retains its original identity and site. Reconnect after the original worker's permission changes. Preserve the observation and report the server's decision on the stock command. A manager can review it with their own authorization. Repeat with an online read-only integration and a worker at another store: both should receive only the data and actions granted to them. Finally, have the owner explain the quantity, reason and person responsible using the ordinary history screen.

Retention periods, duties that must be separated and evidence required for a particular item or fiscal process need confirmation with the merchant and its advisers. [Local discovery](../venezuela/local-discovery.md)
