# A checkout’s first pickup order

Suppose your checkout sells three blue medium shirts for pickup at Store A. This proposed HTTP exchange lets you answer three questions independently. Did the WMS record the sale? Did Store A reserve all three shirts? Did the customer receive them? The paths, fields and responses below are illustrative. This exchange has not been run against a service. [Proposed merchant API](merchant-api.md) · [Pickup routing policy](../operations/order-routing.md)

## 1. Save the identifiers before submitting

Quantities on this wire are exact decimal strings. An order for three shirts sends `"3"`, and the item accepts only whole units. The revision is a separate integer. The [HTTP framework trial](../architecture/server-tooling.md#observed-http-contract-on-two-frameworks) demonstrates why both ends need to validate the representation they actually send.

Save the source installation, commercial order ID, source line ID, exact request body and operation key in the checkout before the first POST. Send a merchant-scoped credential on every WMS request. The WMS validates that the credential may submit for this source and pickup site. The token below is a placeholder.

```http
POST /v1/orders HTTP/1.1
Authorization: Bearer <merchant-token>
Content-Type: application/json
Idempotency-Key: sale-412-create

{
  "source_installation": "checkout-centro",
  "external_order_id": "412",
  "fulfillment": {"method": "pickup", "location_id": "store-a"},
  "lines": [
    {"external_line_id": "line-7", "external_sku": "CAM-AZ-M", "unit": "each", "quantity": "3"}
  ]
}
```

Suppose the WMS records the order and the connection drops before the checkout receives its reply. **Send the identical POST with the same key.** In this example the source SKU has no catalog mapping. The retry returns the original creation result.

```http
HTTP/1.1 201 Created
Location: /v1/orders/ord_83
Content-Type: application/json

{
  "id": "ord_83", "external_order_id": "412", "revision": 1,
  "status": "pending_mapping",
  "issues": [{"code": "unknown_sku", "external_line_id": "line-7", "external_sku": "CAM-AZ-M"}],
  "lines": [{"external_line_id": "line-7", "unit": "each", "ordered": "3", "reserved": "0", "handed_over": "0"}]
}
```

Persist `ord_83` and its lookup URL alongside sale 412. The status and zero reservation tell the checkout that it can acknowledge capture and still owes the customer a pickup promise. Changing the quantity under `sale-412-create` would produce an idempotency conflict. An accidental new key for the same source and sale must still resolve the existing commercial order under the duplicate policy. [Order identity and replay](merchant-api.md#when-the-reply-is-lost)

## 2. Repair the mapping and reprocess the same order

The merchant verifies that `CAM-AZ-M` from `checkout-centro` means one blue medium shirt per unit. An authorized integration or staff action then submits a **new command against `ord_83`**. Here Store A has five eligible shirts and no other commitments. The selected pickup store can reserve all three.

```http
POST /v1/orders/ord_83/reprocess HTTP/1.1
Authorization: Bearer <merchant-token>
Content-Type: application/json
Idempotency-Key: sale-412-reprocess-1

{}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "ord_83", "revision": 2, "status": "reserved", "location_id": "store-a",
  "lines": [{"external_line_id": "line-7", "unit": "each", "ordered": "3", "reserved": "3", "handed_over": "0"}]
}
```

If this reply is lost, retry the reprocess request with its own original key. Store A still has five physically present, three reserved and two available. An identical retry of the **creation** command can still return its earlier revision-1 outcome. Use the lookup URL to read the current order. If Store A has fewer than three eligible shirts, the order needs staff attention and another site’s stock cannot make this pickup ready.

## 3. Show pickup status from the current order

When staff verify that all three shirts are in the pickup bag, the WMS can mark revision 3 `ready_for_pickup`. The three units stay protected. A signed webhook may tell the checkout that an order changed. An illustrative body is shown below. Verify the signature over the original body and signing metadata before trusting the hint, then durably record the event ID before acknowledging. [Proposed webhook contract](merchant-api.md#webhooks-and-catch-up)

```json
{
  "event_id": "evt_83_3", "type": "order.updated", "schema_version": 1,
  "resource_id": "ord_83", "revision": 3, "occurred_at": "2026-11-26T15:00:00Z"
}
```

On a hint, fetch the known order. Also poll open orders by their saved URLs when notifications are delayed or silent. If the customer has already collected by the time an older notification arrives, the lookup can return revision 4.

```http
GET /v1/orders/ord_83 HTTP/1.1
Authorization: Bearer <merchant-token>
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "ord_83", "source_installation": "checkout-centro", "external_order_id": "412",
  "revision": 4, "status": "collected", "location_id": "store-a",
  "lines": [{"external_line_id": "line-7", "unit": "each", "ordered": "3", "reserved": "0", "handed_over": "3"}],
  "handover_ref": "pickup_29"
}
```

Display the newest known revision. Replaying `evt_83_3` should never revert “collected” to “ready.” In the example Store A finishes with two on hand and two available. For a pilot acceptance check, interrupt both POST responses, replay the ready notification after collection, and have staff reconcile sale 412 to **one WMS order, one three-unit reservation and one three-unit handover**. The signed message can be delivered repeatedly. The order and movement records supply the evidence that the sale affected stock once.
