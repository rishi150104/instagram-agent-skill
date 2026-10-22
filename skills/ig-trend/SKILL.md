---
name: ig-trend
description: >-
  Give it a keyword or topic and it finds the reels on that topic with the most
  views in the last 90 days, listed in descending order of views, each with its
  outlier multiple over its own account's median and a link to the reel. Use
  when the user says "what's trending on X", "top reels about X", "find me
  reels on this topic", "20 most viewed reels for this keyword", or has a topic
  and no accounts to start from. For account-driven research, use /ig-viral.
---

# ig-trend

Topic in, up to 20 reels out. `/ig-viral` starts from accounts; this starts from
a keyword. Run it when the user has a subject and no idea who is winning at it.

Instagram has no public keyword search that returns reels sorted by views, and
no API that gives you other people's view counts. So this is reading, at human
speed, in the user's own browser. The same rules as `/ig-viral` apply:

- **Never log in on the user's behalf and never ask for a password.** If the
  browser lands on a login page, stop and ask the user to log in themselves.
- **Reading, not scraping.** One search, a few dozen posts, one visit per
  account. No crawler, no scraping service, no background loop.
- **Copy the formula, never the video.** Attribute every row to its account.

## Where the numbers actually are

This was tested live and the obvious assumption was wrong. Know this first:

| page | views shown? |
| --- | --- |
| Search results grid | **No.** Tiles are just thumbnails. |
| A single reel page | **No.** Likes and comments only, plus the posted date. |
| An account's **Reels tab** (`/handle/reels/`) | **Yes**, on every tile, as text ("455K", "8,435"). |

So the view count for a reel can only be read from its own account's Reels tab,
and ranking by views can only happen **after** visiting the accounts. The flow
below is built around that.

## Step 1: pin down the query

Ask for, or confirm:

- **Keyword or topic.** One phrase. If it is broad ("fitness"), ask for the
  angle, because the top reels for a broad word are whatever is biggest.
- **Window.** Default **90 days**. The user can override it.
- **Count.** Default 20. Fewer is a valid result (see the end).

## Step 2: collect candidates from search

Use a browser tool connected to the user's own Chrome. Check first that they
are logged in.

1. Open `https://www.instagram.com/explore/search/keyword/?q=<topic>` (spaces as
   `%20`).
2. Scroll to load more tiles (about 20 scroll ticks gets 40 to 50 posts).
3. Collect every tile link matching `/p/<code>/` or `/reel/<code>/`. The code
   is the reel's identity for the rest of the run.

The grid mixes reels, carousels and image posts, and they can be old. Filter in
the next step.

## Step 3: open each candidate for account, date and type

Open each candidate post and read three things from the page:

- **Account:** the first profile link in `main` (`/handle/`).
- **Posted date:** the `<time>` element's `datetime` attribute.
- **Is it a reel:** the page has a `<video>`. No video, skip it.

Drop anything outside the window. **Do not guess the date from the shortcode.**
Codes sort roughly by time but not tightly enough to trust near the window
edge: check the date for anything close to the cutoff.

This is the slow step, one page per candidate. A typical run has about 50 posts
in the search and about 20 reels that survive the date and type filter.

## Step 4: read views and the median from each account's Reels tab

For each distinct account (once per account, not once per reel):

1. Open `https://www.instagram.com/<handle>/reels/`.
2. Read **followers** from the profile header.
3. Read the first 12 tiles' view counts. **Leave out pinned reels**: they are
   old picks and inflate the median. Median = middle value of those.
4. Find the candidate by its shortcode among the tiles, scrolling down until it
   appears (about 8 to 14 scrolls at most). Its tile text is the view count.
5. If the candidate never appears, the reel is not on that account's Reels tab
   (collab posts, removed reels, tab not showing it). **Drop it.** Do not
   estimate its views.

Rules for the numbers:

- Keep views as shown ("455K"). Convert to a number for ranking only.
- Multiple = candidate views divided by the account's median.
- Median needs about 6 or more non-pinned reels. Fewer: mark the multiple blank
  and say which rows.
- The median uses the account's latest reels, not the reels from the window. It
  is a measure of a normal day for that account now.

Tooling note for the JavaScript tool: use top-level `await` and return a string
(`JSON.stringify(...)`). An async function wrapped in an IIFE returns an empty
object.

## Step 5: rank and report

Sort by **views, descending**, as the user asked, with the multiple beside it.
Do not re-sort by multiple. No posted-date column; the reel URL goes in its
place. Reel URL format: `https://www.instagram.com/reel/<code>/`.

```
TREND  ·  "landing page optimization"  ·  last 90 days  ·  15 reels  ·  13 accounts

 #  views    x median  account (followers)        reel URL
 1  455K      13.5x    @ayzz.thedesigner (271K)   https://www.instagram.com/reel/Dcly790oumZ/
 2  347K      22.8x    @thecuttingedge.school ... https://www.instagram.com/reel/DcL0QNnTwY5/
 3  324K      80x      @yatesvids (9.6K)          https://www.instagram.com/reel/Dcp1PduBGnH/
 ...
```

Then say three things and stop:

1. **Where views and multiple disagree.** A reel high on views and at or below
   1x is that account's normal day. Copy nothing from it. A reel with fewer
   views and a high multiple, especially on a small account, is the one that
   found something.
2. **Which reels to look at first**, by multiple, small accounts first.
3. **Sample and confidence in plain words.** One search supports "this is what
   the search surfaced", not "this is what works".

## Optional: hooks and formulas

The table above does not need hooks. To name formulas and score them, play each
reel you care about with the user, write down the first spoken or on-screen
line verbatim, and run `swipe.py` from `/ig-viral` on a TSV:

```
account	followers	median	views	hook
```

```bash
python3 ../ig-viral/swipe.py captured.tsv --out ~/.claude/instagram/swipe.md
```

Hooks are not readable from the page, so do this for the top few, not all.
Hand the best formulas to `/ig-reel` with the formula id already chosen.

## What this cannot tell you

- **Fewer than 20 is normal.** Candidates that are old, carousels, or missing
  from their account's Reels tab are dropped. Report what survived, and say how
  many candidates were dropped and why. Do not pad the list.
- The list is "top views among what the search surfaced". A reel Instagram did
  not show for this search is missing.
- View counts are rounded and are a snapshot of the day you looked.
- Medians are approximate when an account has few reels.
- Cost: about 50 post pages plus one profile per account is dozens of browser
  steps. Tell the user it will take a while before starting.

## Fallback: YouTube Shorts

If the user is not logged in and will not log in, or Instagram will not show
enough, run the same search on YouTube Shorts, where views and captions are
public and no login is involved. `/ig-viral` has the `yt_dlp` commands. Label
the output as Shorts, not Reels, every time.

Nothing is posted, followed, liked or messaged by this skill. It reads.
