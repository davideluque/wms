# Entering quantities and dates in a Venezuelan store

On a receiving screen, let the clerk enter **1,275 kg** and confirm **1,275 kg (1275 g)** before posting. In an exchange file, send the precise quantity and unit under an agreed schema, for example `{"quantity":"1.275","unit":"KGM"}`. Keep the scale reading, source unit and applicable precision with the transaction. The selling system can round a price according to its own rules. That rounding must never change the measured stock.

## Decimal comma at the counter

Unicode CLDR defines the symbols and patterns used to present local numbers. ECMAScript Intl exposes formatted numbers and their parts. A formatter can show the clerk the result after parsing. It does not establish what an unknown scale, spreadsheet or legacy export intended. [Unicode CLDR number formatting and parsing](https://www.unicode.org/reports/tr35/tr35-numbers.html) · [ECMA-402 internationalization API](https://tc39.es/ecma402/2025/)

| Input | Proposed screen or import behavior |
| --- | --- |
| Clerk enters `1,275` in **Peso neto (kg)** | Under the pilot’s comma-decimal entry rule, accept 1.275 kg if the scale and item permit a 1 g increment. Store 1,275 integer grams or an exact decimal quantity. Add a second reading of `0,725` kg and check that the stock ledger totals exactly 2,000 g. |
| Clerk pastes `1.275` | A local grouping mark and a dot-decimal scale feed can imply radically different quantities. For manual entry, disallow grouping and ask “¿Cuánto pesó? Escribe 1,275 para un kilo y 275 gramos.” Show the unit and permit a correction. A documented scale feed configured for dot decimals can parse the same text as 1.275 kg. |
| Register file contains `1,275` | Record the file’s encoding, delimiter, quoting, numeric convention and unit. A comma-delimited file must quote a field containing the decimal comma. Preview the parsed result against the printed receipt before accepting a new file format. A row with an ambiguous convention goes to review under its original line ID. |
| Scale sends `1,275` kg but its declared resolution is 10 g | Preserve the raw value and ask the receiver to verify the instrument or unit. An input precision conflict requires a decision before posting. A countable item such as one sealed ice bag can have its own integer count regardless of its printed nominal weight. [Catalog units and scale precision](../operations/catalog-and-units.md#precision-and-products-sold-by-weight) |

Keep UTF-8 text such as **Depósito Ñandú** on the screen, label and export. A search index may normalize equivalent spellings or offer a forgiving search, with the original name shown for confirmation. Preserve identifier strings such as `00042` and `AÑO-01` exactly as supplied. Define any identifier comparison rules per source and flag collisions. A text search match cannot authorize merging products. [Unicode normalization forms](https://www.unicode.org/reports/tr15/)

## A delayed sale over midnight

Configure each store’s time zone explicitly, for example `America/Caracas`. IANA records a historical offset change for Caracas. Instants should travel with UTC or an explicit offset, while the store retains its zone and operator-defined business day or register shift. A customer’s date-only pickup promise or printed expiry date stays a local calendar date. Ask the merchant what happens when a shift spans midnight and when finance closes a day. a2 describes branch sales synchronizing after connectivity returns, which makes the distinction observable on a real register. [IANA Caracas time-zone change](https://www.iana.org/time-zones/releases/2016d) · [RFC 3339 timestamps](https://www.rfc-editor.org/rfc/rfc3339) · [a2 branch POS synchronization](https://a2.com.ve/producto/13-manager-punto-de-ventas/)

Assume this fictional store reports operating days by Caracas calendar date, opens with **8 bags**, and records a physical handover for each counter sale.

| Source event | Physical time in Caracas | WMS receives | Stock effect |
| --- | --- | --- | --- |
| Sale 501 hands over two bags offline | 26 Nov, 23:58 (`2026-11-27T03:58:00Z`) | 27 Nov, 00:07 | Attribute two bags to 26 Nov and preserve the late arrival. |
| Sale 502 hands over one bag online | 27 Nov, 00:02 (`2026-11-27T04:02:00Z`) | 27 Nov, 00:02 | One bag belongs to 27 Nov even though this event reaches the WMS first. |

At 00:02 the WMS shows seven remaining. After sale 501 arrives, the displayed balance becomes **five**. A reconstructed movement report shows six after the late sale’s physical time, then five after sale 502. Preserve source IDs, original event time, received time, and posting time where it differs. Replaying sale 501 changes zero units. Flag an already closed day for the person authorized to reconcile it and retain the earlier report’s as-of time. If the register calls both sales part of the same trading shift, preserve that identifier as well. Confirm the merchant’s rule before assigning either sale to a fiscal reporting period. [Source register and shift handoff](../integrations/aroa-pos.md) · [Venezuelan warehouse and accounting handoffs](../venezuela/local-discovery.md#dispatch-and-accounting)

## What to reproduce at the site

An in-memory check used Node **18.19.1**, ICU **74.2**, CLDR **44.1** and time-zone data **2023c**. Intl reported the resolved locale as `es-VE`. With three fractional digits it formatted 12345.678 as **12.345,678** and 1.275 kilograms as **1,275 kg**. With `America/Caracas` and a 24-hour clock it displayed the example instants as **26/11/2026, 23:58**, **27/11/2026, 00:02** and **27/11/2026, 00:07**. These are observed results from that runtime. The proposed parser, register semantics and posting workflow still need field tests.

On the oldest supported device, try the actual scale, physical keyboard, phone numeric keypad and two merchant exports. Ask the worker to read the normalized quantity back and correct an ambiguous entry. Round-trip the raw SKU, accented location name, exact weight and both sale times through the WMS and receiving system. Pass when the ledger totals 2,000 g, the two sales leave five bags after replay, and the shift supervisor can show which sale arrived late and which business day was assigned.
