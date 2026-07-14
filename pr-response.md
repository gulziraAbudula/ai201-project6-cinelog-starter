# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:**
I renamed all save_to_watchlist() to add_to_watchlist(), I used git grep "save_to_watchlist" to find the function name exists in the project, and renamed it.
**How I verified:**
I checked both files and checked if save_to_watchlist() exist, and it doesn't exist anywhere after rename.

## Comment 2 — Deduplication
**What I did:**
I modified watchlist_service add_to_watchlist(user_id, film_id) function to match the pattern that services/collection_service add_to_collection() use. add_to_watchlist(user_id, film_id) already had logic related to existing check - FilmNotFoundError, it did not have the logic that checks if it already checks for if the film already exists and it is in the watchlist. 
**How I verified:**
I verified by checking the functionality of add_to_collection() method line by line, and same pattern for add_to_watchlist(). And later added a unit test for this functionality.

## Comment 3 — Missing test
**What I did:**
I added a unit test for add_to_watchlist() deduplicate logic, I followed the same pattern as the test case for add_to_collection(). The test function raises FilmNotFoundError if add_to_watchlist fails
**How I verified:**
I ran the pytest specific to this function - pytest tests/test_watchlist.py -v, and pytest related to all tests. 

## Comment 4 — Default visibility
**My position:** Keep `public=True` as the default on `WatchlistEntry`.

**Reasoning — the behavior I'm optimizing for:**
CineLog is a *social* film-logging app, not a private notes tool. The value of a
watchlist here is discovery and recommendation — friends seeing "what you're
planning to watch next" is the feature that drives engagement and the network
effect the product depends on. Defaulting to public means the social layer works
the moment a user adds their first film; they don't have to hunt for a setting to
turn sharing on. A user who signs up for a social app generally *expects* their
activity to be visible, so `public=True` matches that expectation and lowers the
friction to the feature actually being useful.

**Tradeoff acknowledged:**
The honest counterargument is privacy-by-default. With `public=True`, an entry is
exposed the instant it's created — before the user has made any conscious decision
to share it. Some people treat a watchlist as private and aspirational ("films I'm
a little embarrassed I haven't seen yet"), and a public default surprises them.
The safer, more principled default is `public=False`: put the burden on *opting in*
to exposure rather than *opting out*, which is what "privacy by design" would
argue for. I'm accepting that real privacy cost in favor of the social-product
goal — but I'd only defend `public=True` if the per-entry `public` toggle is
surfaced prominently in the UI, so opting an entry back to private is a single,
obvious action rather than a buried setting. If this were a general-purpose or
regulated app instead of a social one, I'd flip to `public=False`.

## Comment 5 — Sort order
**My position:** Sort by `date_added` descending (newest first) — I agree with the
maintainer's preference.

**Engagement with reviewer's point:**
The maintainer prefers date-added, and I think that's right because a watchlist is
a *queue of intent*, not a reference catalog. Users add films as they discover
them, and the thing they come back to answer is "what should I watch next?" —
which the most recently added items answer best. Alphabetical sorting treats the
list like a static library you look titles up in; but you rarely scan a watchlist
by title, and alphabetical has a concrete downside: a newly added film gets buried
wherever the alphabet drops it, so the user loses sight of what they just added.
Date-added-descending keeps the freshest intent at the top.

There's also a consistency argument: `get_collection()` already returns entries
sorted by `date_added` descending (see `test_get_collection_returns_newest_first`),
so sorting the watchlist the same way gives users one consistent mental model
across both lists instead of two different rules.

**Where alphabetical would win, and my third option:** Alphabetical is genuinely
better once a list is large and the user knows the exact title they're looking
for. So rather than hard-code one, I'd make `date_added` DESC the *default* and
accept an optional `sort` query param (e.g. `?sort=title`) later. That keeps the
newest-first default the maintainer and I both want, while leaving room for
lookup-by-title without a breaking change.

## Comment 6 — Rebase
**What conflicted:**
I rebased `feature/watchlist` onto the updated `main`. The only conflict was in
`.gitignore` — both branches had added one. The UUID migration from main merged
cleanly with no conflict.

**How I resolved it:**
The two `.gitignore` versions only differed by one extra line (`.pytest_cache/`),
so I kept all the lines from both, removed the conflict markers, then ran
`git add .gitignore` and `git rebase --continue`.

**How I verified no conflict remains:**
`git status` showed a clean working tree with no unmerged files, and there are no
`<<<<<<<`/`>>>>>>>` markers left in the repo. `git log` shows my commits stacked
on top of main with a linear history (no merge commits).

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->