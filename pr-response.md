# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in
`services/watchlist_service.py` to match the project's `verb_to_noun` naming
convention already used by `add_to_collection()`, `remove_from_collection()`,
and `get_collection()`. Updated the one call site in
`routes/watchlist/watchlist.py` (both the import and the function call).
**How I verified:** Ran `grep -rn "save_to_watchlist" .` to confirm no
remaining references to the old name anywhere in the codebase, then ran
`pytest tests/ -v` to confirm nothing broke.

## Comment 2 — Deduplication
**What I did:** Added a duplicate check to `add_to_watchlist()`, mirroring
the pattern already used in `add_to_collection()`: after confirming the film
exists, query for an existing `WatchlistEntry` with the same `user_id` and
`film_id`. If one exists, raise a new `AlreadyInWatchlistError` instead of
creating a duplicate row.
**How I verified:** Ran `pytest tests/ -v` after the change to confirm
existing tests still pass, and added a dedicated test
(`test_add_to_watchlist_duplicate_raises`) confirming that adding the same
film twice raises `AlreadyInWatchlistError` and that only one entry exists
in the database afterward.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py`, modeled directly on
`test_add_to_collection_nonexistent_film_raises` from `test_collection.py`.
Added `test_add_to_watchlist_nonexistent_film_raises`, which confirms that
calling `add_to_watchlist()` with a film_id that doesn't exist raises
`FilmNotFoundError`. I also added two additional tests beyond what was
strictly requested — a happy-path test (`test_add_to_watchlist_creates_entry`)
and a duplicate test (`test_add_to_watchlist_duplicate_raises`) — to match
the same three-part coverage (happy path, duplicate, nonexistent ID) that
`CONTRIBUTING.md` requires for new service functions.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` to confirm all
three new tests pass, then `pytest tests/ -v` to confirm the full suite
still passes together.

## Comment 4 — Default visibility
**My position:** Watchlists should default to `public=False`, not `True`.

**Reasoning:** A watchlist is different from a collection — it's a list of
films someone hasn't watched yet, which can reveal more about a person's
current interests or mood than a list of films they've already finished.
Users should have to opt in to sharing that, not opt out. This also matches
how I use similar apps myself — as a Letterboxd user, I always set my own
watchlist to private, since it's a personal planning list rather than
something I default to sharing. Defaulting to private respects that most
users won't have thought about the visibility of a feature they haven't
used yet, and it's much easier to make something public later (a deliberate
choice) than to discover it was public the whole time (an accidental
exposure).

**Tradeoff acknowledged:** The original default (`public=True`) optimizes
for CineLog's community aspect — public watchlists could drive discovery,
letting users see what films are trending among people they follow.
Defaulting to private sacrifices some of that discovery value, since fewer
watchlists will be visible unless users actively opt in. I think that
tradeoff is worth it here because the downside of over-sharing (unintentional
exposure) is worse than the downside of under-sharing (slightly less
discovery), especially for a feature users haven't had a chance to
understand yet.

## Comment 5 — Sort order
**My position:** I agree with the maintainer — sort by date added
(newest first), not alphabetical.

**Reasoning:** A watchlist is inherently a "what's next" list, not a
reference list. Most people build it up over time and want to see what
they just added, either because it's top of mind or because they're
deciding what to watch tonight. Alphabetical order is more useful for
looking something up you already know is there — closer to how you'd use
an already-watched collection — but a watchlist behaves more like a queue.

**Engagement with reviewer's point:** The maintainer's reasoning — "most
users want to see what they added recently" — matches my own experience
with similar watchlist features: I rarely scroll alphabetically to find
something to watch, I look at what's newest because that's usually what
I was just thinking about when I added it. Sorting by `date_added`
descending also keeps this consistent with `get_collection()`, which
already sorts newest first, so the two features behave the same way from
a user's perspective instead of introducing an unexplained inconsistency
between them.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->