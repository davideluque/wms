# WMS for Venezuelan merchants

A small or medium business should be able to trace a sale from the channel that took the order, through the location that promised the stock, to the person or carrier who received it. The project is in product discovery and technical research. The proposed pilot connects a warehouse with a small number of stores.

| Start here | What you'll find |
| --- | --- |
| [Product principles](PRINCIPLES.md) | How the software should behave when people make mistakes, work is interrupted or a manual decision is pending. |
| [Proposed MVP](MVP.md) | The first sale, pickup, dispatch and replenishment workflows, with a ten-unit acceptance walkthrough. |
| [Decisions for the product owner](DECISIONS.md) | Open choices, current assumptions and the evidence that could change them. |
| [Research library](research/README.md) | Investigations grouped by merchant discovery, warehouse operations, user experience, integrations and architecture. |

## Follow a question

- **Could this work for my business?** Compare a bag of ice, a weighed crate and a clothing variant. The Spanish interview guide starts with an actual sale. [Merchant examples](research/discovery/three-merchant-scenarios.md) · [Guía de conversación](research/discovery/merchant-conversations.md)
- **How would a picker find a shelf?** Follow the stable bin ID from the printed rack label to the same place on a two-dimensional floor plan. [Location addressing](research/operations/location-addressing.md) · [Map design](research/experience/warehouse-map.md)
- **What happens when two people take the last unit?** Run a small model that enumerates the possible orderings. Then examine the database and operator checks a real implementation would need. [Race conditions](research/architecture/race-conditions.md)
- **How would customers access it?** Compare a merchant subdomain on the shared service with a dedicated installation under the customer's domain and operational control. [Tenant isolation](research/architecture/tenant-isolation.md) · [Enterprise installation](research/architecture/enterprise-installation.md)
