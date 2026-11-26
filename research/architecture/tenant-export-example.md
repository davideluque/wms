# Taking an unfinished transfer to a new installation

A merchant leaving the shared WMS needs enough history to answer why a store received four bags when the warehouse sent six. Importing a spreadsheet of current balances would hide the original six-bag case, the worker’s count and the two bags still due. This synthetic package follows one SKU across that handoff. All names, identifiers, evidence and records are invented.

## Scope and verification

Merchants A and B both use SKU `0042`, supplier receipt `R-218`, transfer `T-17`, movement `M3` and attachment name `slip-1.txt`. A’s slip says four bags. B’s says nine. Select rows under the authorized merchant and checkpoint, then follow references using the same merchant on every hop. The destination checks the expected merchant supplied by its administrator against the manifest, rows and object references. A shared external ID or filename alone cannot establish ownership. [Tenant isolation outside the stock ledger](tenant-isolation.md#carry-the-boundary-outside-the-stock-ledger)

The example manifest lists every payload file. Compare the entry set, length and SHA-256 of the original bytes, reject duplicate or unsafe paths and check that the slip belongs to the receiving movement. In a delivered product, authenticate the manifest against an independently trusted signer or digest before trusting those hashes. A party that can replace the manifest and payload together can compute matching hashes. RFC 8493 describes payload-manifest validation for archive exchange. This fixture uses a small custom JSON format. [BagIt, RFC 8493](https://www.rfc-editor.org/rfc/rfc8493)

**manifest.json**

```json
{
  "format": "wms-exit-example/1",
  "export_id": "A-exit-3",
  "merchant": "A",
  "checkpoint": 3,
  "as_of": "2026-11-26T10:05:00Z",
  "connection_mode_on_import": "disabled",
  "files": [
    {"path":"catalog.json","bytes":83,"sha256":"7cb0018be07ba364ddf52ac9ce87b99334b46d15eecca3b938aa3cb23058592f"},
    {"path":"moves.jsonl","bytes":227,"sha256":"46212dbd8a57778b5b6ae2b4c72e847b54d5f7f64dd23b6740bf04422e5ccaf6"},
    {"path":"work.json","bytes":213,"sha256":"d056dad25bee9484b05a8dd31ca95a20284748a4c29aada311cf3cceba5692b6"},
    {"path":"outbox.json","bytes":137,"sha256":"8a7cd2429bd99ee0ffccaaacf1100dfd90cb0e2799b212ef91e8b26361efe400"},
    {"path":"evidence/A/slip-1.txt","bytes":49,"sha256":"0f528303652500198aedf7c862b0d47cc7fe245563c5996afee966b1bcadc0a0"}
  ]
}
```

Each payload below is UTF-8, with a single line feed after every displayed line, including the final line. The move rows list merchant, sequence, movement ID, operation ID, UTC time on the manifest date, actor, document, source, destination, observed quantity, observed unit and base bags, in that order. All three movements refer to the catalog’s sole SKU. `TRANSIT` denotes custody between sites. `supplier` is outside the merchant’s ledger.

**catalog.json**

```text
{"merchant":"A","sku":"0042","base_unit":"bag","pack":"case-v1","bags_per_case":6}
```

**moves.jsonl**

```text
["A",1,"M1","recv-218","08:00","ana","R-218","supplier","WH",2,"case-v1",12]
["A",2,"M2","ship-17","09:00","leo","T-17","WH","TRANSIT",1,"case-v1",6]
["A",3,"M3","accept-17-1","10:00","ines","T-17","TRANSIT","STORE",4,"bag",4]
```

**work.json**

```text
{"merchant":"A","transfer":"T-17","from":"WH","to":"STORE","sku":"0042","unit":"bag","sent":6,"received":4,"pending":2,"state":"partial","owner":"ines","evidence":{"path":"evidence/A/slip-1.txt","movement":"M3"}}
```

**outbox.json**

```text
{"merchant":"A","id":"D1","event":"M3","connector":"retail-channel","delivery":"ack_unknown","attempts":1,"external_ref":"ACK-QUERY-17"}
```

**evidence/A/slip-1.txt**

```text
Merchant A | T-17 | M3 | 4 bags counted by ines.
```

The only integration fields carried over are a logical connector name, event reference and prior attempt state. The export excludes tokens, passwords, signing keys, sessions, endpoint credentials and signed attachment URLs. The destination must override imported execution settings with local controls. Keep publishers stopped, the outgoing queue held and external delivery disabled. Receiving an outbox entry reconstructs history. It does not authorize contact with a carrier or checkout.

## Reconstruct before permitting writes

1. Stage the full package under A. Validate format, hashes, row scope, unique sequence through checkpoint 3, references and unit conversion. Preserve original operation IDs and results so a retry of `accept-17-1` returns `M3`. Use one atomic import or an equivalent fenced staging switch. A failed validation must leave the active record set untouched.
2. Replay the physical movements in order. Receipt `M1` adds **2 cases × 6 = 12 bags** at WH. Dispatch `M2` puts **6 WH + 6 TRANSIT** on record. Receipt `M3` leaves **6 WH + 2 TRANSIT + 4 STORE = 12 bags**. Match the pending transfer amount to both `6 − 4 = 2` and transit stock. Four are physically at the store. The two pending bags must remain unavailable there.
3. Reimport the same package under the same export ID and manifest digest. Return the prior result without posting again. Reject a reused export ID with different content. Also enforce stable movement and operation identities within the merchant so a newly named package cannot duplicate an existing business event. If the destination already holds conflicting history, stop for reconciliation.
4. After sign-off, staff can record a genuinely new receipt of the remaining two bags against `T-17`, with its own operation ID. That closes the transfer with **6 WH + 0 TRANSIT + 6 STORE = 12**. Keep the earlier four-bag observation and slip. Before activating a connector, reauthorize it, check what the source already delivered and reconcile the uncertain `D1` by its original reference. A missing acknowledgment needs an explicit operator decision if the partner cannot resolve it. [Customer-operated cutover and recovery](enterprise-installation.md#upgrades-recovery-and-tenant-moves)

## Replay evidence and limits

An in-memory Python rehearsal read the five payloads directly from this note. Their lengths totaled **709 bytes**, and all five SHA-256 digests matched. Restoring twice and retrying the original receipt left A at **6 / 2 / 4** and B’s store at **9**. Importing the same movements under a different export ID also preserved the balances. Four negative cases left the active state unchanged. They covered a one-byte slip edit, an extra B attachment with a valid digest but no authorized reference, a B movement under recomputed checksums, and reuse of the export ID with different validated content. A new two-bag receipt closed the transfer at **6 / 0 / 6**. The simulated outbox kept D1 held with its previous outcome unknown. The replay used local data structures without a network sender.

The merchant should receive a completion report stating the checkpoint, file and row counts, quantities per site and disposition, unresolved work and integration state. For a rehearsal, run again with B’s colliding identifiers, alter one byte of A’s slip, retry the accepted four-bag receipt and reimport the same bundle. A production exit also requires the complete schema, all affected SKUs and historical pack revisions, grants and identity remapping, outstanding reservations and holds, declared export coverage, attachment authorization checks, fence verification and proof of zero external traffic while publishers are held. [Merchant account exit](../experience/merchant-account.md#who-can-act-when-the-account-changes)

## A late scanner write during the move

A local trial on 26 November 2026 used Python 3.12.3 and two native SQLite 3.45.1 databases. A’s initial export ended at revision 3 with two bags in transit and four at the store. The destination started closed to writes. On the source, every receipt checked A’s write permission in the same transaction that consumed the remaining transfer quantity. Closing the source to A’s writes and reading its final revision also formed one transaction.

| Controlled order of events | Source checkpoint | Observed destination result |
| --- | ---: | --- |
| The old scanner posted one bag before the source fence, and its reply was lost. | 4. The first fence attempt encountered the open writer. After the writer committed, the fence captured revision 4. | Importing through 4 included the scanner’s original outcome. Its retry returned 4 and moved zero additional bags. |
| The fence closed first. The scanner sent its one-bag observation to the old address afterward. | 3. The source declined the write. | After activation the same command posted once at the destination as revision 4. A second retry returned 4. |

In both cases the original four-bag receipt still returned revision 3. A retry changing the one-bag payload to two returned a conflict. Two different clerks then claimed the last pending bag. We ran both arrival orders: the first posted revision 5 and the other went to review. The destination ended with **0 in transit and 6 at the store**, and the old installation remained fenced for A. While A was fenced and its destination still closed, B used the same scanner command ID at the source and increased its own store from nine bags to ten. B was absent from the destination.

**Pilot implication.** Keep the original observation and command ID on the scanner until the authoritative destination settles it. Copy the source’s command results through the final fenced checkpoint before opening the new writer. Route a conflicting second physical observation to a person. An installation returning “fenced” establishes no stock outcome by itself.

<details>
<summary>Repeat the four schedules with Python’s standard library</summary>

Save as `trial.py` and run `python3 trial.py`. The source and destination are temporary SQLite files. The copy runs at controlled quiescent points and replaces the small fixture state for A. The test exercises one source write lock and both serial orders of the destination contenders. SQLite permits only one simultaneous writer per database, so B’s observed progress was **after the fence committed**, while A’s migration was still underway. Directory routing, authentication, multi-node failures, loss of physical evidence and delivery to external systems remain separate trials.

```python
import sqlite3
from pathlib import Path
from tempfile import TemporaryDirectory

def connect(path):
    return sqlite3.connect(path, isolation_level=None, timeout=0)

def setup(db):
    db.executescript("""
      PRAGMA journal_mode=WAL;
      CREATE TABLE site(m TEXT PRIMARY KEY, active INTEGER, rev INTEGER, transit INTEGER, store INTEGER);
      CREATE TABLE done(m TEXT, id TEXT, qty INTEGER, result INTEGER, PRIMARY KEY(m,id));
    """)

def receive(db, m, key, qty):
    active, rev, pending, store = db.execute(
        "SELECT active,rev,transit,store FROM site WHERE m=?", (m,)).fetchone()
    if not active:
        return "fenced"
    old = db.execute("SELECT qty,result FROM done WHERE m=? AND id=?", (m,key)).fetchone()
    if old:
        return old[1] if old[0] == qty else "conflict"
    if qty <= 0 or qty > pending:
        return "review"
    db.execute("UPDATE site SET transit=?,store=?,rev=? WHERE m=?",
               (pending-qty,store+qty,rev+1,m))
    db.execute("INSERT INTO done VALUES (?,?,?,?)", (m,key,qty,rev+1))
    return rev+1

def txn(db, fn, *args):
    with db:
        db.execute("BEGIN IMMEDIATE")
        return fn(db,*args)

def fence(db):
    return db.execute("UPDATE site SET active=0 WHERE m='A' RETURNING rev").fetchone()[0]

def copy_a(src, dst):
    # Source is quiescent at each of these two deliberate observation points.
    row = src.execute("SELECT m,0,rev,transit,store FROM site WHERE m='A'").fetchone()
    done = src.execute("SELECT * FROM done WHERE m='A'").fetchall()
    with dst:
        dst.execute("BEGIN IMMEDIATE")
        dst.execute("INSERT OR REPLACE INTO site VALUES (?,?,?,?,?)", row)
        dst.execute("DELETE FROM done WHERE m='A'")
        dst.executemany("INSERT INTO done VALUES (?,?,?,?)", done)

def run(root, old_first, contenders):
    src, gate, dst = connect(root/"old.db"), connect(root/"old.db"), connect(root/"new.db")
    setup(src)
    setup(dst)
    src.executemany("INSERT INTO site VALUES (?,1,3,?,?)", [("A",2,4),("B",1,9)])
    src.execute("INSERT INTO done VALUES ('A','accept-17-1',4,3)")
    copy_a(src,dst)
    assert txn(dst,receive,"A","late",1) == "fenced"
    if old_first:
        src.execute("BEGIN IMMEDIATE")
        assert receive(src,"A","late",1) == 4
        try:
            txn(gate,fence)
        except sqlite3.OperationalError as exc:
            assert exc.sqlite_errorcode == sqlite3.SQLITE_BUSY
        else:
            raise AssertionError("fence passed an active writer")
        src.commit()  # Its reply never reaches the scanner.
    cutoff = txn(gate,fence)
    assert cutoff == (4 if old_first else 3)
    assert txn(src,receive,"A","late",1) == "fenced"
    assert txn(src,receive,"B","late",1) == 4
    copy_a(src,dst)
    assert dst.execute("SELECT rev FROM site WHERE m='A'").fetchone() == (cutoff,)
    assert dst.execute("SELECT count(*) FROM site WHERE m='B'").fetchone() == (0,)
    dst.execute("UPDATE site SET active=1 WHERE m='A'")
    assert txn(dst,receive,"A","accept-17-1",4) == 3
    assert txn(dst,receive,"A","late",1) == 4
    assert txn(dst,receive,"A","late",1) == 4
    assert txn(dst,receive,"A","late",2) == "conflict"
    assert [txn(dst,receive,"A",k,1) for k in contenders] == [5,"review"]
    assert dst.execute("SELECT rev,transit,store FROM site WHERE m='A'").fetchone() == (5,0,6)
    assert src.execute("SELECT rev,store FROM site WHERE m='B'").fetchone() == (4,10)
    assert src.execute("SELECT rev,active FROM site WHERE m='A'").fetchone() == (cutoff,0)
    print(f"source fence={cutoff}, late result=4, winner={contenders[0]}, A transit/store=0/6, B store=10")
    for db in (src,gate,dst):
        db.close()

for old_first in (True,False):
    for pair in (("clerk-1","clerk-2"),("clerk-2","clerk-1")):
        with TemporaryDirectory() as temp:
            run(Path(temp),old_first,pair)
```

</details>

