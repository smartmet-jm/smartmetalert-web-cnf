# smartmetalert-web-cnf

SmartMet Alert's `capmap-config.js` and `index.html` for Jamaica
(`alert.metservice.gov.jm`), kept in git so that a change to them gets a diff
and a review — they are served to end users as JavaScript.

## Deploying

The container bind-mounts these two files over the ones in the image, so
**committing here is not deploying**. Copy them to whatever path the
`smartmetalert` service mounts and recreate the container.

**The `smartmetalert` stack is not in `smartmet-jm/portainer`** — only
`weather-app-warnings-backend`, which routes the `/v1` prefix of the same host
and consumes `https://alert.metservice.gov.jm/capfeed.php`. So the container
exists but its compose file was never committed, and the mount path needs
confirming on the host. For reference, Colombia uses
`/smartmet/cnf/smartalert/` (see `smartmet-co/portainer`).

## Why these files changed

The app image moved to the 2026-10-21 build (`0de2f65`, the PHP 8.5.12 base
image fixing CVE-2026-109618 / CVE-2026-109619). Three things then break
against the old mounted files:

- **Menus disappear.** The app's own `index.html` gained a
  `#topright-toolbar` / `#filter-panel` structure (`select#eventType`,
  `select#lang`, `#legend-warning-types`) that the rewritten `js/capmap.js`
  populates. Mounting the old file over it leaves nothing to populate.
  Confirmed on the live site: the served `index.html` had **zero** matches for
  that DOM. The copy here is the app's own `index.html` at `0de2f65` with only
  the translation block swapped to `en-JM`.
- **Warning polygons turn transparent.** `js/capmap.js` used to hardcode
  `fillOpacity: 1` and ignore the config; it now reads
  `polygonOptions.fillOpacity || 0.85`. Jamaica's long-standing `0.2` was dead
  config and became live. Set to `1`, which is what the old app drew.
- **The day menu crowds.** Jamaica shows six day buttons (`day0`–`day4` plus
  `allDay`). Without `showDayNames`, each takes a long label — "Day after
  tomorrow", "Day 4", "Day 5" — while the rewritten CSS lays the row out
  `flex-wrap: nowrap` with `min-width: 80px` per button, a design for short
  two-line content. `showDayNames: true` gives a weekday over a date instead.

Cosmetic gap: `en-JM` carries no weekday keys, and `t()` falls back to the key,
so buttons will read `Mon`, `Tue`… in English until weekday translations are
added to `fmidev/smartalert-web`.
