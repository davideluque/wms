# Opening inventory for a shop and warehouse

The opening balance is a signed-off physical count at a known time, with existing customer commitments, holds and transfers attached. A catalog import alone cannot tell a picker what they can promise. Start with one merchant, one warehouse, the participating stores and countable goods. The steps below are proposed pilot controls. Shopify’s protected inventory import and Business Central’s count journals provide examples of checking expected quantities and posting differences. [Shopify inventory CSV](https://help.shopify.com/en/manual/inventory-and-locations/setup/inventory-csv) · [Microsoft physical and cycle counts](https://learn.microsoft.com/en-us/dynamics365/business-central/inventory-how-count-adjust-reclassify)

## Map what the cashier sells to what staff count

Collect a dated catalog and stock export from each source, recent register receipts, one supplier pack label, the names staff use for locations, open orders and transfer records. Walk a sale and a receipt with staff. Ask them to point to the selling unit, the case it arrives in, the shelf holding returns and any goods picked for customers. Record which system already changes stock for each action.

| Example merchant row | Proposed import result | Question to settle on the floor |
| --- | --- | --- |
| Ice, `HIELO-3`, one sealed 3 kg bag. A supplier bundle contains six bags. | Use the sealed bag as the base unit. Three unopened bundles and four loose bags open as 22 bags. Keep the observed quantities and the verified factor of six. | Does every bundle contain six bags? Is any ice made, sold loose or lost to melting before it can be sold? Record damaged bags or a measured loss separately. |
| Apparel, register SKU `CAM-01`, two source variant IDs for blue M and blue L. | Create two variant mappings. Mark the shared SKU or barcode ambiguous wherever the source sends only that code. Count the two sizes independently. | Can the cashier or scanner identify the size at sale? Test an exchange of M for L. GS1 assigns distinct GTINs to distinct size and colour combinations when used. [GS1 variant guidance](https://support.gs1.org/support/solutions/articles/43000734083-how-many-gs1-gtins-do-i-need-when-i-have-a-product-with-many-sizes-and-colours-) |
| Produce, `TOM-SUELTO`, one crate with a measured 13.6 kg net. Another weighs 15.1 kg. | Flag loose tomatoes outside the countable-goods pilot. Two crates do not establish a constant kg conversion. A separately sold, fixed pack can enter the pilot after its mapping is verified. | Does checkout report actual net weight, a price-embedded label, or just a piece count? A weight-based extension needs tare, scale precision, fractional sale quantities and spoilage reconciliation. [GS1 variable-measure definition](https://support.gs1.org/support/solutions/articles/43000734396-what-is-a-variable-measure-trade-item-) |

An illustrative **WMS staging file** records observations in the unit counted. Two rows may describe the same SKU if they refer to disjoint physical groups. The two ice rows below refer to intact bundles and loose bags, respectively. Staff must exclude the bags inside those bundles from the loose count.

```csv
batch,line,location,bin,item,count_unit,counted,base_per_count_unit,condition
OPEN-01,001,WH,FREEZER-1,HIELO-3,bundle,3,6,saleable
OPEN-01,002,WH,FREEZER-1,HIELO-3,bag,4,1,saleable
OPEN-01,003,STORE-A,RACK-1,CAM-AZ-M,each,3,1,saleable
OPEN-01,004,STORE-A,RETURNS,CAM-AZ-M,each,1,1,hold
OPEN-01,005,STORE-A,PICKUP,CAM-AZ-M,each,2,1,saleable
```

Treat source variant IDs, barcodes and row IDs as text. Scope aliases by merchant and source. On preview, show the original row, resolved variant, location, factor and base quantity. Flag ambiguous identifiers, conflicting repeated line IDs, unknown locations, missing units and fractional counts for discrete items. An explicit zero means staff counted none. A blank means the count is incomplete. Give an exact retry of an accepted batch the same result. A corrected file should reference the prior batch and show its differences before posting. Preserve the original exports and approval.

## Take the count while keeping sales explainable

1. **Set a cutoff.** An after-close count is simplest when all registers can sync. Record the last sale, receipt, dispatch and transfer at each site. Assign one owner per zone, including back room, pickup shelf and returns. Count physical units without displaying the book balance. Record time, unit and counter. A second person recounts unexplained differences, especially a disputed pack size or a saleable item found on hold.
2. **If the store must trade, count one zone at a time.** Briefly control access to the zone or log every movement with time, quantity, reference and whether it occurred before or after that item’s count. Move the observations to a common cutoff. For example, four bags observed at 16:00, one carried out at 16:05 and two received at 16:10 give five at 16:15. Replay each movement once. A register that may hold unsynced sales leaves that zone provisional. If the order of a movement and count is uncertain, isolate and recount the affected item.
3. **Reconcile by variant, location and condition.** Compare book on-hand at cutoff with counted on-hand before allocating open orders. Investigate unit errors, stale locations, misplaced stock and unposted movements. Distinguish an approved count adjustment from a transfer correction. Keep evidence for both sides of a transfer. A unit physically dispatched before cutoff stays in transit until received.
4. **Check commitments against the count.** In the example, Store A has six blue M shirts physically present. One is on hold. Two physically sound shirts on the pickup shelf match an open order for two. That leaves three available, two committed and one held. Tie each commitment to an open order line and keep sales disabled for unresolved negative availability. Check totals per site, item and status before comparing the merchant-wide sum.

## Switch over, recover and keep counts current

Run the first import in preview. Ask the merchant to sign the count time, unresolved rows and locations eligible for promises. Activate one owner for outbound stock updates at the agreed cutoff. Record a checkpoint per source and whether each sale’s physical handover was already counted at cutoff. For a delayed pre-cutoff sale already reflected in opening stock, link the sale history with zero additional stock deduction. Post each post-cutoff handover once under its source ID. If inclusion is uncertain, hold the affected item out of promising until staff reconcile or recount it. Compare each activated location’s WMS and channel quantities after the first live sale. Shopify’s inventory CSV can compare exported `On hand (current)` with the level at import and skip rows changed meanwhile. Investigate skipped rows individually. Equivalent protection belongs on the WMS import. [Shopify inventory import validation](https://help.shopify.com/en/manual/inventory-and-locations/setup/inventory-csv)

If a batch fails before activation, withdraw its staged rows and preview the corrected file. If staff discover an error after sales restart, pause outbound stock updates for the affected items, preserve subsequent sales and receipts, post a linked correction and reconcile before resuming. If the merchant must revert to the old system, name one writer, export movements since cutoff, replay them there once, and compare current balances before it resumes promising stock. Restoring yesterday’s snapshot would erase today’s transactions.

After launch, schedule small rolling counts for fast sellers, discrepancy-prone items and locations that recently received transfers. Let observed variance determine the frequency. Use the same count, recount and approval trail. Track overdue counts, unexplained units by location and repeat discrepancies by cause. [How to reconcile a count while the shop is trading](cycle-counting.md) · [Microsoft cycle counting](https://learn.microsoft.com/en-us/dynamics365/business-central/inventory-how-count-adjust-reclassify)

**Operator acceptance.** A second-shift employee should be able to resume an interrupted count, distinguish zero from uncounted, explain why the example offers three shirts, locate the pending pickup and held shirt, and identify who resolved an ambiguous SKU. Replay the opening batch, sell one unit during a rolling count, and correct one pack factor after a sale. Pass when the opening balance posts once and every current balance can be reconstructed from the count and subsequent movements.

## Pilot trial — two offline registers cross the opening count

A clerk counts **10 bags** at one store. Two registers upload completed carry-outs later. Both use receipt numbers 0700 and 0701. In this fixture, the register plus its numbering epoch identifies a source namespace within the merchant and POS installation. A real connector should use the source’s stable transaction and line IDs where available and preserve the selling site at the time of handover.

| Register and receipt | Bags | Evidence about the physical count | Posting against the 10 opening bags |
| --- | ---: | --- | --- |
| East 0700 | 2 | Customer already carried them out when staff counted. | Record the sale. Its stock effect is already reflected in the count. |
| West 0700 | 3 | Staff confirmed handover after the count. | Deduct 3. |
| East 0701 | 1 | Staff confirmed handover after the count. | Deduct 1. |
| West 0701 | 2 | The register and the count sheet cannot establish which happened first. | Preserve the sale, open a review and suspend promises for this item/site. Show 6 as provisional. |

**Observed on 26 November 2026.** Python 3.12.3 with native SQLite 3.45.1 replayed the four records in both table order and reverse order, including full-batch retries. Deduplicating on the bare receipt number and subtracting the first occurrence produced **7** bags in table order and **5** in reverse order. Scoping the IDs correctly but subtracting every late upload produced **2** in both orders. Recording cutoff evidence per handover retained all four sales, posted only the two established post-count handovers and produced **6 provisional bags and one review** in either order.

The trial exercised both possible review outcomes on fresh fixtures. If the two-bag handover had already affected the physical count, the balance closed at **6**. If it happened afterward, the balance closed at **4**. Replaying the review and all sales repeated no deduction. These two answers use different supplied evidence. Neither register timestamps nor upload order establish which answer describes a real store. For the pilot, ask for the zone count, movement log and register-close evidence, or recount with intervening movements accounted for. Record the decision, reviewer and evidence reference before reopening promises.

<details>
<summary>Reproduce the four synthetic runs using Python’s standard library</summary>

Save as `trial.py` and run `python3 trial.py`. The in-memory database represents one store and SKU with one stock-bearing line per sale. The unresolved count is the signal to withhold promises. Register software, application permissions and a live promise endpoint are outside this fixture.

```python
import sqlite3

# One merchant, one source installation, one site and one countable SKU.
# Register keys include their numbering epoch. Each receipt has one line.
rows = [
    ("E-1", "0700", 2, "included"),
    ("W-1", "0700", 3, "after"),
    ("E-1", "0701", 1, "after"),
    ("W-1", "0701", 2, "unknown"),
]

def trial(batch, ruling):
    db = sqlite3.connect(":memory:")
    db.executescript("""
      CREATE TABLE sale(reg TEXT, ref TEXT, qty INTEGER, evidence TEXT,
                        PRIMARY KEY(reg,ref));
      CREATE TABLE review(reg TEXT, ref TEXT, ruling TEXT, evidence_ref TEXT,
                          PRIMARY KEY(reg,ref));
      CREATE TABLE posting(reg TEXT, ref TEXT, qty INTEGER, PRIMARY KEY(reg,ref));
      CREATE TABLE bare(ref TEXT PRIMARY KEY, qty INTEGER);
    """)
    def import_row(row):
        reg, ref, qty, evidence = row
        with db:
            db.execute("INSERT OR IGNORE INTO bare VALUES (?,?)", (ref,qty))
            old = db.execute("SELECT qty,evidence FROM sale WHERE reg=? AND ref=?",
                             (reg,ref)).fetchone()
            if old is not None:
                if old != (qty,evidence):
                    raise ValueError("changed source record")
                return
            db.execute("INSERT INTO sale VALUES (?,?,?,?)", row)
            if evidence == "after":
                db.execute("INSERT INTO posting VALUES (?,?,?)", (reg,ref,qty))

    def state():
        balance = db.execute("SELECT 10-COALESCE(sum(qty),0) FROM posting").fetchone()[0]
        unresolved = db.execute("""SELECT count(*) FROM sale LEFT JOIN review USING(reg,ref)
                                   WHERE sale.evidence='unknown' AND review.ruling IS NULL""").fetchone()[0]
        return balance, unresolved

    for row in batch + batch:
        import_row(row)
    assert db.execute("SELECT count(*) FROM sale").fetchone()[0] == 4
    assert state() == (6,1)
    bare = db.execute("SELECT 10-sum(qty) FROM bare").fetchone()[0]
    all_late = db.execute("SELECT 10-sum(qty) FROM sale").fetchone()[0]
    reg, ref, qty, _ = rows[3]
    def resolve():
        with db:
            old = db.execute("SELECT ruling FROM review WHERE reg=? AND ref=?", (reg,ref)).fetchone()
            if old:
                assert old == (ruling,)
                return
            db.execute("INSERT INTO review VALUES (?,?,?,?)", (reg,ref,ruling,"CHECK-21"))
            if ruling == "after":
                db.execute("INSERT INTO posting VALUES (?,?,?)", (reg,ref,qty))
    resolve()
    resolve()
    for row in batch:
        import_row(row)
    final, unresolved = state()
    assert final == (6 if ruling == "included" else 4) and unresolved == 0
    db.close()
    return bare, all_late, final

for batch, expected_bare in [(rows,7), (rows[::-1],5)]:
    for ruling in ("included","after"):
        bare, all_late, final = trial(batch,ruling)
        assert (bare,all_late) == (expected_bare,2)
        print(f"bare={bare} all_late={all_late} provisional=6 review={ruling} final={final}")
```

</details>
