# Offline floor capture with IndexedDB

A scanner operator needs to know whether a count survived a lost connection and whether retrying will record it twice. This browser trial tested the local part of that workflow. One IndexedDB transaction wrote a physical observation and its pending sync command. The UI could report “saved on this device” after the transaction completed. Server acceptance would require a separate acknowledgement. [IndexedDB transaction lifecycle](https://www.w3.org/TR/IndexedDB-3/#transaction-lifetime-concept)

## Observations from the trial

On 26 November 2026, Playwright 1.60.0 drove local headless Chromium 148.0.7778.0 on Linux using a fresh temporary persistent profile. Two loopback HTTP servers served the same synthetic page on different ports. Paths `/m/a` and `/m/b` on the first port represented merchants A and B. Each operation opened and closed the database. The fixture used fictional bin IDs and quantities and no external service.

| Operation | What a subsequent read found |
| --- | --- |
| Save A’s `cmd-1`, bin A-04, quantity 3 | One observation and one pending command with the same merchant, command ID and quantity. |
| Start A’s `cmd-abort`, quantity 11, then abort in the observation write’s success callback before adding its command | The transaction aborted. Both stores contained only the earlier `cmd-1`. A successful individual write callback preceded the rollback. |
| Reload the page, retry `cmd-1` with 3, then reuse it with 4 | The page recovered the original pair. The identical repeat returned `replayed`, the changed payload returned `payload conflict`, and both stores still held one record of 3. |
| Open merchant B on the same origin, then save its own `cmd-1`, bin B-02, quantity 7 | B’s indexed read began empty and returned only B’s pair after the save. An unrestricted read from B’s page returned **both merchants’ records**. |
| Close Chromium cleanly and relaunch with the same profile | A fresh connection found both merchants’ pairs, with quantities 3 and 7. |
| Open an identically named database on the second loopback port | Both stores were empty. Returning to the original port recovered A’s records. Browser storage separation follows the origin. Both paths on a shared origin remained in the same storage boundary. [MDN same-origin storage](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy) |

## Reproduce the core transaction

On a fresh loopback origin, open a version-1 database named `floor-trial`. Run this setup during `onupgradeneeded`, using the upgrade request’s database as `db`.

```js
for (const name of ["observations", "queue"]) {
  const store = db.createObjectStore(name, { keyPath: ["merchant", "command"] });
  store.createIndex("byMerchant", "merchant");
}
```

After opening the database, use the following function. The final parameter injects the abort for this experiment. `bin` and `units` are the entire command payload in this fixture.

```js
function capture(db, merchant, command, bin, units, interrupt = false) {
  return new Promise((resolve, reject) => {
    const key = [merchant, command];
    const tx = db.transaction(["observations", "queue"], "readwrite");
    const queue = tx.objectStore("queue");
    let outcome = "saved";
    tx.oncomplete = () => resolve(outcome);
    tx.onabort = () => reject(tx.error || new Error(outcome));
    const existing = queue.get(key);
    existing.onsuccess = () => {
      const prior = existing.result;
      if (prior) {
        if (prior.bin === bin && prior.units === units) outcome = "replayed";
        else { outcome = "payload conflict"; tx.abort(); }
        return;
      }
      const record = { merchant, command, bin, units };
      const first = tx.objectStore("observations").add(record);
      first.onsuccess = () => {
        if (interrupt) { outcome = "injected abort"; tx.abort(); return; }
        queue.add({ ...record, state: "pending" });
      };
    };
  });
}
```

Call `capture(db, "a", "cmd-1", "A-04", 3)`, then call it with `"cmd-abort"`, `11` and `true` to inject the fault. Inspect both stores through their `byMerchant` indexes with `getAll("a")`. Reopen the database after a reload, retry the original command and then vary its units. Repeat on the second merchant path and second port using the table above as expected results. Generate the command ID once for a new operator action and reuse it for that action’s retries. Two deliberate observations of the same SKU need distinct command IDs.

## What the pilot still needs to prove

### Changing the address on an active scanner

A second browser trial used two loopback HTTP servers on different ports with Chromium 151.0.7922.34. On the source receiving page, save one four-bag observation for merchant A under command `R-218-partial` in IndexedDB and await the transaction commit. Switch the source receiving route to a 302 redirect to the destination receiving route. Reopen the original bookmark in the same browser profile. Open the same database name and read the queue at the destination, then visit a recovery page still served directly by the source origin and read again. The destination returned **zero** commands. The source recovery page returned the original ID, merchant and quantity. The fixture used local ports and one browser profile. It exercised neither authentication nor synchronization to a server. [Browser origin and storage isolation](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy)

For cutover, list each active device with its old origin, pending command count, oldest observation and last server acknowledgement. A device that has not checked in has an **unknown** count. Before redirecting its normal bookmark, drain the queue or give the shift lead a recovery path on the old origin. Authenticate the operator, show the original merchant and site, and reconcile each original command ID against the active installation. Keep the old stock writer fenced. A missing or wiped device needs a physical receiving check and a named discrepancy owner. Sign off each device when the accepted receipt or documented discrepancy accounts for its work. Check printed URL labels separately because they can still point to the old host after all devices are clear. [Printed label migration](../experience/labels-and-printing.md#taking-printed-labels-to-a-new-installation) · [Tenant move and interrupted receipt](tenant-export-example.md)

### Other device and synchronization limits

- Bind the merchant identity at capture time and scope every queue read, count and send to that merchant. On account switch, make pending work visible only to its authorized operator. Same-origin filtering is an application convention. A shared device with stronger confidentiality requirements needs a separately evaluated isolation design.
- The replay check retained its queue rows. After successful sync, retain a durable command receipt for the supported retry horizon or check the immutable observation by the same key. Persist the server’s acknowledgement and enforce the same command identity and payload check on the server. This trial exercised neither server sync nor duplicate rejection after queue cleanup.
- Clean restart succeeded. Browser eviction, clearing site data, private browsing, a full disk, abrupt power loss or loss of the device can remove unsynced work or prevent a save. Show a storage-write failure to the operator, track the age of pending work and test recovery on the actual handheld. The trial did not measure throughput, mobile WebViews, prolonged disconnection or concurrent tabs. [MDN storage quotas and eviction](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)
