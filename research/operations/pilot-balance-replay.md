# A reproducible stock reconciliation for the pilot

Use this worked example to check the [MVP walkthrough](../../MVP.md). Each column counts distinct physical units of one SKU. “Warehouse · orders” and “Warehouse · transfers” are quantities reserved but still physically in the warehouse. Once goods leave, their location or custody changes. “Handed over” is cumulative in this example and includes both the counter sale and the carrier collection.

| Step | Warehouse · free | Warehouse · orders | Warehouse · transfers | Store · free | In transit to store | Handed over | Total accounted for |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Opening | 8 | 0 | 0 | 2 | 0 | 0 | 10 |
| Reserve three for online sale | 5 | 3 | 0 | 2 | 0 | 0 | 10 |
| Retry that sale | 5 | 3 | 0 | 2 | 0 | 0 | 10 |
| Carry one out of store | 5 | 3 | 0 | 1 | 0 | 1 | 10 |
| Approve transfer of three | 2 | 3 | 3 | 1 | 0 | 1 | 10 |
| Dispatch transfer | 2 | 3 | 0 | 1 | 3 | 1 | 10 |
| Receive two at store | 2 | 3 | 0 | 3 | 1 | 1 | 10 |
| Book carrier pickup | 2 | 3 | 0 | 3 | 1 | 1 | 10 |
| Carrier takes online order | 2 | 0 | 0 | 3 | 1 | 4 | 10 |

The final “1 in transit” is the book quantity awaiting resolution. Its physical whereabouts are unknown after the short receipt. A supervisor should be able to see both facts. If the clerk later finds it at the store, record a further receipt. If a loss is confirmed, post a loss linked to that transfer. Neither event requires changing the original departure.

## Verify the arithmetic

This small Python 3 model treats a movement as moving a positive integer between two buckets. It includes the sale retry, a lost reply followed by an identical receipt, and one changed-payload command conflict. A pickup booking has no stock posting. Run the code block as a single script. The model exercises the example arithmetic and replay assumptions. It does not simulate database isolation, hardware failure or a real connector.

```python
stock = {
    "warehouse_free": 8, "order_reserved": 0, "transfer_reserved": 0,
    "store_free": 2, "in_transit": 0, "handed_over": 0,
}
applied = {}

def move(command, source, destination, units):
    payload = (source, destination, units)
    if command in applied:
        if payload != applied[command]:
            raise ValueError("Command payload changed")
        return
    assert source != destination and 0 < units <= stock[source]
    stock[source] -= units
    stock[destination] += units
    applied[command] = payload
    assert sum(stock.values()) == 10

move("online-1", "warehouse_free", "order_reserved", 3)
move("online-1", "warehouse_free", "order_reserved", 3)
move("pos-1", "store_free", "handed_over", 1)
move("transfer-approval", "warehouse_free", "transfer_reserved", 3)
move("transfer-departure", "transfer_reserved", "in_transit", 3)
move("receipt-1", "in_transit", "store_free", 2)
move("receipt-1", "in_transit", "store_free", 2)
try:
    move("receipt-1", "in_transit", "store_free", 1)
except ValueError:
    pass
else:
    raise AssertionError("A changed receipt reused an operation ID")

move("carrier-handover", "order_reserved", "handed_over", 3)
assert stock == {
    "warehouse_free": 2, "order_reserved": 0, "transfer_reserved": 0,
    "store_free": 3, "in_transit": 1, "handed_over": 4,
}
assert len(applied) == 6
print("10 units reconciled across 6 distinct stock commands")
```
