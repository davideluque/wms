# Giving shelves and bins usable addresses

A picker standing at the receiving door should be able to find a bin from the words on a task and the signs on the floor. Walk the site from both entrances with staff from different shifts. Ask each to name the same shelf and point to its first position. Record the terms they agree on, the places with duplicate names and the spots where their directions diverge. Microsoft’s warehouse location formats and Oracle’s location master both support segmented addresses and separate operational settings such as location type or pick sequence. [Microsoft location formats](https://learn.microsoft.com/en-us/dynamics365/supply-chain/warehousing/tasks/configure-locations-wms-enabled-warehouse) · [Oracle location master](https://docs.oracle.com/en/cloud/saas/warehouse-management/25d/owmol/location-master.html)

## A scheme staff can read from either end of the aisle

| Address component | Example at the pilot site | Sign or convention to agree |
| --- | --- | --- |
| Site, floor and zone | Depósito Centro, planta baja, seco | Use the site and floor in search results. Put the zone name at the entrance. A small stockroom can omit hierarchy levels it never uses. |
| Aisle and bay | Pasillo A, módulo 03 | Place the aisle letter at both ends and indicate which way the numbers increase. Choose one fixed reference end, such as receiving. Signs for bays remain readable when staff approach from dispatch. |
| Face, level and bin | Cara E, nivel 02, posición B | Identify facing rack sides with permanent face letters or marked landmarks. Number levels bottom-up. Mark the start of the bin sequence on the rack face. Confirm which side of the shelf-edge divider owns each label. |
| Operational area | Recepción, Retiro, Devoluciones | Give each area that can hold stock an address. Keep the area’s pick eligibility and the goods’ condition as separate data. For example, a customer’s staged order remains committed even though the pickup bay is accessible. [Microsoft location profiles](https://learn.microsoft.com/en-us/dynamics365/supply-chain/warehousing/tasks/create-location-profile) |

For the example bin, show **Seco · A-03-E-02-B** with the expanded description “Pasillo A, módulo 03, cara E, nivel 02, posición B.” Print a scannable stable internal ID, such as `UBI-0047`, beside it. A location scan should resolve its site and readable address. Accept manual entry when a label is damaged. For a wall or endcap, describe its named fixture directly. A fictional aisle and rack number would make the search harder.

Store each location’s immutable ID, site, parent, active address, aliases, role, operational status and optional map spot. Require unique current addresses within a site and floor, and unambiguous scan IDs within the merchant. Keep inventory and task references on the immutable ID. Aisle headings help navigation. Use identified leaf bins for stock, or create an explicit floor-storage bin when a whole marked area holds goods. This avoids counting a rack and its contents twice.

## What changes when someone moves a shelf?

| Event | Catalog, signs and task handling | Inventory effect |
| --- | --- | --- |
| Same physical bin renamed | Keep its ID. Change the displayed address, update signs and keep the old address as a dated search alias. Open tasks display the current address. Retain the old description on posted movements. | Zero units move. |
| Item assigned a new home | Change the SKU’s preferred putaway/pick bin. Existing stock and outstanding picks still point to their actual bins. For a physical move, scan the source, destination and quantity. | Post the quantity actually moved. Five shirts transferred out of A and into B leave site on-hand unchanged. |
| Location closed, rack replaced or divided | Agree what happened to the physical places. Retire superseded IDs, create destinations, move or count remaining stock, reassign open work and remove obsolete signs. Preserve old IDs for historical lookup. | Each physical quantity has a destination, a recorded disposition or an open exception. A map edit alone changes zero stock. |

Keep the proposed walking sequence in its own field. Addresses remain usable when a receiving door closes or the team changes its pick order. Avoid encoding the current SKU, owner, balances or the worker’s direction of travel in an address. If a partner needs a standardized external location identifier, attach its GLN or agreed sublocation identifier alongside the local ID. [Oracle pick sequence](https://docs.oracle.com/en/cloud/saas/warehouse-management/25d/owmol/location-master.html) · [GS1 GLN sublocations](https://ref.gs1.org/standards/gln-allocation/3.0.2/)

## Floor acceptance

Give a first-shift and a second-shift picker **Seco · A-03-E-02-B**. Start one at receiving and the other at dispatch. Both should find the same level and scan `UBI-0047`. Ask them to find it again by typing the address with the map unavailable. Reassign the shirt’s preferred bin and verify that the existing stock stays put until a physical transfer. Rename the original bin and retry an older pick ticket. The device should show the verified current address and preserve the historical one. Record wrong-side picks, duplicate labels, illegible signs and where verbal directions were needed. [Internal label field tests](../experience/labels-and-printing.md) · [Indoor map and calibration](../experience/warehouse-map.md)
