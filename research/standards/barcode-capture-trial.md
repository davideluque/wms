# What the GS1 parser can recover from a warehouse scan

Dropping a group separator changed an expiry date into part of a valid lot number. The official GS1 parser accepted both inputs. A receiving screen that requires expiry for that item must check that it actually received AI 17, then ask staff to read the label if the field is missing.

## Offline trial, 26 November 2026

Built the official GS1 Barcode Syntax Engine C library in a temporary checkout at commit `81a1ae31fe2529571c8ae2b5a1f5202e81ce977d`. It reported **1.4.1**. The trial explicitly loaded the bundled dictionary dated **2026-01-27** and used its default validation settings. All eight inputs were synthetic strings in scanner-output format, with no image decoding, network lookup or merchant records. The first three characters, such as `]C1`, are the scanner’s AIM symbology identifier. In the table, `<GS>` represents one actual byte, ASCII 29 (hex 1D). [Pinned engine source](https://github.com/gs1/gs1-syntax-engine/tree/81a1ae31fe2529571c8ae2b5a1f5202e81ce977d) · [Pinned dictionary](https://github.com/gs1/gs1-syntax-engine/blob/81a1ae31fe2529571c8ae2b5a1f5202e81ce977d/src/c-lib/gs1-syntax-dictionary.txt) · [GS1 scan-data API](https://gs1.github.io/gs1-syntax-engine/gs1encoders_8h.html)

| Fixture | Scanner output sent to the engine | Observed result |
| --- | --- | --- |
| Retail EAN-13 | `]E09521234500018` | Accepted raw GTIN `9521234500018`; no bracketed AI string was returned. |
| Case, expiry then lot | `]C101195212345000151727063010L7` | `(01)19521234500015(17)270630(10)L7`. The final lot needs no trailing separator. |
| Same case, lot then expiry | `]C1011952123450001510L7<GS>17270630` | `(01)19521234500015(10)L7(17)270630`. The reader supplied the boundary before AI 17. |
| Same bytes with the separator removed | `]C1011952123450001510L717270630` | Accepted as `(01)19521234500015(10)L717270630`. Expiry was absent. |
| Pallet’s SSCC | `]C100095212345000000018` | `(00)095212345000000018`. The leading zero survived. |
| Merchant bin in general Code 128 | `]C0UBI-0047` | Rejected by the GS1 scan API as an unsupported symbology identifier. |
| Same bin typed manually | `UBI-0047` | Rejected by the GS1 scan API because the symbology identifier was missing. |
| Retail GTIN with altered check digit | `]E09521234500019` | Rejected because the primary message check digit was incorrect. |

**Reproduction.** In the pinned checkout, run `make -C src/c-lib libstatic` and link a small C driver against `src/c-lib/build/libgs1encoders.a` using `gs1encoders.h`. For each row, initialize a fresh context with `gs1_encoder_init_ex`, pointing `syntaxDictionary` to the bundled file. Pass the scan string to `gs1_encoder_setScanData`. On success inspect `gs1_encoder_getAIdataStr` and `gs1_encoder_getDataStr`. On failure inspect `gs1_encoder_getErrMsg`. In C the separator fixture can be written as three adjacent literals: `"]C1011952123450001510L7" "\x1d" "17270630"`. A separate literal after the escape keeps subsequent hexadecimal digits out of the escape sequence. [C library and examples](https://github.com/gs1/gs1-syntax-engine/tree/81a1ae31fe2529571c8ae2b5a1f5202e81ce977d/src/c-lib)

## Consequences at the receiving dock

1. **Capture what the reader produced.** Preserve the symbology identifier and separators through scanner, device and input field. The trial began after this step. Test the merchant's actual keyboard-wedge or camera configuration against the same labels, particularly with a variable-length field followed by another AI. [GS1 Barcode Syntax Resource](https://ref.gs1.org/tools/gs1-barcode-syntax-resource/)
2. **Validate the fields the job requires.** Both lot values in the separator experiment satisfy the syntax. For an expiry-controlled SKU, flag missing expiry and preserve the original scan for review. Guessing a date from numeric characters inside a lot would also split legitimate lot numbers. [GS1 AI definitions](https://ref.gs1.org/ai/)
3. **Resolve the correct kind of record.** Look up GTINs against the merchant’s variant and package mapping to decide what one case means. Look up the SSCC against a handling unit or its manifest. For an expected bin scan, resolve the known local identifier `UBI-0047` against the active site’s location list. A check digit verifies part of the syntax. It cannot establish an assigned identity, an available quantity, or which physical label a worker scanned. [Catalog identity and units](../operations/catalog-and-units.md) · [Location addressing](../operations/location-addressing.md)
