# Decisions for the retail pilot

These questions are open for the product owner. Research can proceed under the working assumptions below. A proposed default becomes a product decision when we record who agreed, when, and the supporting evidence.

**Direction already set.** The first customers are small and medium businesses. The product should support a hosted service and allow an enterprise to run its own installation with its own domain, infrastructure and control over data flow. The open choices concern implementation, proof and rollout. The ice, farm and clothing businesses are discovery candidates. Participation and product fit have yet to be established.

For the first channel decision, a concrete question is whether a completed sale identifies the **physical handover site** after synchronization. In the reviewed AROA revision the originating register survives on the local transaction and is omitted from its server transaction payload. A single-site pilot and a multi-store pilot therefore need different evidence before that feed can drive inventory. [AROA register-attribution review](research/integrations/aroa-pos.md#can-a-synchronized-sale-identify-the-store)

| Choice to revisit | Working assumption | Evidence that would change it |
| --- | --- | --- |
| Which merchant and sales channel go first? | Design for one Venezuelan retailer that owns its stock. Support manual sales and one connected checkout. Evaluate Shopify and AROA POS. | A willing pilot merchant shows its actual sales mix, exports, stock adjustments and a recurring problem. Prioritize the channel that reproduces that problem. |
| Who may promise the last unit? | A connected order requests a reservation at one site. A disconnected observation remains visibly pending. | A store must sell through outages while online channels continue accepting orders. Compare per-site stock allocation, safety stock and explicit oversell handling with the merchant. |
| How automatic should replenishment be? | Suggest a transfer when available plus timely, confirmed outstanding inbound falls below minimum. Require an authorized person to approve. | Historical replay shows reliable inputs, few overrides and a manager willing to authorize automatic transfers within agreed limits. |
| What counts as customer delivery? | Track ready, collected by carrier and handed to customer separately. Accept a recorded manual reference for the pilot. | A named carrier offers verified status updates, or the merchant's claims process requires proof of delivery, a signature or other evidence. |
| Which facts belong to the WMS? | The WMS owns physical movements and commitments. The channel owns commercial sales. Billing owns fiscal documents. | A pilot POS already posts authoritative inventory, or a merchant's fiscal workflow requires a different issue point. Reconcile one sale, cancellation and return across systems before choosing ownership. |
| What justifies a native app? | Prove receiving, picking and handoff in a browser on the merchant's hardware. Preserve a path to native device integrations. | Floor trials show an essential capability, device-management requirement or interaction target that the browser cannot meet reliably. |
| Which merchant examples belong in the first trial? | Explore finished ice packs, packed produce and clothing variants against the same receiving, transfer and sale flows. Select one merchant for the initial rollout. | A walkthrough reveals variable weights, sector controls or production steps essential to fulfilling the sale. Decide whether to support those before enrolling that merchant. |

### Product packaging and navigation

| Choice to revisit | Working assumption | Evidence that would change it |
| --- | --- | --- |
| Does a pilot need a visual floor plan? | Keep a stable identity for every pick location and support lookup by its printed code. Trial a simple 2D plan if staff lose time finding locations. | Walk a new picker through an unfamiliar aisle and compare a list, aisle signs and the same task with a map. Include a site with more than one floor or freezer room if relevant. |
| How should we deliver hosted accounts and verify enterprise portability? | Pilot one merchant per tenant, vendor subdomains and pooled tables protected by row-level security. For enterprise, package the same release for the customer's domain and data services. | Rehearse tenant-scoped export and recovery, a domain change with queued work, and an independent installation with vendor connectivity disabled. Actual buyer hardware, recovery objectives and data controls determine the first supported configuration. |

## Evidence for the next discussion

- **Choosing a first business.** Compare the unit actually sold, the handoff at which stock becomes available and the cost of today's correction. [Three merchant scenarios](research/discovery/three-merchant-scenarios.md) · [Spanish conversation guide](research/discovery/merchant-conversations.md)
- **Choosing a first connector.** Trace a single sale from the checkout to the physical handover and the administrative system. Ask whether the branch and central office export the same transaction under different references. A merchant already using a2, Profit Plus or Saint may give us a better first handoff than either of the original candidate channels. [Shopify](research/integrations/shopify.md) · [AROA POS](research/integrations/aroa-pos.md) · [Venezuelan business systems](research/integrations/venezuelan-business-systems.md)
- **Choosing how staff find stock.** Try finding the same bin from both aisle entrances and on each supported device. [Shelf addresses](research/operations/location-addressing.md) · [Map options and fixture](research/experience/warehouse-map.md)
- **Choosing a delivery model.** Compare scoped restore, support access and who controls domain, keys, updates and outbound connections. [Tenant isolation](research/architecture/tenant-isolation.md) · [Enterprise installation](research/architecture/enterprise-installation.md)

The [MVP walkthrough](MVP.md) gives each decision a concrete order or stock movement to test. The [research index](research/README.md) holds the fieldwork and technical sources.
