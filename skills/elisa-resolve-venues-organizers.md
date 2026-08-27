---
name: elisa-resolve-venues-organizers
description: Resolve the venues and organizing bodies behind ELISA Project events from ELISA's live API — who is hosting, where, and which events tie back to them.
api: ELISA Events API
base_url: https://elisa.tech/wp-json/tribe/events/v1/
spec: openapi/elisa-events-calendar-openapi.json
operations:
  - GET /venues
  - GET /venues/{id}
  - GET /venues/by-slug/{slug}
  - GET /organizers
  - GET /organizers/{id}
  - GET /organizers/by-slug/{slug}
  - GET /events
auth: none
---

# Resolve ELISA venues and organizers

`Venue` and `Organizer` are addressable resources in their own right, not just fields on an
event. That is what makes the reverse lookup — *which events is Linaro organizing?* — a single
call rather than a scan.

## 1. Enumerate

```
GET /organizers
GET /venues
```

Both are paged the same way as `/events` (`page`, `per_page`, and absolute `next_rest_url` /
`previous_rest_url` cursors), and both return `404` rather than an empty `200` when nothing
matches.

## 2. Resolve one, by ID or slug

```
GET /organizers/{id}
GET /organizers/by-slug/{slug}
GET /venues/{id}
GET /venues/by-slug/{slug}
```

## 3. Reverse the relationship

Filter the event archive by the ID you just resolved:

```
GET /events?organizer=<id>
GET /events?venue=<id>
```

Note the asymmetry: the filters take **IDs**, while the resource lookups take an ID *or* a slug.
Resolve slug → ID first, then filter.

## Geolocation is not available here

The spec declares `geoloc`, `geoloc_lat` and `geoloc_lng` filters on `/events`, but each is
documented as "Requires Events Calendar Pro". That add-on is not active on this deployment —
these parameters are inert. Do not build a proximity search on them.

## Identifiers

IDs are plain integers (WordPress post IDs) with no type prefix. An ID is only meaningful
together with its resource path — `42` as a venue and `42` as an organizer are unrelated
objects. Always carry the resource type alongside the ID.

See `data-model/elisa-data-model.yml` for the full entity graph.
