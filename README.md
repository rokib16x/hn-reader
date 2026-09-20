# HN Reader

A faster, searchable Hacker News front end. One self-contained `index.html` — no build step, no dependencies, no framework, no account, no tracking.

**[Live demo →](https://rokib16x.github.io/hn-reader/)**

Hacker News has no search box. This adds one, along with time-window filtering, whole comment threads in a single request, and a library of saved stories and followed authors that lives entirely in your browser.

```bash
git clone https://github.com/rokib16x/hn-reader.git
cd hn-reader
python3 -m http.server 4180
# open http://localhost:4180
```

That's the whole setup. It's a static file — drop it on GitHub Pages, Netlify, S3, or open it over any web server.

## Features

- **Feeds** — Top, New, Best, Ask, Show, Jobs, paginated 30 at a time
- **Search** — the thing news.ycombinator.com doesn't have. Debounced as you type, `⌘K` to focus, scoped to stories / comments / everything / Ask HN / Show HN, sortable by relevance or date, with live hit count and query time
- **Time windows** — 24 Hours / Week / Month / All Time, which transparently switch the data source
- **Threaded comments** — whole trees in one request, per-thread collapse, lazy "load N more replies"
- **Saved stories** — bookmark from the list, the row menu, or the story page
- **Followed authors** — follow from a profile or a story menu; their newest submissions merge into one feed
- **User profiles** — karma, join date, bio, recent submissions
- List / Grid layouts, light + dark themes, visited-story dimming

Read-only by design. Anything that needs an account links out to news.ycombinator.com.

## Why two APIs

The official [HackerNews/API](https://github.com/HackerNews/API) is a read-only Firebase mirror with no search, no pagination and no filtering. The [Algolia HN Search API](https://hn.algolia.com/api) — HN's own search backend — fills those gaps. This app uses each for what it does best.

### Firebase — `https://hacker-news.firebaseio.com/v0`

| Endpoint | Used for |
|---|---|
| `topstories` `newstories` `beststories` `askstories` `showstories` `jobstories` | live ranked feeds (arrays of up to 500 ids) |
| `item/{id}` | story records with authentic current score |
| `user/{id}` | karma, join date, about, `submitted[]` |
| `maxitem` `updates` | unused; available for a live-tail feature |

Comment `kids` are ids only, so a full thread would cost one request per comment.

### Algolia — `https://hn.algolia.com/api/v1`

| Endpoint / param | Used for |
|---|---|
| `search` / `search_by_date` | the search page, relevance or newest |
| `tags=story,comment,ask_hn,show_hn,job,author_x` | search scopes, section feeds, the Following feed |
| `numericFilters=created_at_i>…` | the 24 Hours / Week / Month windows |
| `items/{id}` | the **entire comment tree in one request** |

A 71-comment thread costs one HTTP request instead of 71.

## There is no login, and there cannot be one

The official API is read-only: every endpoint is a `GET`, with no auth, token, API key or write path. `user/{id}` returns only the public profile. Voting, commenting and submitting exist solely as cookie-authenticated form posts to news.ycombinator.com, which are CORS-blocked from a browser and would mean proxying your password through a server.

So the library **is** the account. Saved stories and follows live in `localStorage`, in one browser: no password, no server, nothing sent anywhere, and no sync between devices.

## Design

Typeset in **Manrope**, with **IBM Plex Mono** carrying every piece of utility text — ranks, scores, timestamps, domains, section labels, controls.

Structure comes from four surface planes rather than from cards:

| Token | Role |
|---|---|
| `--shell` | chrome behind everything — masthead, sidebar, colophon |
| `--bg` | the reading surface |
| `--panel` | inset bands — control strips, section headers, quoted text |
| `--raise` | things above the page — menus, toasts |

The masthead, sidebar and content column are separated both by a `--line-2` edge and by a step in surface value, so chrome never blends into what you're reading. Corners are square, there are no drop shadows or gradient fills, and headlines run at light weights with tight negative tracking.

Colour is spent sparingly — the wordmark, the active feed rule, saved bookmarks, and scores above 1,000 points. Active controls use a solid inversion (the text colour becomes the fill) rather than a translucent accent wash, which reads muddy over a tinted panel. Every colour is a token defined for both themes.

## Implementation notes

- **Outbound links** open via `window.open` from a delegated click handler, because some embedded browsers silently ignore `target="_blank"`. The reader window is never navigated away, and `↗` marks every link that leaves it. If a pop-up is blocked, the page says so rather than failing silently.
- **Comment HTML is sanitised** to an allowlist of tags before insertion; links are forced to `rel="noopener noreferrer"` and non-`http(s)` hrefs are unwrapped.
- **No layout shift on filter changes** — the list pins its height while loading, so the document never collapses to the skeleton and springs back. Sticky offsets all derive from one `--topbar` token.
- **`localStorage` access is wrapped in try/catch** throughout, so the page still works in private windows and with site data blocked.

## Contributing

Issues and pull requests are welcome. It's one file — open `index.html`, edit, reload. Keep it dependency-free and keep both themes working.

## Licence

[MIT](LICENSE).

Not affiliated with, endorsed by, or operated by Y Combinator or Hacker News. "Hacker News" and the Y Combinator logo belong to Y Combinator. This is an independent client for their public API.
