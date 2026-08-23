---
name: impossible-metals-harvest-media
description: >-
  Enumerate and retrieve Impossible Metals' published visual and document assets — Eureka vehicle
  photography, seabed and nodule imagery, technical diagrams and report PDFs — from the 727-item media
  library, with the licensing caveat that matters before anyone reuses them.
api: impossible-metals:impossible-metals-media-api
operations:
  - listMedia
  - getMediaItem
---

# Harvest Impossible Metals media assets

727 attachments at capture, each with a direct `source_url`, a MIME type, alt text and generated size
variants. This is the fastest way to find, say, every PDF the company has posted, or the highest-resolution
render of the Eureka collection system.

Base URL: `https://impossiblemetals.com/wp-json`. Anonymous, read-only, no key.

## Steps

1. **Page the library** with `listMedia`:

       GET /wp/v2/media?per_page=100&page=1&_fields=id,date,slug,title,alt_text,mime_type,media_type,source_url,post

   Read `X-WP-Total` (727 at capture) and `X-WP-TotalPages` from the first response and loop `page=` to the end.

2. **Filter by type** rather than fetching everything:

       GET /wp/v2/media?media_type=image&per_page=100
       GET /wp/v2/media?mime_type=application/pdf&per_page=100

   PDFs are usually the reports and technical documents; images are the vehicle and seabed assets.

3. **Pick the right size.** `getMediaItem` returns `media_details.sizes` with a distinct `source_url` per
   registered size (thumbnail, medium, large, full). Take the smallest that meets your need — `full` on a
   press photo is large and there is no CDN transform in front of it.

       GET /wp/v2/media/<id>

4. **Find assets by subject** by searching the media collection itself:

       GET /wp/v2/media?search=eureka&per_page=100&_fields=id,title,alt_text,mime_type,source_url,post

   Do NOT reach for `/wp/v2/search` here. Its `subtype` enum on this site is
   `post, page, tribe_venue, tribe_organizer, tribe_events, avada_faq, category, post_tag, faq_category`
   and friends — `attachment` is not among them, so the global search index does not cover media at all.

5. **Trace an asset back to its context** through `post`, which names the post or page the attachment was
   uploaded to. Cite that page, not the bare file URL.

## Rules

- **Licensing is not granted by accessibility.** These files are public and unauthenticated; that is not a
  licence. Impossible Metals publishes no media-use terms, and the site has no terms-of-service page — a
  `/terms-of-use/` probe returned 404 on 2026-08-23. Treat every asset as all-rights-reserved and get
  written permission before republishing. Do not let an agent redistribute these on a user's behalf.
- `alt_text` is frequently empty. Do not present it as a caption when it is blank.
- Download `source_url` directly; it is a static file on the same origin, not a REST route, and it returns
  no REST error envelope — check the HTTP status, not the body.
- 727 items at 100 per page is 8 requests. Do them serially with a real User-Agent. No rate-limit headers
  are published, so there is no signal to tell you when you have gone too fast.
- Store the `id` and `modified` of what you fetched so a later pass can use `modified_after` instead of
  re-walking the library.
