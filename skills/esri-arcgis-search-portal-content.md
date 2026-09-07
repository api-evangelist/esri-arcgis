---
name: esri-arcgis-search-portal-content
description: Search an ArcGIS Online organization for items, read an item's metadata, list a user's content, and read the organization's own configuration.
generated: '2026-09-07'
method: generated
source: openapi/esri-arcgis-portal-api-openapi.yml, conventions/esri-arcgis-conventions.yml, data-model/esri-arcgis-data-model.yml
api: ESRI ArcGIS Portal API
operations:
  - getPortalSelf
  - searchPortal
  - getItem
  - getUserItems
  - getUser
---

# Search and read ArcGIS portal content

Base: `https://www.arcgis.com/sharing/rest` (ArcGIS Online) or `https://<org>/<context>/sharing/rest`
(ArcGIS Enterprise).

## Steps

1. **Identify the organization** — `getPortalSelf`
   `GET /portals/self?f=json&token=<token>` returns the `Portal` the token belongs to, including its
   `id`, `urlKey`, `customBaseUrl` and enabled capabilities. Do this first: item URLs and privileges are
   organization-scoped.

2. **Search** — `searchPortal`
   `GET /search` with `q=<query>`, `start`, `num`, `sortField`, `f=json`.
   Paging is offset-based: read `total`, `start`, `num` and `nextStart` from the response and keep going
   until `nextStart` is `-1`.
   Useful `q` filters: `owner:<username>`, `type:"Feature Service"`, `tags:<tag>`, `orgid:<portalId>`.

3. **Read an item** — `getItem`
   `GET /content/items/{itemId}?f=json` returns the `Item`: `title`, `type`, `owner`, `url`, `tags`,
   `typeKeywords`, `access`.
   `item.type` is the only reliable way to know what an item is — item ids are opaque 32-character hex
   with no type prefix.

4. **List a user's content** — `getUserItems`
   `GET /content/users/{username}/items?f=json` (folder-scoped variants exist).

5. **Read a user** — `getUser`
   `GET /community/users/{username}?f=json` returns the `User` and their `groups[]`.

## Rules that will bite you

- **`f=json` is not the default.** `/search` and most sharing resources default to `f=html`.
- **Privileges, not scopes.** ArcGIS credentials carry privileges chosen at credential-creation time.
  A 200 response containing `{"error":{"code":403}}` means the token authenticated but lacks a privilege
  — see `scopes/esri-arcgis-scopes.yml`.
- **Deletes are reversible for at least 14 days, if the recycle bin is on.** Deleted items go to the
  organization Recycle bin and can be restored via `POST /content/users/{username}/items/{itemId}/restore`
  for at least 14 days (336 hours). But the recycle bin is on by default only for organizations created
  after the June 2024 ArcGIS Online update, and restored items lose their metadata and all group
  sharing. Confirm the setting before you rely on it. See `conventions/esri-arcgis-conventions.yml`.
- **No idempotency key exists.** A retried `addItem` creates a second item.
- **To react to changes, register a webhook** rather than polling `/search`: see
  `asyncapi/esri-arcgis-webhooks.yml` for the 32 trigger events (`/items/add`, `/items/delete`,
  `/users/signin`, …).
