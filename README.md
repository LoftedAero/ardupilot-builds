# Lofted Aero flight controller firmware

Builds of [LoftedAero/ardupilot](https://github.com/LoftedAero/ardupilot) for Lofted Aero
flight controllers. This repository holds no source code; each release's `version` names the
commit it was built from. It is the catalog Lofty GCS reads to set up and update these boards
(the "Lofted Aero Flight Controllers" card on its Firmware screen), from the `main` branch.
Lofty GCS reads it from `raw.githubusercontent.com`, which serves files with CORS headers, so
the browser version can read it too; GitHub release downloads cannot be fetched from a web page.

```
index.json                      { "format": 1, "releases": ["2026-11-13", "2026-11-07"] }
releases/<release>/boards.json  the boards in that release and their firmware options
releases/<release>/<files>      the images boards.json names
```

## Adding a release

1. Make a folder under `releases/`. Its name is one path segment (letters, digits, `.`, `_`,
   `-`); a date works.
2. Put the images in it, two per firmware option, both from the same build: the `.apj`, and the
   `_with_bl.hex` (the ArduPilot bootloader with that firmware behind it). Lofty GCS writes the
   `.hex` to a blank board over the STM32's ROM DFU, and the `.apj` to a board that has the
   bootloader.
3. Write its `boards.json` (below). Every file needs its exact size in bytes and its SHA-256
   (`sha256sum`); Lofty GCS refuses a file that does not match.
4. Add the folder's name to `index.json`.

Never change a published release's files: users may install any release, and the hashes are
what prove a file is the one published. Publish a new release instead.

## boards.json

```json
{
  "format": 1,
  "version": "4.8.0-dev 78b82326",
  "date": "2026-11-13",
  "boards": [
    {
      "id": "f405-srx",
      "name": "F405-SRX",
      "boardId": 5999,
      "receiver": { "target": "newbeedrone.rx_2400.c324d", "serial": 2, "bootPin": 71, "enPin": 70, "baud": 420000 },
      "firmware": [
        {
          "id": "stabrx",
          "name": "StabRX",
          "file": "F405-SRX-StabRX.apj",
          "size": 563996,
          "sha256": "…",
          "withBootloader": { "file": "F405-SRX-StabRX_with_bl.hex", "size": 1854676, "sha256": "…" },
          "passthroughProtocol": 28
        }
      ]
    }
  ]
}
```

- `version` and `date` are shown in the release list; releases are ordered by `date`.
- `id` is the board's permanent key: Lofty GCS keys its setup pictures on it, so keep it when
  the board is renamed. `name` is what users see, and the newest release that lists a board
  decides it.
- `boardId` is the hwdef's `APJ_BOARD_ID`. Every `.apj` must carry it.
- `firmware[].withBootloader` is the build's `_with_bl.hex`. It must start at `0x08000000` and
  hold the same firmware as the `.apj`; Lofty GCS's live test checks both.
- `receiver` is the onboard ExpressLRS receiver: its ExpressLRS target, the `SERIALn` it is on,
  the ArduPilot GPIO numbers of the ESP32's BOOT and EN lines, and the CRSF baud the build
  expects. Leave it out for a board without one.
- `firmware[].passthroughProtocol` is the `SERIALn_PROTOCOL` used while the receiver is written:
  a protocol that build does not compile in (28, Scripting, while `AP_SCRIPTING_ENABLED` is 0).
- `firmware[].id` picks the tile Lofty GCS shows the option on: `ardupilot`, `stabrx`,
  `f35b-70` or `f35b-90`. An id Lofty GCS does not know is not offered, so a new product needs
  a Lofty GCS release too. Lofty GCS shows its own name for each id; `name` is for readers of
  this file.
- A release may list only some of a board's firmware options; each option's release list shows
  the releases that have it.
