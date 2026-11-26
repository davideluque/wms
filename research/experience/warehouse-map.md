# Finding a bin on a warehouse map

For a small merchant, start with a searchable address list and one 2D plan per floor. A picker selecting a task sees the aisle, the rack face, and the level and bin printed on the physical sign. The plan shows the approach from a named landmark such as receiving. A location without mapped geometry is still searchable and scannable. A map here is a wayfinding aid for existing premises. Structural dimensions, load ratings and regulated egress design require separate authoritative plans.

## Rendering options

| Approach | Use it when | Implementation consequence |
| --- | --- | --- |
| SVG plus ordinary HTML list | One floor has a manageable number of labeled racks. This is the proposed pilot. | Store floor coordinates once, use an SVG `viewBox` for scaling, and link selected shapes and list rows by location ID. Offer labeled buttons and keyboard focus. [W3C SVG coordinate systems](https://www.w3.org/TR/SVG/latest/coords.html) |
| Canvas | Profiling on the merchant’s device shows that redrawing a dense plan or many markers is too slow with the SVG implementation. | Draw only visible geometry, implement hit testing and maintain a synchronized HTML list. Drawn objects do not provide their own DOM focus targets. [MDN drawing graphics](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Client-side_APIs/Drawing_graphics) |
| Leaflet with `CRS.Simple` | Staff need pan, zoom, image overlays and selectable markers across a large non-geographic floor. | Define bounds in map units, independently of image pixel dimensions. Leaflet takes points in `[y, x]` order. Its vertical map axis increases upward, so adapt a downward-increasing floor axis at the renderer boundary. [Leaflet non-geographical maps](https://leafletjs.com/examples/crs-simple/) |
| Georeferenced map and GeoJSON | A campus or carrier workflow needs actual outdoor positions or a surveyed tie between building and earth coordinates. | Keep the local floor model. Store the independently verified transform for an external geographic layer. RFC 7946 coordinates use WGS 84 longitude and latitude in degrees. An indoor metre pair is unsuitable as a GeoJSON position. [RFC 7946, §4](https://datatracker.ietf.org/doc/html/rfc7946#section-4) |
| 3D or rack elevation view | Shift tests show that workers find the aisle correctly but repeatedly pick the wrong vertical level. | Try a labeled 2D rack elevation first. A 3D view adds geometry, camera interaction and device requirements. Benchmark this with a real vertical-picking task before adopting WebGL. [MDN WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API) |

All rendering options should offer the same actions through search and a list grouped by floor and aisle. Include the rack face, level, bin, operational status and a readable approach. Keep filtering, floor selection and zoom controls usable by keyboard and touch. For map editors, supply coordinate fields or tap-to-place alongside drag. A stale or missing floor plan should lead directly to the address and manual scan workflow. [W3C keyboard operation](https://www.w3.org/WAI/WCAG22/Understanding/keyboard) · [W3C alternatives to dragging](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements)

## Floor coordinates and calibration

Choose a fixed origin for each floor with x increasing rightward and y downward on its published plan. Use metres for measured floors. Label an unmeasured sketch as schematic and omit distance estimates. Store floor ID, coordinate convention, source plan revision, pixel-to-floor transform, control points and date verified. Represent an aisle or rack by its footprint, with a separate approach point on the accessible face. Link the location’s immutable ID to its floor feature and level. Multiple levels can share a footprint. Stock remains attached to the bin ID when a drawing is replaced.

**Example fixture.** Use a 12 m × 8 m room and a raster 1,200 × 800 pixels after cropping. The example origin is the top-left corner. Two marks on a horizontal wall at pixels (100, 100) and (900, 100) correspond to floor points (1, 1) m and (9, 1) m. That gives 100 pixels per metre. Check an independent point away from that line, e.g. a column at (300, 600) pixels should fall at (3, 6) m. Record the check error and the merchant’s tolerance for distinguishing neighbouring bays. For an image rotated or offset at import, solve for translation, rotation and uniform scale from known point pairs. If the independent check fails, inspect crop, aspect ratio or distortion and recalibrate before showing distances. [QGIS control points and transformations](https://docs.qgis.org/testing/en/docs/user_manual/working_with_raster/georeferencer.html)

At the fixture scale, a pick-face marker for `UBI-0047` at (6.5, 3) m appears at pixel (650, 300). SVG can use `viewBox="0 0 12 8"` with metre-based geometry. For Leaflet with a lower-left map origin and an 8 m floor height, adapt that marker to `[8 - 3, 6.5]`, or `[5, 6.5]`. Pinch zoom, screen rotation or replacing the raster must preserve the saved floor coordinates. Show “Nivel 02 · Posición B” in the selected-bin detail. Drawing all levels at slightly different floor points would imply different walking destinations. [SVG viewport transforms](https://www.w3.org/TR/SVG/latest/coords.html) · [Leaflet coordinates and bounds](https://leafletjs.com/examples/crs-simple/)

### Approach to UBI-0047 from receiving

The plan covers the 12 m × 8 m fixture. Four columns represent one horizontal metre and each row represents one vertical metre. From receiving, follow aisle A to **P**, the approach point at (7.5, 3) m. Face left toward **F**, rack face E at (6.5, 3) m. **C** marks the independent calibration column at (3, 6) m.

```text
     0 m                     6 m                    12 m
0 m  +-----------------------------------------------+
1 m  |               +---------+                     |
2 m  |               | Mod. 03 |                     |
3 m  |               |         F<--P  Pasillo A      |
4 m  |               +---------+   ^                 |
5 m  |                             |                 |
6 m  |           C                 |                 |
7 m  |                             ^ Recepción       |
8 m  +----------------------------   ----------------+
```

Standing at P and facing the rack, read the level numbers from the floor upward. Select **nivel 02, posición B** and scan the physical location label.

```text
Módulo 03 · Cara E         Posición A      Posición B
                        +---------------+---------------+
Nivel 03                |               |               |
                        +---------------+---------------+
Nivel 02                |               | >> UBI-0047   |
                        +---------------+---------------+
Nivel 01                |               |               |
Piso                    +---------------+---------------+
```

The upper view locates the rack face. The elevation identifies one bin on it. Both resolve to the same location record, where the WMS can look up stock by its immutable ID.

## Routing, editing and disconnected work

For the first floor, show “Desde recepción, siga el pasillo A hasta el módulo 03, cara E.” A rack marker alone establishes a target. A computed route additionally requires a verified graph of walkable aisle segments, connected intersections, permitted directions and closures. End routes at the approach point. Until staff have walked the graph, offer the address and landmark directions. Keep the date and owner for layout verification.

Pickers can read the latest published revision and flag a missing sign or inaccessible approach. Give named site editors draft access and the site supervisor publication rights. Cache the last published plan and its timestamp for disconnected picking. An offline draft records its base revision, feature ID and changes. On reconnect, compare with the current revision, flag competing edits to the same feature and require an explicit choice. A retired or moved target requires a fresh address check before confirming work. Editing a shape or tapping a marker does not post a stock movement.

**Acceptance walk.** Place a marked obstruction between receiving and the sample rack. Search the location, select it from the plan, and walk to the correct rack face from each entrance. Test the list with keyboard only and with the image unavailable. Scan the bin at level 02. Rotate the device and verify the marker stays on the same face. Offline, change the marker in a draft while a second editor changes it online. After reconnecting, retain both proposals for resolution and serve the previous published version until approval. If routing is enabled, test that it reaches the marked approach using open aisles around the obstruction. Record time to find, wrong-face picks and any need to ask a coworker. [Physical addresses and signs](../operations/location-addressing.md)
