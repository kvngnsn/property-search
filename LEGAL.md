# Legal notes

This page explains in plain English what this skill does, where the legal gray areas are, and who is responsible for what.

**This is not legal advice.** See the [disclaimer](#disclaimer) at the end.

## How it works

When you ask it to, your own Claude Code session uses its web search and web fetch tools to read public listing pages: the same pages anyone can see in a browser without logging in. It reads a limited number of pages per request (about 30 at most), which is roughly what a person might click through during an evening of house hunting, though it opens them faster than a person would. It then gives you a table of basic facts (price, beds, baths, address, and so on) with a link back to each original listing.

It runs only when you ask. It doesn't run on a schedule, doesn't log in to anything, and doesn't keep listings anywhere except a results file in your current folder, and only if you ask it to save one.

## The nuance: this is still automated access

Low volume and good manners don't make this the same as browsing by hand. When Claude Code fetches a page, an automated program is accessing that site. That's true even though you asked for it and even though it reads only a handful of pages.

Some sites' terms of use prohibit automated access outright, whatever the volume or purpose. For example:

- **Redfin's** Terms of Use (version dated September 29, 2025) say you may not "automatedly crawl or query the Services for any purpose or by any means", including "spiders, robots, crawlers and any other automated activity", without Redfin's prior express written permission.
- **Zillow's** Terms of Use likewise prohibit using robots, spiders, scrapers, or other automated means to access its services without permission.

Neither has an exception for personal use, low volume, or an AI agent acting for a person. Read plainly, they cover an AI agent like this one. **Using this skill on those sites may breach their terms**, and other sites may have similar terms. Terms change, so check the current versions yourself.

A few more things worth knowing:

- **robots.txt is not the same as terms of use.** The skill respects robots.txt, but a site can allow a page in robots.txt and still prohibit automated access in its terms.
- **The law is unsettled.** How enforceable these terms are, especially against someone who never created an account or clicked "I agree", is a contested question, and the answer varies by country and state. This document doesn't try to answer it.

**You are responsible** for checking and following the terms of each site the skill reads. If a site's terms prohibit automated access and you want to stay within them, don't use the skill on that site. You can tell Claude something like "don't fetch from Zillow or Redfin, just give me links", then open those links and browse by hand.

## What the skill does to reduce risk

These rules are written into [`SKILL.md`](skills/property-search/SKILL.md) as hard rules:

- **Low volume.** At most about 30 page fetches per search. No loops, no polling, and no scheduled re-runs.
- **Blocks are respected.** If a site blocks it, shows a CAPTCHA, or asks for a login, the skill skips that site and gives you a direct link to open yourself. It never uses proxies, rotates or spoofs user agents, logs in, or otherwise tries to get around a block.
- **robots.txt is respected.** The skill checks robots.txt before reading pages on a site and skips disallowed pages.
- **Links, not copies.** It doesn't store or republish listing photos or full descriptions, which are usually someone else's copyrighted work. It records a few basic facts and links back to the source.
- **Fair housing.** It won't filter or rank listings by protected characteristics such as race, color, religion, national origin, sex, disability, or familial status, which are protected under the federal Fair Housing Act, or by characteristics protected under state and local law.
- **Personal use only.** Results are for you. The skill isn't meant for building a database, a dataset, or another listing site.

These rules reduce risk. They don't eliminate it, and they don't make automated access permitted where a site's terms prohibit it.

## Listing accuracy

Listings go stale quickly, and search snippets can be out of date. The skill may miss listings, show ones that are already gone, or misread a page. Always confirm details on the original listing. Never send a deposit or personal documents before seeing a property and verifying who you're dealing with.

## Disclaimer

- This project is provided **"as is"**, without warranty of any kind, under the [MIT License](LICENSE).
- The author isn't responsible for how you use it or for what happens as a result, including any breach of a website's terms, loss of access to a site, or decisions made based on inaccurate or out-of-date listings.
- **This is not legal advice**, and reading it doesn't create an attorney–client relationship with anyone. It's a general, good-faith description of the issues as the author understands them.
- If you need to know whether your use is allowed, talk to a lawyer licensed where you live about your specific situation.
