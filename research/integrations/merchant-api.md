# Connecting a merchant's sales channel

A useful first integration takes one order, recovers from a lost reply and answers “where is my order?” without a support call. The following contract is a proposal for the WMS public API. Confirm it by having someone unfamiliar with the WMS connect a sample checkout using the documentation alone.

## First integration walkthrough

The [sample pickup order](first-order-walkthrough.md) shows the actual proposed request and response shapes, including what a checkout can safely tell the customer after losing its first reply.

1. Issue a revocable credential scoped to one merchant and permitted actions. A stock reader needs no order-writing rights. Map the merchant's location and product identifiers before accepting sales.
2. Have the checkout submit its order ID, stable line IDs, requested units and fulfillment method. A store pickup includes its selected location. Resolve the merchant from the credential. Treat an unknown product or location as actionable and preserve the commercial order reference for repair.
3. Return a WMS order identifier and lookup URL. Distinguish “order recorded, awaiting stock” from “three units reserved at Store A.” Request acceptance alone cannot establish availability.
4. Expose an inventory read with unit, location, available quantity, as-of time and revision. Publish a read-only order view with quantities ordered, reserved and handed over by line. A delayed webhook consumer can look up the authoritative state by ID.

Publish the request, response, webhook and error schemas together with working examples. OpenAPI supports HTTP operations and provider-initiated webhooks. RFC 9457 provides an error format with a stable problem type. For a syntactically valid order with an unknown SKU, return the recorded order ID, affected line and external SKU with a pending-mapping status. An unchanged submission replays that recorded outcome. After staff repair the mapping, a new reprocessing command references the existing order and attempts reservation once. The lookup shows its current status. [OpenAPI specification](https://spec.openapis.org/oas/v3.1.1.html) · [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457.html)

## When the reply is lost

Suppose the checkout submits sale 412 with an operation key, the WMS reserves the final three units, then the reply disappears. The client retries with the same key and payload. A proposed contract records the command result atomically with the order write, returns the original result, and rejects a changed payload under that key. Scope each client key by authenticated merchant, source installation and operation type. Check the caller’s current action and site permissions before returning a cached result. Preserve order identity by merchant, installation and external order ID, and line identity within that order, beyond the retry window. The WMS assigns its own merchant-wide movement IDs. A later customer edit is a new operation on the existing order. HTTP defines idempotence at the method level for certain methods. A retryable order-creation POST needs a documented application contract. [RFC 9110 §9.2.2](https://www.rfc-editor.org/rfc/rfc9110.html#name-idempotent-methods)

## Webhooks and catch-up

Proposed envelope fields include event ID, merchant-scoped resource ID, event type, schema version, resource revision and occurrence time. On a stock notification, the recipient needs the affected SKU, location, unit and revision. A customer address belongs only in authorized fulfillment flows that actually require it.

Sign the original body, message ID and attempt timestamp. Verify the signature before parsing or acknowledging. Retry with a stable message ID and a fresh attempt timestamp. Document how signature rotation works. The consumer should accept into durable storage before replying with success, deduplicate there, and process the business operation with a recoverable record. [Standard Webhooks specification](https://github.com/standard-webhooks/standard-webhooks/blob/main/spec/standard-webhooks.md)

If revision 19 arrives before 18, fetch current state before applying a change that assumes an earlier revision. A quiet endpoint can also miss its final notification. Offer paged catch-up from an opaque cursor, with a stable snapshot or upper bound, a published retention period, and a recovery path from an expired cursor. Start a fresh snapshot with its corresponding continuation cursor so changes during enumeration are covered. Show operators the last successful delivery and provide replay using the original event ID.

The [inventory catch-up trial](inventory-catch-up.md) compares three ways to read a changing catalog. It includes an item removed during a download and a new stock revision that arrives while the consumer is catching up.

## Local trial of the order and its webhook

**Observed on 26 November 2026.** A Python 3.12.3 fixture used two independent native SQLite 3.45.1 files. One transaction wrote the order and its outbox event. A second transaction recorded the received event and added its units to the consumer’s demand total. Four short-lived processes exited immediately before or after those commits. All assertions passed without a webhook service or external endpoint.

| Interruption or delivery order | Observed result |
| --- | --- |
| Producer stopped before commit, then stopped after committing A’s 3-unit order 412 | The first interruption left both source tables empty. Retrying after the second retained one order and one event. |
| A’s event 2 arrived before event 1 | The consumer reported a gap and applied zero units. B’s event 1 for its own order 412 still applied seven units. |
| Consumer stopped before committing A’s event 1, then after committing it on retry | The first attempt left no A receipt marker or demand. The next delivery found the committed marker and added zero units. |
| A’s event 2 was replayed, followed by the entire source log | A reached five units and B stayed at seven. Three source orders yielded three outbox rows and three committed consumer markers. |

This fixture uses an immutable, retained log and contiguous sequence numbers **per merchant**. A gap leaves the cursor unchanged until catch-up. A production webhook subscriber receiving only selected event types may see skipped numbers by design, so its cursor must correspond to the stream it actually receives. SQLite serialized fixture writers with `BEGIN IMMEDIATE`. These observations cover local process crashes and replay. They leave PostgreSQL contention, retention expiry, network delivery, power failure and independently applied external effects for separate trials.

<details>
<summary>Reproduce with Python’s standard library</summary>

Save as `trial.py` and run `python3 trial.py`. It creates its own temporary databases, checks the exit status of each interrupted process, and prints `3 orders, 3 events, 3 applied, A=5, B=7` on success.

```python
import multiprocessing as mp
import os
import sqlite3
from pathlib import Path
from tempfile import TemporaryDirectory

def connect(path):
    db = sqlite3.connect(path, isolation_level=None)
    db.execute("PRAGMA synchronous = FULL")
    return db

def crash(selected, point):
    if selected == point:
        os._exit(73)

def post(path, tenant, order_id, units, failure=None):
    db = connect(path)
    with db:
        db.execute("BEGIN IMMEDIATE")
        old = db.execute("SELECT units FROM orders WHERE tenant=? AND id=?",
                         (tenant, order_id)).fetchone()
        if old:
            if old[0] != units:
                raise ValueError("order payload changed")
            return
        db.execute("INSERT INTO orders VALUES (?,?,?)", (tenant, order_id, units))
        seq = db.execute("SELECT COALESCE(MAX(seq),0)+1 FROM events WHERE tenant=?",
                         (tenant,)).fetchone()[0]
        db.execute("INSERT INTO events VALUES (?,?,?,?)", (tenant, seq, order_id, units))
        crash(failure, "before")
    crash(failure, "after")

def deliver(path, event, failure=None):
    tenant, seq, order_id, units = event
    db = connect(path)
    with db:
        db.execute("BEGIN IMMEDIATE")
        cursor = db.execute("SELECT COALESCE(MAX(seq),0) FROM seen WHERE tenant=?",
                            (tenant,)).fetchone()[0]
        if seq <= cursor:
            return "duplicate"
        if seq != cursor + 1:
            return "gap"
        db.execute("INSERT INTO seen VALUES (?,?,?)", (tenant, seq, order_id))
        db.execute("""INSERT INTO demand VALUES (?,?) ON CONFLICT(tenant)
                      DO UPDATE SET units=units+excluded.units""", (tenant, units))
        crash(failure, "before")
    crash(failure, "after")
    return "applied"

def interrupted(fn, *args):
    child = mp.get_context("spawn").Process(target=fn, args=args)
    child.start()
    child.join(10)
    if child.is_alive():
        child.terminate()
        child.join()
        raise TimeoutError("fixture child did not exit")
    assert child.exitcode == 73

def read(path, sql):
    with connect(path) as db:
        return db.execute(sql).fetchall()

def run(root):
    source, sink = root / "source.db", root / "sink.db"
    with connect(source) as db:
        db.executescript("""
          CREATE TABLE orders(tenant TEXT, id TEXT, units INTEGER, PRIMARY KEY(tenant,id));
          CREATE TABLE events(tenant TEXT, seq INTEGER, id TEXT, units INTEGER,
                              PRIMARY KEY(tenant,seq), UNIQUE(tenant,id));""")
    with connect(sink) as db:
        db.executescript("""
          CREATE TABLE seen(tenant TEXT, seq INTEGER, id TEXT, PRIMARY KEY(tenant,seq));
          CREATE TABLE demand(tenant TEXT PRIMARY KEY, units INTEGER);""")
    events = "SELECT tenant,seq,id,units FROM events ORDER BY tenant,seq"
    balances = "SELECT tenant,units FROM demand ORDER BY tenant"
    interrupted(post, source, "A", "412", 3, "before")
    assert read(source, "SELECT * FROM orders") == read(source, events) == []
    interrupted(post, source, "A", "412", 3, "after")
    post(source, "A", "412", 3)  # Retry after the committed reply was lost.
    post(source, "A", "413", 2)
    post(source, "B", "412", 7)
    a1, a2, b1 = read(source, events)
    assert [a1, a2, b1] == [("A",1,"412",3), ("A",2,"413",2), ("B",1,"412",7)]
    assert read(source, "SELECT COUNT(*) FROM orders") == [(3,)]
    assert deliver(sink, a2) == "gap" and read(sink, balances) == []
    assert deliver(sink, b1) == "applied"
    interrupted(deliver, sink, a1, "before")
    assert read(sink, balances) == [("B",7)]
    assert read(sink, "SELECT tenant,seq FROM seen") == [("B",1)]
    interrupted(deliver, sink, a1, "after")
    assert deliver(sink, a1) == "duplicate"
    assert deliver(sink, a2) == "applied"
    for event in read(source, events):
        assert deliver(sink, event) == "duplicate"
    assert read(sink, balances) == [("A",5), ("B",7)]
    assert read(sink, "SELECT COUNT(*) FROM seen") == [(3,)]
    print("3 orders, 3 events, 3 applied, A=5, B=7")

if __name__ == "__main__":
    with TemporaryDirectory() as temp:
        run(Path(temp))
```

</details>

## Contract checks before promising an integration

Run these checks with the identifiers the merchant can actually supply. A Shopify order can include location-specific fulfillment work, while a register may send a completed counter sale only after reconnecting. [Shopify order and fulfillment ownership](shopify.md#ownership-from-checkout-through-handover) · [AROA register transaction mapping](aroa-pos.md#proposed-first-connection)

| Exercise | Expected observation |
| --- | --- |
| Two simultaneous submissions of a mapped order with the same scoped key | One order, one reservation when stock fits, and the same recorded outcome |
| Unknown SKU, identical retry, then mapping repair | One order stays pending through the retry. Reprocessing the same order after repair attempts one reservation and preserves the original submission outcome. |
| Same external order, new retry key | Existing order identified under the documented duplicate policy |
| Cancel after pickup or dispatch | Completed physical movement survives. The caller receives the return or correction path. |
| One merchant presents another merchant's resource ID | No order details, inventory totals or customer data cross merchant boundaries |
| Receiver is offline until delivery retries expire | Snapshot and catch-up converge on the same per-location balances as the WMS |
| Change an optional field, then add an unfamiliar event type | Existing integrations retain their documented behavior. Provide a migration plan before any breaking change. |

Determine actual retry windows, rate limits, cursor retention and supported schema versions from pilot volumes, device disconnection times and the merchant's support needs. Publish the selected values and an example reconciliation before enabling a live channel.
