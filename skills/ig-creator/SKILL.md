---
name: ig-creator
description: >-
  Give it a niche and it finds small Instagram creators in that niche: accounts
  with under 2,000 followers and at least 30 posts, listed with followers, post
  count, bio and a profile link. Use when the user says "find small creators in
  X", "micro creators", "accounts under 2k followers", "who is starting out in
  this niche", or wants collab, comment or outreach targets and has no names.
---

# ig-creator

Niche in, up to 20 small accounts out. `/ig-trend` finds reels; this finds the
people. Run it when the user needs small accounts to study, engage with or
approach, and does not know any.

Instagram has no API that filters accounts by follower count. So this is
reading, at human speed, in the user's own browser. The same rules as
`/ig-trend` apply:

- **Never log in on the user's behalf and never ask for a password.** If the
  browser lands on a login page, stop and ask the user to log in themselves.
- **Reading, not scraping.** A few searches, a few dozen posts, one visit per
  account. No crawler, no scraping service, no background loop.
- Nothing is followed, liked, commented or messaged by this skill. It reads.

## Step 1: pin down the query

Ask for, or confirm:

- **Niche.** One phrase. If it is broad ("fitness"), ask for the angle, because
  a broad word surfaces whoever is biggest, and big accounts are what we drop.
- **Count.** Default 20. Fewer is a valid result.
- **Thresholds.** Followers **under 2,000**, posts **at least 30**. The user
  can override either. There is no posting-frequency check.

Tell the user it will take a while before starting: dozens of browser steps.

## Step 2: collect candidate accounts

Check first that the user is logged in.

Use both approaches and pool the results:

1. **Keyword search:** open `https://www.instagram.com/explore/search/keyword/?q=<niche>`
   (spaces as `%20`).
2. **Hashtag search:** open
   `https://www.instagram.com/explore/search/keyword/?q=%23<tag>` for a few
   hashtags that fit the niche (the `/explore/tags/<tag>/` URL just redirects
   there). Instagram shows no post count for a tag.
3. Scroll about 40 ticks per page (about 300 posts on a busy hashtag; the
   count was still climbing steadily at 40, and posts further down came from
   smaller accounts). Collect every tile link matching
   `/p/<code>/` or `/reel/<code>/`.
4. Open each post and read the account: the first profile link in `main`
   (`/handle/`). Dedupe handles across all pages. The handle is not on the
   search tile (the image alt text is the caption), so each post must be opened.

## Step 3: check each account once

Open `https://www.instagram.com/<handle>/` and read the header:

- **Followers** and **posts**. The fastest read is the page's
  `meta[property="og:description"]`, which says `256 Followers, 15 Following,
  331 Posts`. The header `li` elements are not reliable. Convert "1,204" and
  "12.5K" to numbers. Under 10K the figure is exact, so the 2,000 cut is safe.
- **Check followers first.** Most candidates are over 2,000. Stop reading that
  profile the moment it fails.
- Keep only followers < 2,000 and posts >= 30.
- **Private accounts:** skip. The grid cannot be read, so niche fit cannot be
  checked. Count them and say so.

## Step 4: check niche fit

For each survivor, read the bio and look at the first row of the grid. Drop
accounts clearly off-niche (a shop that happened to use the tag, a personal
account with one on-topic post).

## Step 5: report

Sort by **followers, descending** (closest to 2,000 first), then posts.

```
CREATORS  ·  "home baking"  ·  14 found  ·  61 accounts checked

 #  account          followers  posts  bio                          profile URL
 1  @flour.and.fern     987       112   sourdough for beginners      https://www.instagram.com/flour.and.fern/
 2  @loafnotes          842        47   weekend bakes, honest fails  https://www.instagram.com/loafnotes/
 ...
```

Then say two things and stop:

1. **What was dropped and why**, as counts: too big, under 30 posts, private,
   off-niche.
2. **Sample and confidence in plain words.** This is what the search and tags
   surfaced, not every small creator in the niche.

## What this cannot tell you

- **Fewer than the target is normal.** Report what survived. Do not pad.
- Counts are a snapshot of the day you looked.
- It cannot tell if an account is active. There is no cadence check, so an
  account with 30 posts from two years ago passes. Say so if the user asks
  about "active".
- Cost: one post page per candidate plus one profile per account.

## Hand-offs

- `/ig-viral`: use the list as the account list to study their reels.
- `/ig-comment` and `/ig-dm`: for engaging with or approaching these accounts.
