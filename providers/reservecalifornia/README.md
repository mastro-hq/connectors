# ReserveCalifornia connector

Searches California State Parks campsites on
[reservecalifornia.com](https://www.reservecalifornia.com) — parks,
campgrounds, night-by-night availability, site details — and shows your own
reservations. Search needs no login, no browser, and no extension. It does
not book: to reserve, use the site.

> **Unofficial — not affiliated with California State Parks or Tyler
> Technologies.** It calls the same JSON APIs the site's own frontend calls,
> at a capped rate.

```bash
# Search — public, no login needed
mastro reservecalifornia parks "pfeiffer"                       # → PlaceId 661
mastro reservecalifornia park --park 661 --date 2026-10-14      # campgrounds → FacilityId 518
mastro reservecalifornia availability --facility 518 --start 2026-10-13 --end 2026-10-19
mastro reservecalifornia unit 41990 --date 2026-10-14           # one site: price, limits

# Your account (login required)
mastro login reservecalifornia
mastro reservecalifornia me
mastro reservecalifornia reservations --from 2026-01-01
```

The site's own URLs map straight onto the ids:
`reservecalifornia.com/park/<PlaceId>/<FacilityId>`. For example,
`/park/661/518` is Julia Pfeiffer Burns SP → Environmental Group Camp Area,
whose sites are JPB1 (UnitId 41989) and JPB2 (UnitId 41990).

The connector is described by [`openapi.yaml`](openapi.yaml) (a valid OpenAPI
3.1 document) plus [`auth.manifest.json`](auth.manifest.json) for the browser
capture.

## Two APIs

The site is a React SPA (Tyler Technologies' recreation platform, formerly
"UseDirect") over two JSON APIs. It reads both base URLs from
`https://www.reservecalifornia.com/config.json`:

| Host | `config.json` key | Auth | Used for |
| ---- | ----------------- | ---- | -------- |
| `california-rdr.prod.cali.rd12.recreation-management.tylerapp.com/rdr` | `rdrApiUrl` | none | parks, availability grid, unit details |
| `rdapi.reservecalifornia.com/api` | `rdApiUrl` | Okta bearer | your account |

Neither host has a bot wall. Search operations declare `security: []` (public,
so no credential is sent and no login is needed), their own `servers`, and
`x-mastro-transport: direct`.

## How auth works

Login is Okta SSO (`signin.reservecalifornia.com`). After the redirect back,
the SPA stores the Okta access token in **plain localStorage** as `accessToken`,
and the same JWT again as `ssoAccessToken`. Each rdapi call sends:

| Header | Value |
| ------ | ----- |
| `Authorization` | `Bearer <accessToken>` |
| `accessToken` | `<ssoAccessToken>` |
| `InstallationsIdentity` | `cali` (tenant, fixed) |
| `StoreId` | `111` (web store, fixed) |

`mastro login reservecalifornia` opens the site and waits until both
localStorage keys exist. If you're already signed in, that's immediate.
The JWT **expires after 60 minutes**. After that, rdapi answers `401` and
you run `mastro login reservecalifornia` again. Okta's own token storage
(`@secure.s.okta-token-storage`) is encrypted with a browser-bound key, so the
refresh token isn't usable from here.

## How replay works

Account calls run through the mastro extension (`via_browser`), and with
`browser_tab_origin` they run from your own `www.reservecalifornia.com` tab.
rdapi's CORS policy admits that origin with credentials, so the fetch shares
the tab's cookie session, exactly like the site's own calls. This needs the
extension running and a reservecalifornia.com tab (the extension opens one if
needed).

rdapi answers **HTTP 200 even on errors**. The envelope is
`{"Response":1,"Result":…}` on success, and Response 2/3 with `Message` /
`ExceptionString` otherwise; the `reservations` steps use `fail_when` to turn
those into errors.

## Endpoints

| Command | Method | Path | Host |
| ------- | ------ | ---- | ---- |
| `parks` | GET | `/fd/citypark/namecontains/{name}` | rdr |
| `park` | POST | `/search/place` | rdr |
| `availability` | POST | `/search/grid` (≈21 nights max per call) | rdr |
| `unit` | GET | `/search/details/{unit}/startdate/{date}/nights/{n}/0/{classification}` | rdr |
| *(hidden)* `facility` | GET | `/fd/facilities/{facility}` | rdr |
| `me` | POST | `/WebAccessCustomer/HouseholdInfo` | rdapi |
| `reservations` | POST | `HouseholdInfo` → `/WebAccessCustomer/Load/MyReservations` | rdapi |

## Drift symptoms & recapture

- **`HTTP 401` / `RecaptureRequiredError`**: the 60-minute token expired. Run
  `mastro login reservecalifornia`.
- **Search returns an error or an unexpected shape**: the search API moved or
  changed. Re-read `https://www.reservecalifornia.com/config.json` (`rdrApiUrl`)
  and diff a search request from the site's Network tab against `openapi.yaml`.
- **Base URLs moved**: re-read `https://www.reservecalifornia.com/config.json`
  and update `servers`.
