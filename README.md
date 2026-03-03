# Apexlist

A **unified difficulty index** for hard games: one cross-game tier list that merges community “hardest clears” leaderboards (Eternal Towers of Hell, Geometry Dash, Celeste) into a single, comparable ranking.

The goal is to answer: *How does the #1 on Pointercrate compare to the #1 on hardclears.com or the top EToH tower?* By normalizing each game’s list into a common scale and blending in rarity (fewer clears = harder) and community input, the apexlist creates one ordered list and a **Difficulty Index (DI)** score per level so achievements across games can be debated and compared.

---

## Core idea

- **User submissions:** level ID, clear time, death count, game, enjoyment, and perceived difficulty.
- **Backend computation:** objective difficulty index from list position, clear counts, and community vote only (no death or clear time in the formula).
- **Unified list:** all levels ranked together (e.g. top 200) with tiers so players can see where a given clear sits in the global “hardest” landscape.

The list is **dynamic**: it treats each game’s current community list as the source of truth, stored in the database and refreshed (daily) so the merged ranking stays up to date.

---

## Data sources (current)

- **Geometry Dash:** [Pointercrate Demonlist](https://www.pointercrate.com/demonlist/)
- **Celeste:** [hardclears.com maps](https://www.hardclears.com/maps)
- **Eternal Towers of Hell:** [TTC community rankings](https://docs.google.com/spreadsheets/d/1bxhj0Xtixkpo_BtzsnIg-qwLTA7qdb7Y6fursQjRZKM/) Will look for a nicer source.

These are the references for “current constants” per game; the system is designed to pull from them periodically and recompute the unified index.

---

## Scoring (concept)

The DI uses **only** current ranking, clear count, and community opinion. Death count and clear time do not affect the score.

- **40% ranking:** Per-game list position, normalized so rank #1 in each game shares the same baseline (0–100 scale).
- **10% clears:** Fewer total clears → higher score (rarity = harder); 0 clears → 100, most clears in game → 0.
- **50% community vote:** Weighted average of perceived difficulty (1–10, scaled to 0–100) from runs. Voter weight = that user’s clears in the same game ÷ max clears in game (more experienced voters count more).
- **Fallback:** When a level has no votes, the 50% community slice is replaced by ranking + clears in the same 4∶1 ratio: **80% ranking + 20% clears**.



---

## User profiles and engagement

Users can (when auth is wired):

- **Submit runs:** level, clear time, enjoyment, perceived difficulty.
- **Vote on placement:** indicate where a level should sit relative to others (for contested spots).
- **Earn a profile score:** e.g. sum of DI for their clears that land in the top 200 hardest achievements, so profiles reflect “how hard is what you’ve cleared” in one number.

---

## Stack and layout

- **Frontend:** Static HTML/CSS/JS (manually written), served by Nginx in production. Lists, filters, tier view, and profile placeholders; can run in “mock” mode with local data to preview the UI without the rest of the stack.
- **API gateway:** Node (Express). Serves games, levels, runs, votes, and user profile endpoints; forwards recompute requests to the Python service; placeholders for OAuth login/callback.
- **Analysis / scoring:** Python (FastAPI). Computes DI as 40% rank score, 10% clears score, 50% community vote (weighted by voter clears in game); fallback 80% rank + 20% clears when there are no votes. Writes "objective DI" back to the database. Includes a daily sync stub to refresh levels from the external lists.
- **Database:** PostgreSQL. Schema for games, source lists, levels (with computed DI fields), users, runs, placement votes, and sync job bookkeeping.
- **Orchestration:** Docker Compose with Traefik as reverse proxy (FQDN + TLS), frontend, Node API, Python service, and Postgres. Intended to run on a Linux host behind Cloudflare (Full TLS, firewall as needed), with images updated via GitOps or similar.

---

## Repo structure (what lives where)

- **`frontend/`** - Static site (HTML, CSS, JS) and Nginx Dockerfile.
- **`backend-node/`** - Express API and Dockerfile.
- **`analysis-python/`** - FastAPI app, scoring logic, sync stub, and Dockerfile.
- **`db/`** - Postgres init script (schema + seed games/source lists).
- **`docker-compose.yml`** - Services, Traefik labels, and networks.

APEXLIST is a single product: one merged “hardest clears” tier list, fed by multiple games and kept up to date so the debate stays grounded in a shared, computed index.

## Future ideas
- **Cross game weighting logic** - Ideally factoring into the user's voting weight their cross game skill, as they will be able to have a clearer view about how each game clear ranks comparitively 
- **Cross game voting** a simple page for voting how they feel a level from x compares to y, users who have cleared maps of similar ranking on each list will be able to vote on how it compares. this will give data on how hard each game is relative to the list. (eg. top 1 celeste =/= top 1 Geometry dash in difficulty) and some how intergrate this into the logic


Made with love
By Rezzy :heart:
