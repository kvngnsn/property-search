# property-search

A [Claude Code](https://claude.com/claude-code) skill that finds homes and rentals for you. Describe what you want in plain English. It searches the web widely (major listing sites, local property managers, apartment complex sites, and classifieds), reads public listing pages, and gives you one deduplicated table sorted by price, with links to the original listings.

> [!IMPORTANT]
> **Read [LEGAL.md](LEGAL.md) before you use this.** This skill makes automated requests to websites. Some listing sites, including Zillow and Redfin, prohibit any automated access in their terms of use. You're responsible for following the terms of each site you use it on.

## What's a skill?

A skill is a folder with a `SKILL.md` file inside it: plain-English instructions, plus a short header saying when they apply. Claude Code reads that header for every skill you've installed, and when your request matches one, it loads that skill's instructions and follows them. There's no code to run and no API keys to set up. It's a recipe that Claude follows using its built-in web search and web fetch tools.

## Install

You need [Claude Code](https://claude.com/claude-code) with its built-in web search and web fetch tools enabled.

```bash
git clone https://github.com/kvngnsn/property-search.git
mkdir -p ~/.claude/skills
cp -R property-search/skills/property-search ~/.claude/skills/
```

Then restart Claude Code.

To update, run `git pull` in the cloned folder and copy the skill folder again. To uninstall, run `rm -rf ~/.claude/skills/property-search`.

## Use

Just ask:

```
find me 2-bed rentals in 94568 under $3,500
```

Or type `/property-search` and it will ask what you're looking for.

More examples:

- `3-bed houses for sale in Austin under $600k with a yard`
- `studio or 1-bed in Capitol Hill, Seattle, under $2,200, cat-friendly, move in Dec 1`
- `townhomes for rent near downtown Raleigh, 2+ beds, 2 baths, max $2,800, need parking`

It only asks for what's missing. Location, rent or buy, and budget are required. Everything else (baths, property type, pets, move-in date, must-haves) is optional, and it tells you what it assumed.

## What you get

A table like this one (the addresses here are made up):

| # | Price | Address | Beds | Baths | Sqft | Available | Pets | Sources |
|---|---|---|---|---|---|---|---|---|
| 1 | $2,950/mo | [100 Example Ave #12, Austin, TX 78704](https://example.com) | 2 | 1 | 870 | Now | Cats | [Craigslist](https://example.com) |
| 2 | $3,195/mo | [123 Sample St #4, Austin, TX 78704](https://example.com) | 2 | 2 | 1,050 | Dec 1 | Cats, dogs | [Apartments.com](https://example.com), [Zumper](https://example.com) |

Listings it found only in search-result snippets, because it couldn't open the page, go in a second, clearly labeled table. Below that you get direct search links for any site it couldn't or didn't read, so you can open them yourself. It then offers to save everything to `property-results-<date>.md` in your current folder.

## Built-in rules

The skill follows these on every search (full text in [`SKILL.md`](skills/property-search/SKILL.md)):

- At most about 30 page fetches per search. No loops or scheduled re-runs.
- If a site blocks it, shows a CAPTCHA, or needs a login, it skips the site and gives you a link instead. It never bypasses blocks, uses proxies, rotates user agents, or logs in.
- It respects robots.txt.
- It never stores or republishes listing photos or full descriptions. It links to the source.
- It never filters or ranks by protected characteristics under the Fair Housing Act.
- Results are for you, not for building a database or another site.

## Limitations

- Many big listing sites block automated access, so for those you'll often get a link to open yourself rather than results.
- Listings change fast and search snippets can be stale. Always confirm on the original listing before you apply or make an offer.
- It finds listings. It doesn't contact landlords or agents, schedule tours, or apply for you.

## Contributing

Issues and pull requests are welcome, especially fixes for search-link URL formats that have changed. Changes that loosen the hard rules in `SKILL.md`, such as raising the fetch budget or working around blocks, won't be accepted.

## License

[MIT](LICENSE). Provided as is, with no warranty. This is not legal advice; see [LEGAL.md](LEGAL.md).
