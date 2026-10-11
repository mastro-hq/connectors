---
name: mastro-reservecalifornia-search
description: Find California State Parks campsites and check night-by-night availability on ReserveCalifornia via the mastro CLI — no login needed. Use when the user asks about camping availability at a California state park ("is anything open at Big Sur next weekend?", "check reservecalifornia.com/park/661/518 for October").
---

# Search ReserveCalifornia campsites

Agent-facing playbook for finding campsites and their availability. If `mastro`
is not on PATH, invoke it as `npx -y mastro-connect`.

## Preconditions

None. Search is public: no `mastro login`, no browser, no extension.

## Ids

Everything is keyed by three numbers. A site URL like
`https://www.reservecalifornia.com/park/661/518` gives you the first two
directly:

| Id | Meaning | Where from |
| -- | ------- | ---------- |
| PlaceId | the park (661 = Julia Pfeiffer Burns SP) | `parks`, or the URL's 1st number |
| FacilityId | a campground/area in it (518) | `park`, or the URL's 2nd number |
| UnitId | one site (41990 = JPB2) | `availability` |

## Commands

```bash
mastro reservecalifornia parks "<name fragment>" --json          # → PlaceId
mastro reservecalifornia park --park <PlaceId> --date YYYY-MM-DD [--nights N] --json
mastro reservecalifornia availability --facility <FacilityId> --start YYYY-MM-DD [--end YYYY-MM-DD] --json
mastro reservecalifornia unit <UnitId> --date YYYY-MM-DD [--nights N] --json
```

Flags are generated from the spec. `--help` on any command is authoritative.

## Reading results

- `park` returns the park with `Facilities` (an object keyed by FacilityId).
  Each facility has `Name`, `Available` (some site is free for the whole
  stay), and `RateMessage`.
- `availability` returns the facility with `Units` (keyed by UnitId). Each
  unit has `Name`/`ShortName` and `Slices`, one per night:
  `{Date, IsFree, IsBlocked, IsWalkin, MinStay}`. **`IsFree: true` means
  bookable.** For a multi-night stay, every night in the range must be free on
  the same unit. `Restrictions` gives the booking window
  (`FutureBookingStarts`/`FutureBookingEnds`, typically 6 months out) and
  `MinimumStay`/`MaximumStay`.
- One `availability` call covers at most ~21 nights. Page through longer
  ranges with successive `--start`/`--end` windows.
- `unit` gives `Rate` (per night), `Fee` (reservation fee), and limits in
  `NightlyUnit` (`MaxAdult`, `MaxOccupancy`, `MaxVehicles`, check-in/out
  times).

## Tips

- Popular sites sell out the moment their window opens. Cancellations
  reappear as `IsFree` slices: check again when the user asks, don't poll in
  a loop.
- To book a site, send the user to the site's own page:
  `https://www.reservecalifornia.com/park/<PlaceId>/<FacilityId>`.
