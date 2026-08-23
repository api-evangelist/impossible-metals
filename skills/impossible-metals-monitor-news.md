---
name: impossible-metals-monitor-news
description: >-
  Track new Impossible Metals announcements — lease applications, MOUs and partnerships, Eureka test
  results, congressional testimony — by polling the company's own post archive with a date window, instead
  of scraping the site or relying on a news aggregator.
api: impossible-metals:impossible-metals-posts-api
operations:
  - listPosts
  - getPost
  - listCategories
  - listMedia
---

# Monitor Impossible Metals announcements

247 posts at capture, covering press releases, news, blog and podcast/webinar entries. The collection
supports server-side date filtering, so incremental polling is cheap and does not require fetching the archive.

Base URL: `https://impossiblemetals.com/wp-json`. Anonymous, read-only, no key.

## Steps

1. **Establish the vocabulary once** with `listCategories`:

       GET /wp/v2/categories?per_page=20&_fields=id,name,slug,count

   Observed 2026-08-23: `16 press-release` (23), `17 news` (52), `18 blog` (86), `25 podcasts-webinars` (61),
   `30 forbes-articles` (14), `27 in-the-press` (17), `24 sustainable-development-goals` (8).
   Several categories are author names (`32 oliver-gunasekara`, `33 jason-gilliam`) — they are bylines, not topics.

2. **Poll with a date window** using `listPosts`. Keep the timestamp of your last successful poll and pass it:

       GET /wp/v2/posts?after=<ISO8601>&orderby=date&order=desc&per_page=100&_fields=id,date,modified,slug,link,title,excerpt,categories

   Use `after` for newly published items and `modified_after` for edits to items you already have. `date` is
   site-local (America/Los_Angeles); `date_gmt` is the one to compare against a UTC watermark.

3. **Narrow to announcements** by adding `&categories=16,17` when you only want press releases and news and
   not the podcast archive.

4. **Read the item** with `getPost` when a title matches:

       GET /wp/v2/posts/<id>

   Or add `&_embed` to step 2 to inline the author, featured image and terms without a second round trip.

5. **Pull the asset** with `listMedia` when a release has an image or PDF worth keeping:

       GET /wp/v2/media?parent=<post_id>&_fields=id,source_url,mime_type,alt_text

## Rules

- Read `X-WP-Total` and `X-WP-TotalPages` on the first response and page with `page=` until exhausted.
  Never assume one page holds everything.
- Advance your watermark only after a page is fully processed, using `date_gmt` from the newest item you
  actually handled — not the current clock. A failed page must be re-fetched, not skipped.
- Responses are edge-cached (`cache-control: max-age=172800` observed). Polling more than a few times a day
  buys nothing; you will be served the same cached body.
- No rate-limit headers exist on this surface. There is no budget to read, so behave conservatively: a real
  User-Agent, exponential backoff on 403 and 5xx, and no parallel fan-out.
- This is a corporate news archive, not an API changelog. It says nothing about changes to the surface itself,
  which are undocumented and unannounced.
