---
name: impossible-metals-research-position
description: >-
  Answer a question about Impossible Metals' stated position on deep-sea mining — environmental impact,
  regulation, techno-economics, competitors or the Eureka collection system — by reading the company's own
  141-entry FAQ knowledge base and its blog archive, rather than by summarising press coverage.
api: impossible-metals:impossible-metals-faq-api
operations:
  - listFaqCategories
  - listFaqs
  - getFaq
  - search
  - listPosts
---

# Research Impossible Metals' stated position

Use this when someone asks what Impossible Metals *says* about a contested topic — seabed ecosystem damage,
sediment plumes, ISA regulation, cost versus land-based mining, how Eureka differs from dredge-and-riser
collectors. The company maintains a structured FAQ of 141 entries in 11 categories; that is a primary source
and it is machine-readable. Press coverage is not.

Base URL: `https://impossiblemetals.com/wp-json`. Anonymous, read-only, no key. Send a real User-Agent —
Cloudflare answers 403 to some requests without one.

## Steps

1. **Pick the right category first.** Call `listFaqCategories`:

       GET /wp/v2/faq_category?per_page=20&_fields=id,name,slug,count

   The categories observed on 2026-08-23, with entry counts:
   `impossible-metals-technology` (32), `environmental-and-social-responsibility-for-deep-sea-mining` (19),
   `deep-sea-mining-regulations` (16), `myth-busting-false-and-misleading-statements-about-deep-sea-mining-dsm` (16),
   `market-for-deep-sea-critical-minerals` (12), `impossible-metals-company` (12),
   `i-techno-economic-analysis-tea` (9), `impossible-metals-competitors` (8), `transport-vessels` (7),
   `home-page` (5), `h-mineral-processing-of-polymetallic-nodules` (4).

   Counts drift. Read them from the response; do not hard-code these.

2. **List the entries in that category** with `listFaqs`:

       GET /wp/v2/avada_faq?faq_category=<id>&per_page=100&_fields=id,slug,title,link,content

   Ask for `content` only when you intend to quote. `per_page` is capped at 100.

3. **Or search across everything** with `search` when the question does not map to a category:

       GET /wp/v2/search?search=<terms>&per_page=10

   This spans posts, pages, FAQ entries and events. Read `subtype` on each hit to see what you found;
   `avada_faq` is an FAQ entry, `post` is a blog post or press release.

4. **Corroborate with the blog** using `listPosts` when the question is about something that happened
   (a lease application, a partnership, testimony) rather than a position:

       GET /wp/v2/posts?search=<terms>&per_page=10&_fields=id,date,title,link,excerpt

   Category `16` is `press-release`, `17` is `news`, `18` is `blog`.

5. **Quote with attribution.** Every record carries a `link` to the human page. Cite that URL, and cite the
   `date` on posts — some of this archive is years old, and deep-sea mining regulation moves.

## Rules

- `title` and `content` come back as `{"rendered": "<html>"}`. Strip the HTML before quoting; do not hand
  raw rendered HTML to a user.
- Trim every request with `_fields`. The default response includes full rendered content on every item and
  is large enough to matter across 141 entries.
- On `{"code":"rest_invalid_param"}` read `data.params` — almost always `per_page` above 100.
- On `{"code":"rest_post_invalid_id"}` the id does not exist or is not published. List first, then fetch.
- **This is the company speaking about itself.** It is a primary source for Impossible Metals' position and
  it is not a neutral source on deep-sea mining. Attribute it as the company's own claim, and do not present
  a `myth-busting` entry as an independent finding.
- No rate-limit headers are returned. Page deliberately, honour `cache-control`, back off on 403 or 5xx.
