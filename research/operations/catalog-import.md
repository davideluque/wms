# From a merchant spreadsheet to the first verified scan

Ask the merchant for the original catalog export and one physical example each of a sellable unit and a case. At preview, a clerk should be able to answer “Which item will this row change?” and “What will the scanner count when I receive two of these?” Keep catalog changes and physical receipts as separate actions with separate references.

## Importing for the first time and updating next month

| Moment | Proposed WMS behavior |
| --- | --- |
| Choose the file | Show the merchant, source catalog and import purpose. Preserve the original bytes alongside the selected encoding, delimiter, column meanings and unit rules. Let staff compare the unmodified cell text to the proposed interpretation. |
| Identify a row | Prefer the source's stable variant or package ID, scoped to the merchant and source. Use a separate field for a supplier code or GTIN. Where the source has no durable ID, let staff confirm the match and download a sheet with WMS IDs for the next update. Treat a row number as evidence within one upload. Sorting the next file can change it. |
| Approve the initial catalog | Show new variants, packages, aliases and unresolved rows. A clerk scans or types the label from a physical sample and compares the proposed unit and factor. Record who confirmed that this carton contains six bags. Hold conflicting rows for correction, including two rows with one external ID and incompatible meanings. |
| Apply a subsequent update | Default to updating selected fields on matched records. Leave unlisted records active. For optional fields, show an empty cell as **Conservar valor actual** in this mode. Offer an explicit, previewed clear for fields that permit it. A new ID, package factor change or retired alias needs an explicit decision. Preserve earlier receipt mappings. |
| Recover from interruption | Link the approval to the exact file, saved column mapping, affected records and expected catalog revisions. Persist its outcome so an interrupted client can retrieve it. A corrected or reordered file gets a fresh preview against the current records. A user can see what would actually change before acting again. |

Odoo documents repeat imports using stable external IDs. Shopify's product CSV has different overwrite semantics depending on whether a column is included, blank or dependent on another column. Make the selected behavior visible before a merchant carries their expectations from one tool into another. [Odoo repeated imports](https://www.odoo.com/documentation/19.0/applications/general/base_import.html) · [Shopify CSV overwrite behavior](https://help.shopify.com/en/manual/products/import-export/using-csv)

## A corrected sheet followed by a receipt

This synthetic rehearsal starts after onboarding one merchant and one supplier catalog. Private code `0042` resolves to an ice case of six bags, `42` to a loose bag and `A8` to a shirt. These are private text IDs. The ice cases and loose bags belong to the same stock variant but have different presentations. A supplier sends a smaller file to correct the case display name.

```text
id;nombre;factor
0042;Caja de seis;6
0042;Caja de hielo;6
42;;1
```

1. Preview in update mode with these three columns. Two rows request different names for the same source ID. Show both row references. For this small pilot batch, keep the entire proposed update pending until the clerk resolves the conflict. The shirt's absence from the file requests no action.
2. On the corrected sheet, keep the first name, drop the conflicting row and put the loose bag first. Compare against the loaded catalog. The preview should show **one rename**, **zero new packages**, the existing loose-bag name retained and three catalog entries still active. Applying the corrected rows again in reverse order should show zero further changes.
3. Scan the marked physical case against a receiving document and record two cases. The receipt stores the original two, verified factor six and **12 bags received**. Scanning the sample for the catalog preview and applying the catalog file had posted zero stock movements.

A local Python CSV and in-memory SQLite rehearsal produced the outcomes above. It exercised one merchant, literal source IDs, one parse profile, conflicting rows and a repeated update. The receipt step used a synthetic lookup of the sample's identifier. Real scanner output, merchant files, user approvals and interruption recovery still need an end-to-end test. [Catalog identities and historical factors](catalog-and-units.md) · [Spanish spreadsheet preview](../experience/venezuelan-entry-and-display.md#preview-a-spreadsheet-export-before-it-changes-stock) · [Recovering identifiers from a spreadsheet](../standards/identification-and-exchange.md#recovering-identifiers-from-a-spreadsheet) · [Opening stock and cutoff](opening-inventory.md)
