# adamliscia.com

One-page portfolio. Plain HTML — no build step, no dependencies. `index.html` is the
whole site; fonts come from Google Fonts, everything else is inline.

## Preview locally

```bash
cd ~/career/site && python3 -m http.server 8899
# → http://localhost:8899
```

## Design notes

Redesigned 2026-11-04 when Adam asked for it to look "as if I paid someone $15,000 to build it".

- **Palette**: bone (`#F5F4F0`), ink navy (`#101828`), brass (`#A0692A` / `#E2B26A` on dark).
  Full dark-mode set via `prefers-color-scheme`.
- **Type**: Instrument Serif for display (italic carries the accent), Geist for body and
  Geist Mono for labels.
- **Structure**: sticky blurred nav → big serif hero → the growth chart as a dark "board" →
  three numbered case studies (**Nerd Leagues → BarCraft → Nerd Leagues TV**, the ranking in
  `background.md`) → How I work → About + timeline → dark contact block.
- **Copy follows the description spec**: Nerd Leagues carries one number, "nearly 800
  players". BarCraft and TV carry none. No latency or security trivia. The text is cut from
  `resume/master.md`.
- **Motion**: scroll reveals, and the chart draws itself when it scrolls into view. All of it
  is skipped under `prefers-reduced-motion`. Without JS everything renders visible
  (`.js` gate).
- **No photo.** The only headshot (`~/repo-archive/adam-liscia-headshot.png`) is a casual shot.
  Add one only if Adam supplies a professional one.
- **Written for a non-engineer.** Each project says what it is, who uses it and what he did.
- **Screenshots** live in `img/`. Never capture nerdleagues.com while logged in.

## Keeping the numbers honest

The stat card and the chart endpoint both say **724** and a reader can compare them
directly, so they must stay equal. Re-pulled 2026-10-17. Update all of these together:

| Where | Currently |
|---|---|
| `SERIES` array (bottom of `index.html`) | ends `["2026-10",724]` |
| `.big` headline | `~800` players (the 2026-10-28 total, 778). The chart still ends at 724 from the 2026-10-17 pull; both are dated in the caption and footer |
| `<meta name="description">` | "724 members" |
| SVG `aria-label` | "rising from 1 to 724" — and the month range |
| `figcaption` | "May 2024 – Oct 2026" |
| footer | per-project dates, see below |
| `yMax` + gridline array | `800` and `[0, 400, 800]` |

**724, not 726.** The series counts users with a non-NULL `created_at`; `COUNT(*)` returns
726 because two rows have no signup timestamp. The stat card uses 724 so the number and the
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

`CNAME` is set to `adamliscia.com` (the apex is canonical since 2026-10-22; www redirects to it). DNS is already half-configured — `www` CNAMEs to
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
