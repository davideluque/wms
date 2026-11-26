# Race conditions in warehouse work

Two orders can each be valid when a clerk opens the screen and incompatible by the time both clerks press **Confirmar**. A race arises when the outcome depends on the order of overlapping actions. Start by naming the fact both actions change, the allowed outcomes, and the evidence the losing worker needs. A unique request ID handles a retry of the same intent. Different customers can legitimately send distinct requests for the same last item.

## A small experiment

Suppose a store has one bag. Each checkout reads availability, remembers whether it could sell the bag, then attempts to reserve. The following Python 3 experiment enumerates all six legal interleavings of those four steps. It compares a system that trusts the earlier read with one that checks eligibility and decrements as a single indivisible action. The counter counts accepted customer promises. Keeping only the final bin quantity would hide a lost update in systems that overwrite the same value.

```python
from itertools import combinations

def simulate(first_worker_steps, guarded):
    stock = 1
    promises = 0
    seen = {}
    for step in range(4):
        worker = "A" if step in first_worker_steps else "B"
        if worker not in seen:
            seen[worker] = stock
        elif (stock if guarded else seen[worker]) > 0:
            stock -= 1
            promises += 1
    return promises

schedules = list(combinations(range(4), 2))
for guarded, expected in [(False, 4), (True, 0)]:
    oversold = sum(simulate(steps, guarded) > 1 for steps in schedules)
    assert oversold == expected
    print(f"guarded={guarded}: {oversold}/{len(schedules)} schedules oversold")
```

This enumerates a small abstract model. It supplies a counterexample to “we checked on the previous screen.” It gives no observed oversell probability and establishes no isolation guarantee for a chosen database. A real implementation needs two connections, the intended application role, and the exact posting query. PostgreSQL documents row-update predicate rechecks and the circumstances in which an entire transaction must retry. [PostgreSQL transaction isolation](https://www.postgresql.org/docs/18/transaction-iso.html) · [Stock verification scenarios](inventory-verification.md)

## Race catalog

| Competing actions | Required outcome under the pilot policy | What to coordinate and show |
| --- | --- | --- |
| Online order and counter order reserve the last eligible item | One receives a confirmed reservation. The other sees a shortage. | Guard eligibility and available quantity at commit. A disconnected POS that already handed the item over presents a real physical conflict for a person to resolve. |
| Store pickup and cancellation | One terminal action consumes the outstanding quantity. A late cancellation after collection enters the return path. | Compete on the same fulfillment line or revision. Preserve any physically observed handover and reconcile an uncertain offline outcome before releasing units. |
| Lot hold and dispatch | If the hold wins, block dispatch. If dispatch won first, include that recipient in the trace. | Lock or validate a shared lot eligibility revision and posting boundary. Check staged orders too. A historical shipment can remain valid while its downstream recipient needs a notice. |
| Two stores approve transfers from one warehouse | Combined approved reservations fit the eligible supply. An unaccepted remainder stays visible. | Recheck the source at approval, reserve quantities once, and update the losing suggestion. Include the destination's timely outstanding inbound when recomputing. |
| Count posted while a sale or move arrives | A unit that left during the count affects stock once. | Establish count scope, business time, movement coverage and cutoff. Late uploads may need an explicit recount because record arrival time alone cannot order the physical acts. |
| Old case scanned while its alias is remapped | A ten-bag case still posts ten. Ambiguity goes to review. | Persist the observed presentation or confirmed mapping revision with the command. Prevent a stale draft from silently inheriting a new conversion. |
| Two sessions reroute, pick or accept the same transfer | No two sites receive authority to dispatch the same demand. One receipt cannot close another store's shipment. | Compare expected state/revision and remaining line quantities, retaining merchant and site scope. Show the winning assignment and how much has already moved. |
| Carrier accepts a booking just before a timeout | One confirmed guide and charge, or a clearly unresolved booking. | Keep the same local attempt identity. Reconcile the provider's result before retrying an operation without partner duplicate protection. A local transaction cannot undo an external charge. |

## Which mechanism fits?

- **Single stock row.** A guarded update can require sufficient eligible stock and return the committed result. A zero-row outcome needs a business explanation such as a competing commitment or changed hold.
- **Several rows or a changing set of rows.** Lock shared decision records in a consistent order or use serializable transactions with a bounded full retry. Two workers can satisfy separate row checks while jointly violating a site-wide minimum. Locking only existing rows also leaves questions about new rows matching a predicate. [PostgreSQL explicit locking](https://www.postgresql.org/docs/18/explicit-locking.html) · [Serialization failure handling](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html)
- **Independent systems.** Persist intent, stable identity, acknowledgement and reconciliation evidence. Blocking a button or retrying an HTTP request cannot settle which party has physical custody. [Merchant API contracts](../integrations/merchant-api.md) · [Carrier timeout recovery](../integrations/venezuela-carriers.md)

For a release trial, exercise both orders of each pair, rollback before commit, loss of the reply after commit, and a second device with a different command ID. Compare movements, commitments, outstanding work and the operator-visible result. Preserve an unexplained physical difference with an owner. [Inventory verification](inventory-verification.md) · [Pilot balance replay](../operations/pilot-balance-replay.md)
