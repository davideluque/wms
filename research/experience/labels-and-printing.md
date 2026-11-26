# Labels that work on the shop floor

Start by asking a picker to find two neighbouring bins, distinguish two sizes of the same shirt and retrieve a two-bag pickup. Write down which marks they use, the distance they read them from, what happens when a scanner fails, and who replaces a missing label. Test the merchant’s existing printer and scanners before selecting equipment. The examples below are proposed internal labels for this pilot.

## What needs a label

| Floor object | What the operator needs | When to print |
| --- | --- | --- |
| Bin or shelf position | A stable location ID and a large, readable aisle, rack and level. Keep the label attached to the physical position. Where neighbouring bins share one shelf edge, make the boundary and direction clear. | Print when opening or relabelling the position. An item moving out of the bin leaves the location label in place. |
| Item or sellable package | The exact variant, distinguishing attributes and unit, such as “Azul · M · 1 prenda” or “6 bolsas de 3 kg.” Scan a supplier label against the item master and verify the package factor on the floor. | Reuse a readable, unambiguous supplier code. Print a merchant code for an unmarked item or when the source code cannot distinguish active variants or packs. Preserve required supplier marks and avoid competing targets for the same scan step. [GS1 retail barcode placement](https://ref.gs1.org/guidelines/2d-in-retail/) |
| Pickup bag | One bag ID linked to the order, pickup store, order reference and bag number within the order. Keep the customer’s contact details in the authorized order view. | Print once the bag has an assigned identity. Reprint for that same bag if damaged. If the order is repacked into a different number of bags, retire superseded bag IDs and record the replacement set. |

Two plaintext examples show the words and human-readable values. On the printed version, add a machine-readable symbol encoding the full internal ID printed on the last line. Choose its symbology after testing the merchant’s readers. If a trading partner supplies a label, first identify whether the scanned value names a product, package, lot or logistic unit. [Interpreting GS1 identifiers on partner labels](../standards/identification-and-exchange.md#what-a-scan-identifies)

```text
UBICACIÓN · DEPÓSITO CENTRAL
Pasillo A   Estante 03   Nivel 2
CÓDIGO INTERNO
UBI-DC-A03-N2
```

```text
RETIRO · TIENDA CENTRO
Pedido 1847       Bolsa 1 de 2
CÓDIGO INTERNO DE BOLSA
BOL-TC-00972
```

Allow a worker to type the value shown below a damaged barcode or search by visible location or order reference. Show the matched site, variant and pack, or bag number before confirmation. One order reference may match several bags. In a two-bag collection, show each bag separately and identify the one still missing. The bag ID establishes which bag staff selected. Recipient confirmation supplies the evidence for actual handover.

## Taking printed labels to a new installation

Take one printed sample of each kind before the move and read its payload. A QR symbol can carry a bare internal code, a GS1 Digital Link or an ordinary web address. Each has a different dependency.

| What is printed | What must survive the move |
| --- | --- |
| A bare bin or bag ID, such as `BOL-TC-00972` | The scanner still reads the same characters. Migrate the merchant’s ID mapping, site, object type and status, including retired aliases. A lookup for an already collected bag must show its prior handover. |
| A supplier’s GTIN or SSCC | Preserve the GTIN-to-package mapping and verified pack factor, or the SSCC-to-handling-unit record. A WMS that supports GS1 Digital Link syntax can also extract an encoded GTIN locally from a supplier QR URL. Opening the supplier’s web content depends on its host. [GS1 Digital Link URI syntax](https://ref.gs1.org/standards/digital-link/uri-syntax/1.7.0/) |
| An internal URL such as `https://old.example/bins/UBI-0047` | A phone camera opens the hostname printed on the label. Keep control of that hostname, its DNS and its HTTPS certificate to serve the original path or redirect it to the authorized new location page. A new installation alone cannot redirect the old host. If the old host becomes unavailable, use a printed ID inside the signed-in WMS and replace URL-only labels whose destination cannot be recovered. [HTTPS authority](https://www.rfc-editor.org/rfc/rfc9110.html#section-4.2.2) · [HTTP redirects](https://www.rfc-editor.org/rfc/rfc9110.html#section-15.4) |

**Merchant trial.** Before retiring the old installation, scan a bin label, a labeled case and a previously collected bag on the destination installation. Check the bin’s site, the case’s units and the original handover reference. Open an old QR URL with the normal phone camera and repeat using the signed-in WMS scanner. In a test account for a second merchant that also uses `UBI-0047`, open the first merchant’s link and verify it exposes no location details. A bare code carries no merchant identity, so check the active merchant, expected site and physical sign before confirming work. Try the wrong site and a retired ID. Every lookup or redirect should produce zero stock movements. [Location IDs and aliases](../operations/location-addressing.md) · [Installation and domain changes](../architecture/enterprise-installation.md)

## Printing with equipment already on site

For occasional labels, try an existing office printer with compatible label sheets or a shelf insert. For repeated single-label work, try an installed roll printer through its normal operating-system driver. Record the precise model, driver, device used to print, stock dimensions, connection and available consumables. Measure setup time, parts of sheets discarded, failed prints and replacement frequency. Those observations can justify a dedicated label printer without guessing at local cost or compatibility.

A browser print view can set paper size and margins through CSS and omit screen controls. A PDF can fix the page geometry for a sheet or one roll label. In the print dialog, select matching stock and actual size or 100% scaling. “Fit” can enlarge or shrink symbols, and the printable area may clip the edges. Print a measured reference line on a trial sheet, check it with a ruler and scan the first and last label on a sheet or roll run. Track “print requested” separately from “label applied.” A browser print event does not establish which label emerged or where staff placed it. [MDN print styles and events](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Media_queries/Printing) · [Adobe PDF print sizing](https://helpx.adobe.com/acrobat/desktop/print-documents/set-up-and-print-pdfs/page-size.html)

Use a device-specific command language only when the installed model documents it and the pilot demonstrates a need, such as reliable continuous roll printing. Validate media detection, feed, label width, darkness, speed and scan quality on that device. Keep the working PDF route available if a dedicated print station is offline. Zebra, for example, documents a print-quality self-test for its ZD411T and reports language and setup issues in its GK888t guide. These are model-specific procedures. [Zebra ZD411T print test](https://docs.zebra.com/us/en/printers/desktop/zd411t-ug/c-mlk-ug-onboard-printer-tools/t-mlk-ug-print-quality-report.html) · [Zebra GK888t troubleshooting](https://docs.zebra.com/us/en/printers/desktop/gk888t-desktop-thermal-printer-user-guide/c-gk888t-ug-troubleshooting/r-gk888t-ug-print-quality-problems.html)

## Reprints, swaps and field acceptance

For a reprint, show the object type, exact variant or bag, site, intended printer and number of copies. Keep the original ID and log the operator, time, reason and print job. If the output is uncertain, check the tray or queue before sending another copy. Have the operator remove the obsolete label and check the replacement against an independent cue, such as the shirt’s sewn-in size tag, the fixed aisle marker or the order’s packing list. A scan of a good label stuck on the wrong object will decode perfectly. For a suspected swap, stop the affected pick or handover, identify both physical objects, correct the labels, and review work already recorded against them.

1. Print the smallest item label, a shelf label and a two-bag pickup using the merchant’s routine equipment. Scan at working distance and under the site’s lighting. Try a curved bag, a rubbed label and any actual condensation exposure. Check attachment, contrast and empty space around the symbol after application. Pass when the correct ID resolves and a coworker can recover by reading the printed value. A field scan checks usability. GS1 print-quality verification for a partner label is a separate measurement. [GS1 quality checks](https://support.gs1.org/support/solutions/articles/43000734141-what-should-i-check-to-ensure-good-quality-barcodes-) · [GS1 verification guidance](https://www.gs1.org/docs/barcodes/2D_Barcode_Verification_Process_Implementation_Guideline.pdf)
2. On a shelf with adjacent sizes, place an M label on an L sample. Present a bin scan when the screen expects an item. Ask the operator to detect both problems and correct them. Check that the wrong scan, reprint and relabel each produce zero stock movement.
3. Interrupt printing midway through a batch. Use the printed IDs to identify the last applied label and resume only missing work. Reprint pickup bag 1, then scan both copies. The order should still show bag 2 outstanding. Complete pickup only after both actual bags and the recipient have been confirmed.
