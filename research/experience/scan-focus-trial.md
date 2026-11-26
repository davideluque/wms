# When a receipt scan lands in the quantity field

A synthetic scan of `421` appended to a quantity of `1` and posted **1,421 units** in a browser form that allowed Enter to submit. A second form limited scan completion to the receipt’s focused scan field and required a separate confirmation to post. It showed the mistyped quantity as a draft and posted **zero**. The worker still had to correct the field before confirming.

## What ran

On 26 November 2026, Playwright 1.60.0 drove headless Chromium 148.0.7778.0 on Linux against temporary local HTML. Each case began on receipt A, with zero units posted, quantity 1, an empty note and one synthetic item code, `421`. The driver used `page.keyboard.type("421")` followed by `page.keyboard.press("Enter")`. Zebra documents that DataWedge keystroke output emulates user keypresses and that its formatting settings can append Enter or Tab. This trial simulated Enter only. [Zebra keystroke output](https://techdocs.zebra.com/datawedge/latest/guide/output/keystroke/) · [Zebra basic data formatting](https://techdocs.zebra.com/datawedge/latest/guide/process/bdf/)

**Baseline form.** Enter in the scanner field immediately posted one recognized item. Enter in the single-line quantity or note field submitted the form using its current quantity. Switching tasks refocused a shared scanner field that still contained its previous characters.

**Task-scoped form.** Choosing **Escanear** armed the scan field for the active receipt. Leaving it cleared its partial input. Enter in that field could add one recognized item to the receipt draft. Enter in a manual field showed **Revisa la entrada manual**. A separate **Confirmar** button posted the reviewed quantity and could be activated with a keyboard. Changing tasks cleared the buffer, disarmed capture and moved focus to the new receipt heading.

## Observed results

| Test action | Baseline | Task-scoped form |
| --- | --- | --- |
| Put the cursor after quantity 1, type `421` and press Enter | Quantity became 1,421. Posted 1,421 to A. | Quantity still became 1,421. Prompted for review. Posted zero. |
| Focus the note, type `421` and press Enter | Note became “421”. Posted the default quantity of one to A. | Note still became “421”. Prompted for review. Posted zero. |
| Complete two separate scans of the same item in the scan field | Posted two units to A. | Showed two draft units and zero posted. Confirming posted two. |
| Type `42` on A, switch to receipt B, then type `1` and press Enter | Completed the old buffer and posted one unit to B. | Displayed **Escaneo en pausa** on B. Both receipts stayed at zero. |

Two additional checks exercised manual recovery in the scoped form. Scanning `999` displayed **Código sin identificar** and added zero draft units. Entering quantity 2 and the note “Caja abierta,” then pressing Enter in the note, posted zero. Focusing **Confirmar** and activating it with Enter posted exactly two.

## When Confirmar already has focus

A follow-up used the same task-scoped receipt fixture, browser and Playwright version. Each case began with three units entered manually and zero posted. The test focused the actual enabled **Confirmar** button, either directly on the form or inside a native `<dialog>` opened with `showModal()`. Typing `421` alone posted zero. The simulated Enter suffix then **posted all three units** in both cases and closed the modal when present. The digits did not change the quantity. The focused button’s ordinary keyboard activation committed the existing draft.

| Follow-up trial | Observed in Chromium |
| --- | --- |
| Install a handler on confirmation buttons that remembers a digit keydown while focus stays on the button. Suppress the next Enter on that button, clear the flag and prompt for review. | With `421` then Enter, both forms posted zero. The dialog stayed open. A second Enter without preceding digits posted three in both forms. |
| Open the guarded dialog with focus on its summary heading instead of a button. Send `421` and Enter. | The dialog stayed open and posted zero. In two fresh runs, Tab reached **Volver**, a second Tab reached **Confirmar**, and deliberate activation by Enter or Space posted three. |

For the pilot, open review on a readable summary of the receipt, bin and quantity. Keep **Volver** and **Confirmar** accessible by keyboard. The tested digit guard covered one specific input sequence. A lone Enter arriving after focus changes has the same effect as an intentional Enter when a commit button is focused. Avoid inferring operator intent from key speed. For installations needing stronger separation, test a scanner profile without an Enter suffix, or an Android integration that receives scanner data as an explicit intent payload and leaves manual keyboard input on its own path. Neither option was exercised here. [Zebra formatting options](https://techdocs.zebra.com/datawedge/latest/guide/process/bdf/) · [Zebra Intent Output](https://techdocs.zebra.com/datawedge/latest/guide/output/intent/)

## Decision for the floor pilot

Show the receipt number, active entry mode and recognized unit beside the scan field. Keep manual typing available. Ordinary wedge characters can still enter a focused quantity or note, so show the exact item, unit and quantity at confirmation. If a proposed receipt exceeds the expected quantity, offer **Revisar diferencia** before posting. A delayed suffix, repeated physical trigger or different scanner profile needs a test on the merchant’s device. A keyboard stream alone could not identify whether the digits came from a scanner or a person.

The assertions checked browser focus, draft state, error text and in-memory posted totals. They did not exercise scanner hardware, wedge timing, device auto-repeat, IME composition, assistive software, persistence, retries or server-side deduplication. On the floor, repeat the tests under the configured scanner profile, including with focus on buttons and dialogs and when an operator starts a new receipt. [Receiving and pickup workflow](floor-workflows.md)
