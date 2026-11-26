# Connecting a merchant's sales channel

A useful first integration takes one order, recovers from a lost reply and answers “where is my order?” without a support call. The following contract is a proposal for the WMS public API. Confirm it by having someone unfamiliar with the WMS connect a sample checkout using the documentation alone.

## First integration walkthrough

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

## Contract checks before promising an integration

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
