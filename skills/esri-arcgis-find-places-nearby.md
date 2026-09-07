---
name: esri-arcgis-find-places-nearby
description: Search points of interest near a coordinate with the ArcGIS Places service and fetch the attributes of one place.
generated: '2026-09-07'
method: generated
source: openapi/esri-arcgis-places-api-openapi.yml, conventions/esri-arcgis-conventions.yml, plans/esri-arcgis-plans-pricing.yml
api: ESRI ArcGIS Places API
operations:
  - findPlacesNearPoint
  - getPlaceDetails
---

# Find places near a point

## Before you start

- ArcGIS API key or OAuth access token, sent as `?token=` or `Authorization: Bearer`.
- Base: `https://places-api.arcgis.com/arcgis/rest/services/places-service/v1`.
  (`.../arcgis/rest/services` on its own returns `404 Path not found` — the version segment is required.)

## Steps

1. **Search** — `findPlacesNearPoint`
   `GET /places/near-point` with `x`, `y`, `radius` (metres), optional `categoryIds`, `searchText`,
   `f=json`, `token=<token>`.
   The response is a `PlacesResponse` with `results[]` of `PlaceSummary` — each has `placeId`, `name`,
   `categories[]`, `location` and `distance`.

2. **Fetch one place** — `getPlaceDetails`
   `GET /places/{placeId}` with `requestedFields=<comma list>`, `f=json`, `token=<token>`.
   Use the `placeId` from step 1.

## Rules that will bite you

- **`requestedFields` is a billing decision.** The Places service prices each attribute family
  separately — place attributes $0.05/1,000, address $0.10/1,000, details $0.13/1,000, location
  $0.35/1,000, each with a 100/month free allowance. Ask only for the families you will use.
- **The search itself is priced separately** from the detail fetch: near-point / within-extent requests
  are $8/1,000 after 500 free per month. Batching one search plus many detail fetches is cheaper than
  many searches.
- **Categories are a controlled vocabulary.** Filter with `categoryIds` from the Places category list
  rather than free text where you can — free text search still costs a search request.
- Error handling and idempotency: identical to every other ArcGIS surface. See
  `errors/esri-arcgis-problem-types.yml` and `conventions/esri-arcgis-conventions.yml`.
