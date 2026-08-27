---
name: elisa-track-project-events
description: Find and monitor ELISA Project events — workshops, seminars, conference appearances — from ELISA's own live calendar API, with the date filters and paging behaviour that surface actually has.
api: ELISA Events API
base_url: https://elisa.tech/wp-json/tribe/events/v1/
spec: openapi/elisa-events-calendar-openapi.json
operations:
  - GET /events
  - GET /events/{id}
  - GET /events/by-slug/{slug}
  - GET /categories
  - GET /tags
auth: none
---

# Track ELISA Project events

ELISA publishes its event calendar as a live, anonymous, machine-readable API. No key, no signup.

## Base

```
https://elisa.tech/wp-json/tribe/events/v1/
```

Discovered from ELISA's own RFC 9727 catalog at `https://elisa.tech/.well-known/api-catalog`,
which anchors `https://elisa.tech/wp-json/`. The OpenAPI for this namespace is served at
`GET /doc` on the same base.

## 1. Find upcoming events

```
GET /events?starts_after=2026-08-27&per_page=20
```

Date filters accept `start_date`, `end_date`, `starts_before`, `starts_after`, `ends_before`,
`ends_after`, and `strict_dates` to control whether the window is inclusive.

## 2. Handle the empty-result trap first

**An archive with no matches returns `404`, not an empty `200`.** This is the single behaviour
most likely to break a naive client. Before writing any retry or alerting logic:

- `404` on `/events` means *no events matched your filters*. It is a normal, expected result.
- Do not retry it, do not escalate it, do not treat it as an outage.
- Only `500` is retryable on this surface (see `errors/elisa-problem-types.yml`).

## 3. Page correctly

The response carries absolute cursors — use them rather than incrementing `page` yourself:

```
next_rest_url, previous_rest_url, total, total_pages
```

## 4. Narrow by topic

```
GET /categories
GET /events?categories=<slug-or-id>&starts_after=2026-08-27
```

Both taxonomies (`categories`, `tags`) accept slugs or numeric IDs.

## 5. Resolve one event

By numeric WordPress post ID or by URL slug — both are first-class:

```
GET /events/{id}
GET /events/by-slug/{slug}
```

Venue and Organizer objects are **embedded in full** in the Event payload. There is no expand
or sparse-fieldset parameter, so do not issue follow-up calls for them.

## Errors

The envelope is the WordPress REST shape, not RFC 9457:

```json
{ "code": "rest_no_route", "message": "...", "data": { "status": 404 } }
```

Branch on `code`, not on the message string.

## Do not write

This skill is read-only on purpose. The same namespace exposes `POST` and `DELETE` on events,
venues, organizers, categories and tags, but ELISA publishes **no reversal operation and no
retention window** for a delete — a repeat `DELETE` returns `410 Gone` and there is no restore
path in the contract. Treat every write on this API as irreversible and keep it out of
autonomous flows.
