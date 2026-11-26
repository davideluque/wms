# Decisions for the retail pilot

These questions are open for the product owner. Research can proceed under the working assumptions below. A proposed default becomes a product decision when we record who agreed, when, and the supporting evidence.

| Choice to revisit | Working assumption | Evidence that would change it |
| --- | --- | --- |
| Which merchant and sales channel go first? | Design for one Venezuelan retailer that owns its stock. Support manual sales and one connected checkout. Evaluate Shopify and AROA POS. | A willing pilot merchant shows its actual sales mix, exports, stock adjustments and a recurring problem. Prioritize the channel that reproduces that problem. |
| Who may promise the last unit? | A connected order requests a reservation at one site. A disconnected observation remains visibly pending. | A store must sell through outages while online channels continue accepting orders. Compare per-site stock allocation, safety stock and explicit oversell handling with the merchant. |
| How automatic should replenishment be? | Suggest a transfer when available falls below minimum. Include confirmed incoming supply and require a person to approve. | Historical replay shows reliable inputs, few overrides and a manager willing to authorize automatic transfers within agreed limits. |
| What counts as customer delivery? | Track ready, collected by carrier and handed to customer separately. Accept a recorded manual reference for the pilot. | A named carrier offers verified status updates, or the merchant's claims process requires proof of delivery, a signature or other evidence. |
| Which facts belong to the WMS? | The WMS owns physical movements and commitments. The channel owns commercial sales. Billing owns fiscal documents. | A pilot POS already posts authoritative inventory, or a merchant's fiscal workflow requires a different issue point. Reconcile one sale, cancellation and return across systems before choosing ownership. |
| What justifies a native app? | Prove receiving, picking and handoff in a browser on the merchant's hardware. Preserve a path to native device integrations. | Floor trials show an essential capability, device-management requirement or interaction target that the browser cannot meet reliably. |

The [MVP walkthrough](MVP.md) gives each decision a concrete order or stock movement to test. The [research index](research/README.md) holds the fieldwork and technical sources.
