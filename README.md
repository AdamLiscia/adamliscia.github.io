# adamliscia.com

One-page portfolio. Plain HTML — no build step, no dependencies. `index.html` is the
whole site; fonts come from Google Fonts, everything else is inline.

## Preview locally

```bash
cd ~/career/site && python3 -m http.server 8899
# → http://localhost:8899
```

## Design notes

- **Palette** — blueprint paper (`#E9EDF4`), ink navy (`#131E33`), brass (`#9C6A24`).
  Deliberately cool rather than the cream-and-terracotta default. Full dark-mode set via
  `prefers-color-scheme`.
- **Type** — Spectral for display, IBM Plex Sans for body, IBM Plex Mono for data and
  labels. The mono carries the measurement theme; Plex was drawn for an engineering company.
- **The rail** — the vertical line at the left with a brass tick at each section start.
  It's an axis: the ticks mark intervals, they aren't ornament. Hidden below 1000px.
- **The chart is the signature.** Real cumulative signup data pulled from the Nerd Leagues
  database on 2026-08-09, hardcoded in `SERIES` at the bottom of the file. The stroke
  draws itself on load and the animation is skipped under `prefers-reduced-motion`.

## Keeping the numbers honest

The stat card and the chart endpoint both say **619** and a reader can compare them
directly, so they must stay equal. Re-pulled 2026-09-17. Update all of these together:

| Where | Currently |
|---|---|
| `SERIES` array (bottom of `index.html`) | ends `["2026-09",619]` |
| `.stat` card | `619` users, `1,230` decks, `514` games, `144` articles |
| `<meta name="description">` | "619 users" |
| SVG `aria-label` | "rising from 1 to 619" — and the month range |
| `figcaption` | "May 2024 – Sep 2026" |
| footer | per-project dates, see below |
| `yMax` + gridline array | `700` and `[0, 350, 700]` |

**619, not 620.** The series counts users with a non-NULL `created_at`; `COUNT(*)` returns
620 because one row has no signup timestamp. The stat card uses 619 so the number and the
curve agree. See `career/metrics.md` → "Note on 488 / 489 / 490".

**The y-axis is not automatic.** It was fixed at 0–500 and the real number passed it, which
would have clipped the curve off the top of the chart. It is now 0–700. Check this every
re-pull — it is the failure that does not announce itself.

**The footer dates each project separately, deliberately.** Only the Nerd Leagues figures
are September 2026. Nerd Leagues TV's counts are launch-day (2026-08-13) and BarCraft's are
2026-08-14, and there is no script for either — `scripts/resume_metrics.py` reads the Nerd
Leagues web DB only. A single blanket date on the footer would silently re-date the other
two by a month.

### Re-pulling the series

`resume_metrics.py` gives the totals but has no monthly-series query. The one used on
2026-09-17, run with the Nerd Leagues venv so `pymysql`/`dotenv` resolve:

```sql
SELECT DATE_FORMAT(created_at,'%Y-%m') m, COUNT(*) n
FROM users WHERE created_at IS NOT NULL GROUP BY m ORDER BY m;
```

then accumulate in order. Historical points do move between pulls: the old series ended
`["2026-08",489]` because that pull was taken on 9 August — the full month closed at 563.
Re-pull the whole series, don't append a point to the old one.

## Deploying to GitHub Pages

`CNAME` is set to `www.adamliscia.com`. DNS is already half-configured — `www` CNAMEs to
`adamliscia.github.io`, so the missing piece is the repo.

1. Create a **public** repo named `adamliscia.github.io` (a user site serves at the root).
2. Push the contents of this directory to its default branch.
3. Settings → Pages → set the custom domain to `www.adamliscia.com`, then enable
   **Enforce HTTPS** once the certificate is issued (can take up to an hour).
4. In **GoDaddy**, delete the apex forwarding rule that currently points at the dead
   bartoolspro.com, and add GitHub Pages' four A records so the bare domain reaches the
   same place:
   `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
5. Archive the old `adam-liscia` repo — it still serves the defunct Bar Tools Pro
   marketing page at `adamliscia.github.io/adam-liscia/`.
