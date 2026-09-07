---
name: esri-arcgis-geocode-an-address
description: Turn a street address or place name into coordinates with the ArcGIS Geocoding service, and turn coordinates back into an address.
generated: '2026-09-07'
method: generated
source: openapi/esri-arcgis-geocoding-api-openapi.yml, conventions/esri-arcgis-conventions.yml, errors/esri-arcgis-problem-types.yml, plans/esri-arcgis-plans-pricing.yml
api: ESRI ArcGIS Geocoding API
operations:
  - findAddressCandidates
  - reverseGeocode
mcp_tools:
  - find_address_candidates
  - reverse_geocode
---

# Geocode an address with ArcGIS

Two ways to run this flow: call the REST operations directly, or call the equivalent tools on the
Esri-hosted MCP server at `https://location-services-mcp.arcgis.com/beta/mcp`
(`find_address_candidates`, `reverse_geocode`). The MCP path needs the same credential.

## Before you start

- Get an ArcGIS Location Platform API key or an OAuth 2.0 access token. See
  `authentication/esri-arcgis-authentication.yml`.
- Send it as `?token=<token>` or `Authorization: Bearer <token>`.
- **Always send `f=json`.** Many ArcGIS REST resources default to `f=html` and will hand you an HTML
  page with HTTP 200.

## Steps

1. **Forward geocode** — `findAddressCandidates`
   `GET https://geocode-api.arcgis.com/arcgis/rest/services/World/GeocodeServer/findAddressCandidates`
   with `singleLine=<address>`, `outFields=*`, `f=json`, `token=<token>`.
   Read `candidates[]`; each carries `address`, `location` (a `Point` with `x`, `y`) and `score`.
   Take the highest `score`, and check `attributes.Addr_type` to know how precise the match is
   (`PointAddress` is rooftop; `PointAddressInt` means the point was interpolated; `StreetName` is
   coarser than the caller probably wants).

2. **Reverse geocode** — `reverseGeocode`
   `GET .../reverseGeocode` with `location=<x>,<y>`, `f=json`, `token=<token>`.
   The response is a `ReverseGeocodeResponse` with `address` and `location`.

3. **Check the spatial reference before you compare anything.** Every `Point` carries a
   `spatialReference.wkid`. Coordinates from two different `wkid`s are not comparable. WGS84 is
   `wkid: 4326`; ArcGIS basemaps default to Web Mercator `wkid: 102100 / 3857`.

## Rules that will bite you

- **`forStorage=true` changes the price, not the answer.** Not-stored geocodes are $0.50/1,000 after a
  20,000/month free allowance; stored geocodes are $4/1,000 with no free tier. Only set
  `forStorage=true` when you are actually persisting the result — Esri's terms require it when you do.
  See `plans/esri-arcgis-plans-pricing.yml`.
- **Read the body, not the status line.** A response can return HTTP 200 carrying
  `{"error":{"code":498,"message":"Invalid token"}}`. Branch on `error.code`, not on the HTTP status.
  Full list in `errors/esri-arcgis-problem-types.yml`.
- **There is no idempotency key.** A retried geocode is a second billable transaction. Deduplicate
  before you call, not after.
- **There are no rate-limit headers.** Nothing in the response tells you how much of your monthly
  allowance is left; check the ArcGIS Location Platform usage dashboard.
- **HTTPS only.** The geocoding service dropped plain HTTP.
