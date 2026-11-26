# Receiving, picking and store pickup on the warehouse floor

Start every task screen with the order or receipt, location, expected item and unit, quantity completed, and one next action. A worker returning from an interruption should be able to answer “¿Qué tengo en la mano y dónde va?” Test the following Spanish prompts with the people who staff the dock, shelves and pickup counter.

## Walk one order through the floor

| Moment | Example on the handheld | What changes |
| --- | --- | --- |
| Receive a delivery | **Recepción 218 · Muelle 1**. “Escanea el producto.” After a case scan show “Arroz 1 kg · 1 caja de 12 unidades”. Offer **Confirmar recepción** and **Informar diferencia**. | Show expected, physically counted and held quantities with their units. For damage, enter quantity, reason and holding location before assigning the issue to a supervisor. Only accepted units become available under this proposed workflow. |
| Put goods away | “Lleva 1 caja a A-03. Escanea la ubicación.” If the worker scans A-08, show “Leímos A-08. Se esperaba A-03.” Offer **Escanear ubicación** or **Solicitar otra ubicación**. | Keep the case in its last confirmed location until the destination is accepted. Preserve the scanned item and quantity through correction. |
| Pick a line | **Pedido 514 · A-03**. “Toma 3 unidades de arroz de 1 kg.” After each deliberate unit scan, show “2 de 3 unidades preparadas”. At shortage, offer **Registrar faltante**. | Capture actual quantity and remaining demand. A shortage leaves the balance open for a supervisor to reassign, approve a partial order or contact the customer. |
| Stage a pickup | “Escanea la bolsa del pedido 514 y la ubicación de retiro.” Then “Pedido 514 · Estante R-02 · 1 de 1 bolsas”. Offer **Marcar listo para retirar**. | The pickup queue shows the staging location. If a transfer is needed, the store verifies physical arrival and completeness before marking the order ready. Track customer notification separately if the store uses it. |
| Hand goods to the customer | **Retiro 514 · R-02**. “Verifica el pedido y escanea la bolsa.” When order and bag agree, offer **Confirmar entrega**. If bag 541 is scanned, show “Esta bolsa es del pedido 541. Busca la del 514 en R-02.” | The clerk confirms the store’s pickup check and the physical handover. A second clerk confirming the same bag sees the existing handover time and operator. If cancellation has claimed the staged quantity, stop collection and show who is checking the bag. If an offline device has conflicting evidence of an actual handover, preserve its observation for review. Shopify documents separate ready-for-pickup and collected milestones in its own workflow. [Shopify pickup workflow](https://help.shopify.com/en/manual/fulfillment/setup/delivery-methods/pickup-in-store) |

## A pallet arrives with an unfamiliar case

Receipt 218 expects 12 one-kilogram bags of rice. At the dock there is one pallet carrying one case. Its labels include a pallet barcode, an unfamiliar case GTIN and a supplier’s short code. The receiver starts with “Escanea la caja de arroz.” Which symbol the reader returns determines the next prompt. [What a GS1 scan identifies](../standards/identification-and-exchange.md#what-a-scan-identifies)

| Reading | Handheld prompt | Effect on this receipt |
| --- | --- | --- |
| Pallet SSCC | “Paleta identificada. Escanea una caja para contar su contenido.” | Link the pallet to receipt 218. Zero bags counted. |
| Supplier code `0042` also used on local labels | “El código 0042 tiene más de un uso. Escanea la etiqueta del producto.” Offer **Buscar producto**. | Keep the scan with the open task. Zero bags counted. |
| Case GTIN with no verified pack size | “Caja sin presentación registrada. Cuenta las bolsas y verifica el producto.” Offer **Contar por unidad** or **Dejar pendiente en el muelle**. | Save one unidentified case in dock custody. Quantity available stays unchanged. |

The receiver opens the case, matches a bag to the known **Arroz 1 kg** variant and counts ten intact bags. The screen asks “Contaste 10 bolsas. Se esperaban 12. ¿Confirmar recepción de 10?” On confirmation, resolve the provisional case into **10 bags received, 10 available, 2 still due**. The case and pallet add zero extra units. Keep the observed count, case code, operator and reason on this receipt. A later delivery using the same unknown case code still requires verification. If staff cannot identify or inspect the contents, leave the case pending for the next shift. [Pack mappings and historical receipts](../operations/catalog-and-units.md)

## A second scan and a second attempt to save

In a browser simulation, a scan directed at the quantity field changed 1 to 1,421, and its Enter suffix submitted the baseline form. A task-scoped scan field plus a separate confirmation kept the changed value in draft for review. [Keyboard-wedge focus trial](scan-focus-trial.md)

For weighed goods, show the confirmed quantity and unit before saving a manual reading. The [Venezuelan entry and display examples](venezuelan-entry-and-display.md) show why the same punctuation can mean different quantities on a scale and in a supplier export.

Two identical product labels can represent two physical units. Give every deliberate unit scan a new local operation ID. A repeated transmission after a timeout reuses the original ID, and the service returns the previously recorded result. For a serialized item or uniquely labeled bag, validate its identifier within the task and explain an earlier acceptance. A scanner in continuous-read mode may decode a label more than once while it remains under the beam. Record the device mode and test single-trigger operation or its repeat timeout before using scan-to-increment. Zebra documents a same-symbol timeout specifically for continuous-read mode. [Zebra DataWedge barcode input](https://techdocs.zebra.com/datawedge/latest/guide/input/barcode/)

| State | What the operator sees and can do |
| --- | --- |
| Intentional second unit | “2 de 3 unidades preparadas · última lectura +1”. Offer **Corregir cantidad** while the task is open and record the correction. In bulk entry mode, one scan selects the product and the worker confirms an explicit count. Label the mode on screen. |
| Unknown barcode or repeated serial | “Código sin identificar. Busca por nombre o SKU.” Offer **Buscar producto** and show the selected unit before counting. For a repeated serial, show “La serie 407 ya se registró en este pedido” with **Ver registro**. |
| Save result unknown | “Verificando si se registró la entrega.” Disable a second handover confirmation while resolving the original ID. On confirmation show “Entrega registrada a las 14:32”. If the system supports durable local capture, a pending action can say “Guardado en este equipo · pendiente de sincronizar”. |
| Concurrent change | “Solo queda 1 unidad disponible. Solicitaste 2.” Show the confirmed count, **Actualizar cantidad** and **Solicitar revisión**. Keep the attempted action visible until resolved. |
| Shift ends mid-task | **Entregar turno** shows the open order, confirmed quantities, physical location, outstanding exception, owner and pending uploads. Incoming staff use **Retomar tarea** under their own login. If a scan exists only on the outgoing device, flag it for recovery there before anyone repeats it. |

## Retomar un cambio pendiente en el mostrador

En Centro, una persona devuelve una camisa azul talla M y pide una L. Caja autoriza el cambio, pero Centro no tiene talla L. La encargada recibe la M, la deja en el estante de devoluciones y entrega la referencia **Cambio 82**. Al pulsar **Retomar cambio**, el siguiente turno ve dónde quedó la prenda, qué autorizó caja y qué falta por entregar.

| Lo que muestra Cambio 82 | Qué puede hacer el siguiente turno |
| --- | --- |
| **Prenda recibida** · 1 azul M · Centro · estante D-02 · pendiente de revisión | Localizar la prenda. Hay una M en custodia y cero M disponibles para la venta por esta devolución. Revisar el caso no vuelve a recibirla. |
| **Decisión de caja** · Cambio autorizado · comprobante C-318 · sin reembolso | Consultar el comprobante que respalda la decisión. El número de cambio o el comprobante permiten retomar el trámite sin pedir nombre ni teléfono. |
| **Entrega pendiente** · 1 azul L · sin reservar · a cargo del encargado de Centro | **Consultar otra tienda**. Mostrar plazo y lugar de retiro solo cuando estén confirmados. Hasta entonces, decir «Ya recibimos la talla M. La talla L sigue pendiente». |

**Si Norte puede entregar la L.** Con la conformidad del cliente, Norte reserva una L disponible para el mismo cambio. Al preparar la prenda marca **Lista para retirar en Norte**. Al entregarla confirma **Talla L entregada** y guarda tienda, hora y referencia de entrega. El caso muestra una L entregada, cero por entregar y el comprobante de cambio de caja. La M sigue en D-02 hasta que Centro registre el resultado de su revisión.

**Si el cliente elige el reembolso.** La persona autorizada verifica primero que Norte no haya entregado la L y que cualquier bolsa preparada siga en la tienda. Cancela la entrega pendiente y libera la reserva, si la había, una sola vez. Mientras caja tramita el pago se lee **Reembolso pendiente**. Con la confirmación de caja y su referencia pasa a **Reembolso confirmado · sin prenda por entregar**. Si ya figura una entrega o falta sincronizarla, se muestra **Verificar entrega en Norte** antes de resolver. El reembolso por sí solo mueve cero prendas. [Devoluciones y cambios entre tiendas](../operations/returns-and-corrections.md)

## Accessibility and device checks

For a web interface targeting WCAG 2.2 AA, the applicable A and AA criteria include keyboard operation, visible keyboard focus, text descriptions of detected input errors, and status messages that assistive technology can announce without moving focus. Color alone cannot convey status. Normal text generally needs at least 4.5:1 contrast. Level AA’s pointer-target criterion uses 24 by 24 CSS pixels with specified exceptions, including spacing. These are selected conformance criteria for the web UI. [WCAG 2.2, criteria 1.4.1, 1.4.3, 2.1.1, 2.4.7, 2.5.8, 3.3.1 and 4.1.3](https://www.w3.org/TR/WCAG22/)

For the pilot, pair a persistent text result with optional sound or vibration. A scanner beep may precede a rejected item or a pending save. Keep the last accepted item and count visible, announce meaningful changes without reading the whole screen, and keep the scan field ready after a successful scan. Give workers a manual lookup for faded labels. Test buttons with actual gloves and one-handed use. For native Android, Google recommends touch targets of at least 48 by 48 dp. Measure web targets in CSS pixels and native targets in dp. [Android accessibility guidance](https://developer.android.com/guide/topics/ui/accessibility/apps)

## Run the floor test

Use a test order and marked sample stock. Recruit at least one person from receiving, picking and the pickup counter, plus the supervisor who resolves exceptions. Observe the existing process first. Use the site’s handheld, shared workstation and real packaging under normal lighting and noise. Include a keyboard-only run and a run with the supported screen reader. Record task time, incorrect taps, requests for help, physical counts, WMS balances and the final posting reference. If capture or recovery varies by client, repeat the same task on the candidate devices. [Browser and native floor-device trial](../architecture/web-to-mobile.md#floor-device-trial)

| Trial | Proposed pilot acceptance check |
| --- | --- |
| Receive two 12-unit cases, quarantine two damaged units and pick three good units. | Physical count and system agree on 19 available, two held and three picked. Every participant can state the unit and next action unprompted in their role. |
| Scan two identical units deliberately, hold one label under continuous read, then replay a confirmed save after a lost response. | Two intentional scans count as two. The held-label trial counts one under the selected scanner profile. Replaying the save adds zero movements. |
| Scan a wrong bin, read a damaged label and scan the wrong pickup bag. Repeat the pickup confirmation on another clerk’s device after collection. | Wrong bin and bag trigger a recoverable, readable error. A worker completes manual lookup unaided. The pickup has one handover, and the wrong bag stays staged. |
| Pause with one shortage at shift change, then resume on a second device. | The incoming worker identifies the physical location, last confirmed quantity and next owner within one minute. No quantity is re-entered. Keyboard and screen-reader users can identify and clear the exception. |

The Spanish text, one-minute threshold and task choices are prototype decisions. Review timings with the site lead against the observed baseline and repeat any failed task after revisions. These checks have yet to be run at a warehouse.
