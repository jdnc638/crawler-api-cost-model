# Website crawler API: what a full crawl costs per page, which plan fits, and the settings that decide whether you get 10 pages or 10,000

A crawler API bill comes down to three numbers: what counts as one page, which request options multiply that page's price, and whether blocked or failed pages still get charged. On ScraperAPI a standard page costs 1 credit, and the same page with JavaScript rendering costs 10. A 100,000-credit plan therefore buys 100,000 pages or 10,000 pages depending on what you point it at.

That ratio matters more than the plan name on the invoice. Below are ScraperAPI's current tiers with credits and concurrency, the per-request credit table, the scope and compliance settings that change what you actually receive, and verified facts from three other crawl APIs so you can compare on more than a headline page count.

> **Quick pick:** most teams crawling a docs site, marketing site or product catalogue should start on **Hobby (100,000 credits, 20 concurrent threads)** and cap the first crawl with a small budget before scaling. If your pages need JS rendering, budget 10 credits each, so Hobby covers roughly 10,000 rendered pages per month. 👉 [See what each ScraperAPI plan includes](https://www.scraperapi.com/pricing?fp_ref=coupons)

One caveat before the table. If you already have a URL list, the crawler is the wrong tool. ScraperAPI's own product guidance points known-URL workloads at the Async API, which is cheaper and more predictable per page, and reserves the Crawler for the case where you don't know the URLs upfront.

## ScraperAPI plans: credits, concurrency and billing rules

ScraperAPI quotes the dollar figure for each tier on its pricing page and takes 10% off for annual billing. The figures below are the ones that decide whether a plan fits: monthly credits, concurrent threads, and the billing switches attached to each tier.

| Plan | API credits / month | Concurrent threads | What changes at this tier | Pricing |
| --- | --- | --- | --- | --- |
| Hobby | 100,000 | 20 | US & EU geotargeting, 30-day analytics, auto-upgrade opt-in | [Check Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | Same geotargeting, 30-day analytics | [Check Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Country-level geotargeting, unlimited analytics | [Check Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | Pay-as-you-go becomes available | [Check Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Pay-as-you-go, 300 threads | [Check Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Pay-as-you-go, auto-upgrade no longer offered | [Check Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Custom credit volume, custom annual discount | [Talk to ScraperAPI about Enterprise](https://www.scraperapi.com/?fp_ref=coupons) |

Secondary rules that affect the monthly total:

- **No credit rollover on any tier.** Unused credits expire at renewal.
- **10% annual discount** on every self-serve plan, custom on Enterprise.
- **Pay-as-you-go** exists only from Scaling upward, at a fixed per-credit rate up to a monthly cap you set.
- **Auto-upgrade** is opt-in on Hobby through Professional and unavailable on Advanced and Enterprise.
- **Free trial allowance:** 1,000 API credits with a maximum of 5 concurrent connections.
- **Going over concurrency returns a 429** and the request is turned away rather than queued. There is no overage charge for the rejection itself.

## What one credit buys, and why your page count is smaller than the plan name

This is the part that turns a quote into an invoice. ScraperAPI prices requests by what the target needs, not by URL count.

| Request type | Credits per request |
| --- | --- |
| Standard page | 1 |
| E-commerce (Amazon, Walmart, eBay) | 5 |
| SERP (Google, Bing, including subdomains) | 25 |
| Social (LinkedIn) | 30 |
| Bot-protection bypass (Cloudflare, DataDome, PerimeterX/HUMAN, Akamai) | 10 |
| JS rendering (`render=true`) | 10 |
| Premium proxies (`premium=true`) | 10 |
| Screenshot (`screenshot=true`) | 10 |
| Premium proxies + JS rendering | 25 |
| Ultra Premium (`ultra_premium=true`) | 30 |
| Ultra Premium + JS rendering | 75 |

Geotargeting, sessions, device type, output format and auto parsing add nothing. Rendering and proxies do.

Run the numbers on Hobby's 100,000 credits. A static blog or docs site returns 100,000 pages. A React storefront where every page needs rendering returns 10,000. Premium proxy plus rendering returns 4,000. Ultra Premium with rendering returns about 1,333 pages, which is fewer pages than many sites have. Business at 3,000,000 credits gives you 300,000 rendered pages or 40,000 at the Ultra Premium rate.

Two controls keep this predictable. Every response carries its cost in the `sa-credit-cost` header, so you can log real spend per page instead of estimating. And `max_cost` caps what any single scrape is allowed to consume, which prevents one badly chosen URL from eating a day of budget. The dashboard's Domain Cost Estimator prices a URL before you send it.

## Four line items that make a quote and an invoice disagree

Per-page pricing is where crawl APIs differ most, and the differences sit in documentation rather than the pricing page.

**Structured output is a paid format on Firecrawl.** A crawled page costs 1 credit and is billed when the page finishes. Asking for JSON adds 4 credits per page on top, so a JSON-first RAG pipeline pays five times the headline rate. Markdown stays at 1 credit.

**Cloudflare splits billing by render mode.** With `render: true` the crawl runs through a headless browser and bills at Browser Run rates. With `render: false` it does a fast HTML fetch through Workers, and during the beta those crawls are not billed at all; after the beta they move to Workers pricing. The JSON format routes through Workers AI by default, which is a separate meter.

**crawlcrawl charges the quota whether or not it fetches.** In its bulk scan endpoint each URL consumes one page of quota even on a cache hit, so `max_age_seconds` saves latency rather than money.

**ScraperAPI charges for the capability, not the page.** The multiplier table above is the whole story. A crawl of 5,000 pages behind Cloudflare with rendering on is 50,000 credits, or a quarter of Hobby, before a single retry.

👉 [Check which ScraperAPI tier covers your monthly page count](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Scope settings that decide between 10 pages and 10,000

Crawl scope is where most disappointing first runs come from. Four defaults are worth knowing before you launch anything.

**ScraperAPI's Crawler gives you two independent brakes.** `max_depth` limits link hops from the seed URL and `crawl_budget` caps credit spend; run both and the job stops at whichever hits first. Free accounts are capped at `max_depth: 1`. The include pattern is a regex, and a pattern of `.*` will follow external links and crawl as far as the queue allows, so start narrow and validate on a small run. Job status is polled, and webhooks plus scheduling are available for recurring crawls.

**Firecrawl's `includePaths` also tests your starting URL.** If the seed doesn't match the pattern, the crawl can return 0 pages. The same API defaults to child paths only, so `/pricing` or `/` is skipped unless `crawlEntireDomain` is true, and `limit` defaults to 10,000 pages. External links pointing at another site's homepage are skipped and reported as `EXTERNAL_LINK` in the errors endpoint, and redirects to an external homepage get the same treatment.

**Cloudflare's two defaults point in opposite directions.** `limit` defaults to **10** while `depth` defaults to 100,000. Launch a crawl without touching `limit` and you get ten pages from a site with forty thousand. `includePatterns` uses wildcards rather than regex: `*` matches anything except `/`, `**` matches across slashes.

**crawlcrawl's `store_html: false` is a one-way door.** It collects URLs, status codes, byte counts and metadata only, which is a cheap way to map a site, but pages without stored HTML cannot later be converted to markdown. Its `depth` is capped at 50 with server-side validation, `max_pages` defaults to 1,000, and a sixth simultaneous run returns 429. Sitemap-first crawling via `seed_kind: "sitemap"` is materially faster than link-following on large sites.

Blocked and failed pages deserve their own line. crawlcrawl returns a target's 4xx and 5xx responses as pages rather than errors, so you filter by status yourself. Cloudflare exposes a granular outcome list including `robots_disallowed`, `crawl_purpose_disallowed`, `too_large` and `browser_unavailable`. Firecrawl reports failed and robots-blocked URLs through its errors endpoint.

## robots.txt and content signals are now a hard gate

If you are building training or RAG data, the declaration you send can kill the request before any page loads.

Cloudflare's crawl endpoint requires you to declare `crawlPurposes` (search, ai-input, ai-train; all three selected by default) and `contentUse` (reference or full, default full). If a target's robots.txt sets a Content-Signal directive marking any declared purpose as `no`, or a `use` level stricter than yours, the request is rejected with a 400. Declaring narrower purposes is not a formality; it is what keeps you inside the site's stated terms, and it is also what can make a legitimate crawl fail if your declaration is broader than the site allows.

Firecrawl follows robots.txt by default and bills pages as they finish. Overriding robots (`ignoreRobotsTxt`) and setting a custom robots user agent are Enterprise-only features that require contacting support, so a self-serve plan cannot turn them on. crawlcrawl sets `respect_robots: true` by default and lets you switch it off per crawl.

## How the alternatives compare on verified terms

|  | Pricing model | Default or rule that catches people | Reasonable fit |
| --- | --- | --- | --- |
| ScraperAPI | Credits per request; standard page 1, JS rendering 10, bot bypass 10, Ultra Premium + rendering 75; 10% annual discount; no rollover | Concurrency overage returns 429 instead of queueing | Mixed sites where some pages need rendering and proxies and others don't |
| Firecrawl | 1 credit per crawled page; JSON adds 4 credits per page; Hobby $16/mo, Standard $83/mo, Growth $333/mo, Scale $599/mo, all billed yearly; Free at $0 | `includePaths` also validates the starting URL; external homepage links skipped as `EXTERNAL_LINK` | Markdown-first pipelines that want crawl, change monitoring and webhooks from one vendor |
| Cloudflare Browser Run `/crawl` | Browser Run usage when `render: true`; `render: false` crawls unbilled during beta, Workers pricing after | `limit` defaults to 10 while `depth` defaults to 100,000; Content-Signal mismatch returns 400 | Static or lightly rendered sites already living inside Cloudflare |
| crawlcrawl | Page quota per run, `max_pages` default 1,000, hard cap 100,000; 5 concurrent runs | `store_html: false` cannot be undone later; cache hits still consume quota | Sitemap-first mapping and cheap URL discovery |

Firecrawl's concurrency runs 2 on Free, 5 on Hobby, 25 on Standard, 50 on Growth and 100 on Scale, with extra credits sold per tier. Cloudflare returns a job ID and reports results asynchronously, with `browserSecondsUsed` in the result payload. crawlcrawl retries webhooks five times with exponential backoff, but a 4xx response is marked dead immediately with no retry, which is worth knowing if your receiver validates payloads strictly.

## Pick by workload, then confirm two things before you pay

The decision rule is short.

- **Static docs, blogs, catalogues with no rendering:** the cheapest credit tier that covers your page count, which usually means Hobby. At 1 credit per page, 100,000 credits is a lot of pages.
- **JavaScript-heavy sites or bot-protected targets:** count 10 credits per page minimum and size the plan from that number, not from raw URL count.
- **You already have the URL list:** skip the crawler entirely and use the Async API.
- **You need recurring change detection:** ScraperAPI's Crawler supports scheduling and webhooks; Firecrawl bundles a Monitor that labels pages new, changed, removed or same.
- **You need to respect narrow AI-training signals:** Cloudflare's declaration model makes that explicit.

Before you enter card details, confirm the billing cycle you are actually buying and the concurrency your job needs. Annual billing takes 10% off and locks the term; monthly costs more but keeps you flexible while you measure real credit burn. On concurrency, a crawl that opens more connections than your tier allows gets 429s rather than a queue, so a 20-thread Hobby plan will stall a job tuned for 100.

Run one capped crawl first, read the `sa-credit-cost` header on the responses, and multiply by your real page count. That number, not the plan name, tells you which tier to buy.

👉 [Compare ScraperAPI plans and pick your credit tier](https://www.scraperapi.com/pricing?fp_ref=coupons)
