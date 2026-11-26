# A supervisor’s daily exception queue

Start the shift with **Necesita atención**. Each row should tell the supervisor what happened, which order or parcel is affected, who can act and when the next decision is due. Show the oldest unowned blocker and the nearest customer or carrier cutoff first. A shift with no known open issues can say “No hay pendientes registrados · última actualización 08:42”. Keep the timestamp visible when the connection drops.

Microsoft’s warehouse management documentation provides a precedent for the underlying record. A worker reports a short pick with the actual quantity, location and reason, and a manager can filter the resulting exceptions by status. GOV.UK’s design guidance recommends using notification banners sparingly and placing information relevant to the current task alongside that task. The proposed queue applies those ideas to a small team where the same person may receive, pick and supervise. [Dynamics 365 work exceptions](https://learn.microsoft.com/en-us/dynamics365/supply-chain/warehousing/work-exceptions-log) · [GOV.UK notification banners](https://design-system.service.gov.uk/components/notification-banner/)

## What earns an interruption

| Event | Operator or manager sees | Owner and next action |
| --- | --- | --- |
| Worker scans the wrong bin and can correct it on the spot | “Leímos B-02. Escanea B-03.” Keep the quantity and order on screen. | Picker retries or requests help. Open a case when the bin is inaccessible, the item is missing or the worker explicitly asks for assistance. |
| Pick short with a customer waiting | “Pedido 514 · faltan 2 unidades en A-03 · retiro a las 11:00”. Show **Asignar recuento** and **Revisar otra ubicación**. | Supervisor assigns one counter. Sales or the pickup clerk contacts the customer if the decision changes the promise. Only authorized staff approve the partial order or a stock adjustment. |
| Carrier booking result uncertain | “Bulto 718 · guía por verificar · recolecta a las 15:00”. Offer **Buscar guía existente** and **Registrar respuesta**. | Dispatch lead checks the existing request and records the guide or carrier response. Escalate to the shift supervisor at the agreed decision time if the pickup is at risk. |
| ERP rejected a dispatch confirmation | “Pedido 533 · el ERP rechazó la unidad CAJ · revisar equivalencia”. Show the physical handoff time, original transaction and **Reintentar envío al ERP** after correction. | ERP owner corrects the mapping and replays the same transaction. Dispatcher can see that the parcel has already left. Close after matching the external posting reference. |
| Device or site loses connectivity | Device shows “Sin conexión · 3 movimientos guardados en este equipo · último envío 10:06” only if durable local capture succeeded. Manager sees “Equipo 2 · última comunicación 10:06 · pendientes por confirmar”. | Device user can inspect the local queue. Supervisor checks the device at the handoff. The server can show its last confirmed count, but cannot know how many new actions a disconnected device captured. |

Use a visible status word, time and action for every state. Announce an updated count or completed assignment without moving keyboard focus. Reserve an interruptive announcement for a time-sensitive decision. A person using a screen reader should be able to finish reading the current row while another routine update arrives. [WCAG 2.2 status messages](https://www.w3.org/WAI/WCAG22/Understanding/status-messages.html)

## One case from report to handoff

- **Record once.** Persist a unique source event ID so replaying the same scan, callback or sync error cannot create a second case. Group repeat observations of the same unresolved condition by site, task or parcel, line and reason. Append each observation, its actual count and timestamp to the existing case. Two independent order shortages can share a bin investigation while keeping separate customer deadlines and resolutions.
- **Notify on a change that requires action.** Create a queue row once. Notify the assigned role once on assignment, once if the agreed response time passes and the backup must act, or when new evidence changes the required decision. Remember which recipient received which transition, including across retries and channels. Background refreshes and unchanged polling results leave the row visible without repeating the alarm.
- **Give one person responsibility.** Use **Sin asignar → En revisión → En espera → Resuelta**. **Tomar caso** records the owner and time and prevents a simultaneous second claim. Waiting requires a named dependency and next review time. Acknowledging an alert records that it was seen. Closing requires a reason and evidence such as a count, guide or ERP posting. If conflicting evidence arrives later, reopen with that evidence and preserve the history.
- **Hand over explicitly.** The outgoing worker records last confirmed physical quantity and location, pending digital actions, what has been tried, the next decision and its deadline. The incoming worker takes ownership under their own login. Until accepted, the outgoing owner or named shift supervisor remains accountable. Provide a supervisor reassignment if the outgoing worker has already left. Choose backup roles and deadlines from the merchant’s staffed shifts and cutoff times.

## A simulated shift

This tabletop example uses invented orders, times and roles. Replay it with a floor worker and supervisor before setting alert thresholds.

| Time | What happens | Expected observable result |
| --- | --- | --- |
| 08:10 | Ana picks four of six units for order 514 and reports a shortage. The handheld resends the event. | One case shows four picked, two missing. Luis claims it. Ana sees his name against her report. |
| 08:25 | Luis finds one more unit and asks sales to confirm a five-unit pickup. | The same case shows five staged, one outstanding, the sales contact and the review time. Pickup remains pending. |
| 10:06 | A handset disconnects after the server confirmed its last scan. Three later moves are stored locally. | The handset lists three pending items. The manager sees the last contact time. Reconnecting clears all three only after individual acknowledgments, with no extra stock moves. |
| 14:40 | A guide request times out. The next shift starts before the carrier replies. | Dispatch retains one case for parcel 718. The new lead accepts it and retrieves the existing guide. Physical handoff is recorded later against the collection receipt. |
| Close | The supervisor reviews unresolved, resolved today and overdue cases. | Every open item has a next owner or a visible unowned state, a due time and physical custody recorded where relevant. Late carrier or ERP confirmations update their existing cases. |

## Measures that change an operating decision

| Measure | What to check and who acts |
| --- | --- |
| Open blockers already past their response deadline, with oldest age | Supervisor reassigns the oldest unowned case or negotiates a new customer promise. Break down by shift and reason. |
| Median time to acknowledgment and closure by case type | Review time from first report to ownership and from report to verified resolution separately. Inspect long waits for an ERP owner, counter or carrier before adding alerts. |
| Duplicate notifications per recipient for one unchanged case transition | Compare notification history with case history. Proposed test target is zero duplicates after three simulated message replays. Track escalations and genuinely reopened events separately. |
| Handoffs missing a next action or owner / all open cases crossing shifts | Shift lead reviews incomplete handoffs before outgoing staff leave. During the trial ask incoming staff to identify the next action without calling their predecessor. |
| Conflicts or duplicate postings after reconciliation / replayed operations | ERP owner compares the WMS movement ID with the stock ledger and external reference. A duplicate posting triggers investigation even when all notifications were delivered. |

These measures are proposed pilot checks. Capture a baseline from the merchant’s current process before assigning production targets. For a stock correction, check the approval against the movement history. For a carrier timeout, the dispatch lead can use the original booking reference to investigate before retrying. [Stock permissions and correction history](../operations/permissions-and-history.md) · [Recovering an uncertain carrier booking](../integrations/venezuela-carriers.md#when-a-booking-times-out)
