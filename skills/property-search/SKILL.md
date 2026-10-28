---
name: property-search
description: Find homes, apartments, condos, or rentals to buy or rent. Searches the web broadly, reads public listing pages, and returns a deduplicated table of matching properties with links to the original listings. Use when the user wants to find a place to rent or buy, e.g. "find me 2-bed rentals in 78704 under $2,800" or "3-bed houses for sale in Austin under $600k".
---

# Property Search

Search the web for properties that match the user's criteria, read public listing pages, and return one deduplicated table that links back to every original listing.

Run only when the user asks, once per request. Results are for the user's own search.

## Hard rules

These apply to every search and override everything else in this file.

1. **Fetch budget.** At most ~30 page fetches per search. robots.txt checks don't count toward this, but do at most one per domain. No loops, polling, background jobs, or scheduled re-runs. If the user wants fresh results later, they ask again.
2. **Blocked means skip.** If a site returns 403 or 429, shows a CAPTCHA or bot check, shows an "access denied" page, or asks you to log in, stop fetching that site for the rest of the search and give the user a direct search link to open themselves (see [Direct search links](#direct-search-links)). Never get around a block: no proxies, no rotating or spoofing user agents, no retry tricks, no logging in, and no cached or mirror copies used to reach a blocked page. Send your first fetch to any domain on its own, and only fetch from it in parallel after that succeeds, so a site that blocks gets one request, not several.
3. **Respect robots.txt.** Before the first page fetch on a domain, read `https://<domain>/robots.txt`. Read the raw rules: if your fetch tool summarizes pages, ask it to quote the `User-agent`, `Allow`, and `Disallow` lines verbatim. If the path you want is disallowed for `User-agent: *` or for a Claude/Anthropic agent (`Claude-User`, `ClaudeBot`, `anthropic-ai`), don't fetch it; use the search result snippet and link instead. A 404 means the site has no rules. If robots.txt itself is blocked, treat the whole site as blocked (rule 2). If a page redirects to a different host, check that host's robots.txt before following the redirect.
4. **Link, don't copy.** Never save, download, or republish listing photos or full listing descriptions. Extract only the short facts in step 3 and link to the source.
5. **Fair housing.** Never filter, rank, or describe listings or neighborhoods by protected characteristics: race, color, religion, national origin, sex, disability, or familial status (Fair Housing Act), plus anything protected by state or local law, such as sexual orientation, gender identity, age, or source of income. If the user asks for that, say you can't and offer neutral criteria instead (price, size, commute, specific amenities). Don't add your own commentary about who lives in an area or who it "suits". Features the user wants for themselves, such as step-free entry, an elevator, or a 55+ community, are fine: they describe the property, not other people.
6. **Personal use only.** Results are for the user. Don't use this skill to build a database, dataset, feed, or another listing site.

## Step 1: Collect criteria

| Field | Required | Notes |
|---|---|---|
| Location | yes | City, ZIP, neighborhood, or "near X" |
| Rent or buy | yes | |
| Budget | yes | Max monthly rent or max price; a minimum is optional |
| Beds / baths | no | Treat "2-bed" as 2 or more unless the user says exactly 2 |
| Property type | no | Apartment, house, condo, townhome, ADU, room |
| Pets | no | Kind and size; this rules out many rentals |
| Move-in date | no | Mostly for rentals |
| Must-haves | no | Parking, in-unit laundry, yard, AC, and so on |

Read what the user already said and ask only for what's missing:

- If a required field is missing, ask for it in one short message, and mention the optional fields in that same message so the user can add them all at once.
- If every required field is present, don't stop to ask about optional ones. Search, then list your assumptions at the top of the results (for example "Assumed: any property type, 1+ baths, no pets") so the user can refine.

## Step 2: Search widely

First work out the area: the city, nearby ZIPs, and neighborhood names.

Then run 12–20 varied web searches, in parallel where you can. Mix all of these:

- **Major sites:** searches limited to zillow.com, redfin.com, realtor.com, apartments.com, zumper.com, hotpads.com, trulia.com, and craigslist.org, each with the location and key criteria. Use your search tool's domain filter (such as `allowed_domains`) if it has one, because some search tools silently ignore the `site:` operator. Otherwise use `site:` and check that the results really come from that site.
- **Local property managers:** `<city> property management rentals available`, `<city> homes for rent property manager`.
- **Apartment complexes:** `<city> apartments 2 bedroom floor plans pricing`. Complexes' own sites usually list live availability and prices.
- **Local classifieds and community boards:** Craigslist, local newspaper classifieds, and university housing boards if the area is near a campus.
- **Neighborhoods:** `<neighborhood> for rent` or `<neighborhood> for sale`, for each neighborhood in or next to the area.
- **Buying only:** also `<city> open house`, `<ZIP> new listings`, and local brokerage sites.

Note every result that looks like a listing: URL, title, and snippet. Snippets often carry the price, beds, and address, which saves fetches.

## Step 3: Read listing pages

Fetch only where fetching works normally, and spend the ~30-fetch budget carefully. Keep a running count.

Fetch in this order:

1. Property manager and apartment complex availability pages. They're current, rarely blocked, and often list several units per page.
2. Individual listing pages whose snippet looks like a match.
3. Search-results pages on aggregator sites, only where robots.txt allows them.

Don't fetch pages the snippet already rules out (over budget, too few beds, "sold", "pending", "off market", "no longer available").

Classifieds are full of reposts: one complex may post the same floor plan dozens of times under different titles. Open one or two posts to identify the complex, then treat its other posts at the same price as duplicates instead of fetching each one.

For each listing, extract only these fields:

`address` · `price` · `beds` · `baths` · `sqft` · `available date` · `pets` · `source` · `URL`

- Use exactly what the page says. If a field isn't shown, write `—`. Never guess.
- For a complex with several matching floor plans, use one row per floor plan, or one row with "from $X" if the site doesn't list units separately.
- Check the location against the actual address, not the title, URL, or the city page it came from. Aggregator "city" pages include neighboring cities, and classified posts titled with one city are often in another. Drop anything outside the requested area unless the user asked for nearby areas.
- Drop listings that clearly fail a hard criterion: over budget, fewer beds than asked, or no pets when the user has pets. Keep listings where a field is unknown.
- Drop listings that are stale: marked rented, or with an available date that has already passed. Mention how many you dropped.
- Note eligibility restrictions the listing states, such as income-restricted/BMR, a senior (55+ or 62+) community, student-only housing, or furnished corporate rentals. These are facts about the property; don't drop the listing for them.
- When a page can't be fetched (blocked, disallowed, or over budget), you may still include the listing from its search snippet if the snippet shows a price, a street address, and a URL. These go in the separate snippet table in step 5.

## Step 4: Dedupe

Treat listings as the same property when the normalized street address, unit, and 5-digit ZIP match.

To normalize an address: lowercase it; abbreviate street types (Street→St, Avenue→Ave, Road→Rd, Drive→Dr, Boulevard→Blvd, Court→Ct, Lane→Ln, Place→Pl); turn "Apartment", "Apt", "Unit", "Suite", and "#" into `#`; drop punctuation and ZIP+4 suffixes.

When merging duplicates:

- Keep every source link.
- Fill each field from whichever source has it.
- If prices differ, show the lowest and note the other, e.g. "$3,350 (Zumper lists $3,400)".

Listings with no street address (a complex name only, or a Craigslist post that says "near X"): merge by complex name, or by matching price, beds, sqft, and cross streets. When in doubt, keep them as separate rows.

## Step 5: Output

Start with one line covering the criteria, your assumptions, today's date, and how many pages you fetched.

Then a markdown table sorted by price, lowest first. The address links to the best source, and the Sources column links to every source:

```markdown
| # | Price | Address | Beds | Baths | Sqft | Available | Pets | Sources |
|---|---|---|---|---|---|---|---|---|
| 1 | $3,195/mo | [123 Example St #4, Austin, TX 78704](https://example.com/listing) | 2 | 2 | 1,050 | Now | Cats, dogs | [Apartments.com](https://example.com/a), [Zumper](https://example.com/b) |
```

If you have listings from search snippets only (step 3), put them in a second table with the same columns, headed "From search snippets (not opened, so unverified)", also sorted by price. Snippets are often weeks old.

After the tables:

- **Open these yourself:** direct search links for every site that was blocked, disallowed by robots.txt, or not reached within the budget. Fill in the user's criteria where the URL format allows.
- **A short caution:** listings change fast, so confirm price and availability on the source before applying. Never send a deposit or personal documents before seeing the place and verifying the landlord; rental scams are common on classifieds.

Then ask: "Want me to save these to `property-results-YYYY-MM-DD.md` in this folder?" Only write the file if the user says yes. Save it in the current working directory, with today's date, the criteria, the table, and the direct search links. Don't include photos or descriptions. If that filename exists, add `-2`, `-3`, and so on.

## Direct search links

Use these to give the user links to open themselves. URL formats change, so prefer a real search-page URL you saw in your search results, and fall back to these templates. `{city}-{st}` is lowercase with hyphens, e.g. `austin-tx`.

| Site | Rent | Buy |
|---|---|---|
| Zillow | `https://www.zillow.com/homes/for_rent/{ZIP}_rb/` | `https://www.zillow.com/homes/for_sale/{ZIP}_rb/` |
| Redfin | `https://www.redfin.com/zipcode/{ZIP}/rentals` | `https://www.redfin.com/zipcode/{ZIP}` |
| Realtor.com | `https://www.realtor.com/apartments/{ZIP}` | `https://www.realtor.com/realestateandhomes-search/{ZIP}` |
| Apartments.com | `https://www.apartments.com/{city}-{st}-{ZIP}/{beds}-bedrooms/` | — |
| Zumper | `https://www.zumper.com/apartments-for-rent/{city}-{st}` | — |
| HotPads | `https://hotpads.com/{city}-{st}/apartments-for-rent` | — |
| Trulia | `https://www.trulia.com/for_rent/{ZIP}_zip/` | `https://www.trulia.com/{ST}/{City}/{ZIP}/` |
| Craigslist | `https://www.craigslist.org/search/city/{city}-{st}?cat=apa&postal={ZIP}&radius=3&max_price={max}&min_bedrooms={beds}` | `https://www.craigslist.org/search/city/{city}-{st}?cat=rea&postal={ZIP}&radius=3&max_price={max}&min_bedrooms={beds}` |
