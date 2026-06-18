# AUDIT — VuduList

Date: 2026-06-16
Auditor: Claude Code (Opus 4.7)
Audit type: Deep
Last commit: `3e44740` — "Merge pull request #3 from ZacharyRW/claude/claude-md-…"

> Existing `CLAUDE.md` is unusually detailed (1,000+ lines) and self-identifies the canonical implementation as `vuduupdatedbyOpenAI`. No prior `AUDIT.md` / `BACKLOG.md`.

---

## 1. Snapshot

A Python Selenium scraper for cataloging movies from a user's [Vudu](https://www.vudu.com/) (now ["Fandango at Home"](https://www.fandangoathome.com/)) account, output as CSV.

- **Source LOC**: 405 across 3 variants: `vudu.py` (91, legacy), `vuduupdatedbyclaude.py` (86, intermediate), `vuduupdatedbyOpenAI` (228, canonical — NOTE: no `.py` extension!).
- **Tests**: none.
- **CI**: none.
- **`requirements.txt`**: none.
- **`LICENSE`**: none.
- **Repo URL**: `https://github.com/ZacharyRW/VuduList`.
- **Health verdict**: 🟡 **needs attention.** The canonical implementation (`vuduupdatedbyOpenAI`) is well-structured (dataclass config, type hints, argparse, env-var auth, headless support). But: **the file has no `.py` extension** (so Python doesn't import-check or syntax-color it cleanly), **the underlying service was rebranded in 2024** (Vudu → Fandango at Home), and **the two earlier variants are kept around as deprecated cruft** when CLAUDE.md openly says "Do NOT use this as reference for new code". The whole repo is one cleanup PR away from being a mature tool — or one Vudu-URL-redirect away from being silently broken.

---

## 2. Bugs & Correctness Issues

### Net-new findings

> **N-#** = newly-surfaced; severity scale S0 (script broken / data loss) · S1 (silent wrong output) · S2 (robustness).

**N-1 · S0 · Vudu was rebranded to "Fandango at Home" in 2024.**
- `LOGIN_URL` (line 34) = `https://my.vudu.com/MyLogin.html…` — likely still works via redirect, but Fandango at Home's primary domain is now `www.fandangoathome.com`. CSS selectors (`.custom-button`, `.border .gwt-Image`, `name="email"`) may have been refreshed during the rebrand.
- The CSS class `.gwt-Image` is from Google Web Toolkit, which suggests the old Vudu site predates many UI rewrites. A modern React/Next.js rewrite would not use these classes.
- **Verify before next run**: open browser, navigate to current site, capture current login/library URLs and selectors. Update `LOGIN_URL`, `MY_MOVIES_URL`, `EMAIL_NAME`, `PASSWORD_NAME`, `SUBMIT_SELECTOR`, `MOVIE_IMG_SELECTOR`.

**N-2 · S0 · `vuduupdatedbyOpenAI` is missing the `.py` extension.**
- Python imports, IDE syntax highlighting, ruff/pyflakes static analysis, and most tooling key off the extension. Without it, the file is treated as a generic text file.
- The CLAUDE.md "File Cleanup Recommendations" section (line ~310 in CLAUDE.md) recommends renaming, but it hasn't been done.
- **Action**: `git mv vuduupdatedbyOpenAI vudu_scraper.py` and update any references.

**N-3 · S2 · No `requirements.txt`.**
- CLAUDE.md says `selenium>=4.0.0` and `webdriver-manager>=3.8.0` (optional), but there's no actual file.
- A new contributor cloning the repo can't `pip install -r requirements.txt`.
- Also affects portability: every install is "trust the user to remember the deps".

**N-4 · S1 · `vuduupdatedbyOpenAI:112-120` — `collect_visible_titles` extracts only `alt` attribute text.**
- If Fandango at Home moved to lazy-loaded image src and the `alt` is now empty (common in modern infinite-scroll UIs that hydrate text separately), the scraper returns `[]`.
- Already noted in CLAUDE.md ("If no titles were found…UI selector changed") but uncatchable until run-time.
- Recommend: assert at startup that the selector finds *at least one* element after navigation; fail fast with diagnostic info.

**N-5 · S2 · `vuduupdatedbyOpenAI:155-158` — scrolling polls 5 × 200 ms then takes another scroll.**
- Tightly coupled to network latency. On a slow network, a 1-second wait between scrolls is insufficient for lazy-loaded content. On a fast network, 1 second per scroll is wasteful for a 1000-movie library.
- Recommend: switch to adaptive wait — `WebDriverWait` on a count-changed condition instead of fixed sleeps. The CLAUDE.md Task 3 already hints at this.

**N-6 · S2 · `vuduupdatedbyOpenAI:164` — `os.makedirs(os.path.dirname(os.path.abspath(out_path)), exist_ok=True)`.**
- `os.path.dirname("./movies.csv")` returns `"."` → `os.makedirs(".", exist_ok=True)` is harmless but redundant.
- `os.path.dirname("movies.csv")` returns `""` → on Python ≤3.9, `os.makedirs("", …)` raises `FileNotFoundError`. On 3.10+, it raises `OSError`. Bug: if user passes a bare filename, this throws.
- Fix: `parent = os.path.dirname(os.path.abspath(out_path)); if parent: os.makedirs(parent, exist_ok=True)`.

**N-7 · S2 · `vuduupdatedbyOpenAI:87-102` — `login` doesn't verify whether login actually succeeded.**
- After `submit.click()`, the post-condition is `"vudu.com" in d.current_url`. But the login page also has `vudu.com` in the URL. If credentials are wrong, the URL doesn't change but the check passes.
- Better post-condition: presence of a known logged-in element on the destination page (the user's avatar, the "My Movies" link, etc.).

**N-8 · S2 · `vuduupdatedbyOpenAI:64` — `--no-sandbox` is set unconditionally.**
- `--no-sandbox` disables Chrome's process sandbox, which is a security feature. It's *required* when running as root in a container; it's *unnecessary* on a typical desktop. Make it conditional or document why.

**N-9 · S2 · `vuduupdatedbyOpenAI:174` — `--pass` is a CLI argument name.**
- `pass` is a Python reserved word. argparse handles this via `dest="password"` (which is what's done — line 174 uses `dest="password"`). Functionally fine. But `--password` is conventional. Cosmetic.

**N-10 · S1 · No password masking when user passes via CLI.**
- `python vuduupdatedbyOpenAI --pass mypassword …` appears in `ps aux` and shell history. The env-var path is safer. README/CLAUDE should explicitly say "prefer env vars to CLI for password".

**N-11 · S2 · `vudu.py:11-12` (legacy file) — `USERNAME = "example@gmail.com"` and `PASSWORD = "example"`.**
- These are placeholder values, but they're in committed source. If a user follows the README of the *old* file, they need to edit and save with real credentials. If they git-commit by accident, real password lands in git history. The canonical file fixed this via env vars.
- Either: delete `vudu.py` entirely (CLAUDE.md says it's deprecated), or make the placeholder string `os.getenv("VUDU_USER", "set-me")` so the placeholder doesn't tempt anyone.

**N-12 · S2 · Three variants on disk** (`vudu.py`, `vuduupdatedbyclaude.py`, `vuduupdatedbyOpenAI`) **with no automated check** that the canonical is up to date.
- The CLAUDE.md "Code Evolution Timeline" maps each commit to a file; clear history. But anyone (including future-you) could "improve" the legacy file thinking it's the active one.
- Recommend: delete the deprecated ones (per CLAUDE.md guidance) and rename `vuduupdatedbyOpenAI` → `vudu_scraper.py`.

**N-13 · S1 · `vuduupdatedbyclaude.py` (not deeply reviewed here)**: per CLAUDE.md, has the same hardcoded chromedriver path issue, and isn't being actively maintained. Triple-verify nothing depends on it (the README only describes the original `vudu.py`).

**N-14 · S2 · `vudu.py:51` — `for _ in range(1, 22):` then inside `for _ in range(1, 25):`.**
- Magic numbers (22, 25). The 22 is "scroll passes"; the 25 is "PAGE_DOWN keystrokes per pass". Total = 550 PAGE_DOWNs. For a small library, way too many; for a large library, possibly not enough.
- Legacy file; lower priority. The canonical implementation got this right via the `idle_rounds_limit` heuristic.

**N-15 · S2 · `README.md` does not describe the canonical `vuduupdatedbyOpenAI` interface.**
- README describes the original `vudu.py` flow (asks for username/password interactively, asks for CSV filename, etc.). The canonical implementation uses argparse + env vars + a default `out_csv`. Documentation drift.

---

## 3. Security Findings

**SEC-1 · LOW · No `LICENSE`** (same as other repos).

**SEC-2 · MEDIUM · `vudu.py:11-12` carries placeholder credentials in committed source** (N-11).
Not a real leak (placeholder values), but invites users to edit and commit real values.

**SEC-3 · LOW · `--pass mypassword` leaks credentials via shell history / `ps`** (N-10).

**SEC-4 · LOW · No CAPTCHA / 2FA handling.**
If Vudu/Fandango at Home enable login CAPTCHA or step-up auth, the script silently fails. CLAUDE.md mentions this in troubleshooting; documentation only.

**SEC-5 · MEDIUM · Selenium `--disable-blink-features=AutomationControlled`** is set (line 65).
This is an anti-detection feature to make the browser look less like an automated tool. It works against the site's TOS in the same gray area as `literotica` scraping — Vudu/Fandango's TOS likely prohibits automated access. Documented as personal-use; same legal posture as the other scrapers in the portfolio.

**SEC-6 · LOW · Two CSV output files in `vudu.py` (line 78: `"Example2.csv"`).**
- Hardcoded filename. If the user re-runs the script without changing the line, the previous output is overwritten without warning.
- Already addressed in the canonical implementation via `--out`.

---

## 4. Documentation Issues

**DOC-1 · `README.md` describes the legacy `vudu.py` interactive interface**, not the canonical `vuduupdatedbyOpenAI` argparse interface (N-15).
- Rewrite to describe the env-var + argparse flow, with the legacy noted as "historical".

**DOC-2 · `CLAUDE.md` is comprehensive** (1,000+ lines), the most detailed in the portfolio.
- Slight concern: at this length, it's a maintenance burden of its own. The "Code Evolution Timeline" + "Common Tasks for AI Assistants" sections are valuable; the "Troubleshooting Guide" / "Future Enhancement Ideas" sections could be split out.
- "Last Updated: 2025-11-19" — pre-dates the rebrand (N-1).

**DOC-3 · No `LICENSE`** (SEC-1).

**DOC-4 · No requirements file** (N-3).

**DOC-5 · CLAUDE.md "Repository:" link points to `https://github.com/ZacharyRW/VuduList`** — verify that resolves; otherwise update.

**DOC-6 · CLAUDE.md doesn't acknowledge the rebrand.**
"Vudu" is the only name used throughout. As soon as the official URLs redirect-and-die, the docs are stale.

---

## 5. Dependency & Version Audit

| Package | Used | Declared | Latest | Action |
|---|---|---|---|---|
| `selenium` | `from selenium import …` | not declared (CLAUDE.md says `>=4.0.0`) | 4.25.x | Add `requirements.txt`; pin to `>=4.20,<5` |
| `webdriver-manager` | optional import | not declared (CLAUDE.md says `>=3.8.0`) | 4.0.x | Optional dep; document |
| `chromedriver` (native) | runtime | system-managed | matches Chrome | OK |
| `Chrome` (native) | runtime | system-managed | latest | OK |

**No CVEs surfaced** for either Python dep.

**DEP-1**: Add `requirements.txt`. Add a `requirements-optional.txt` for `webdriver-manager`.

---

## 6. Static Analysis Output

- `py_compile` on `vudu.py` and `vuduupdatedbyclaude.py`: clean.
- `vuduupdatedbyOpenAI` has no `.py` extension; renamed via copy to check: clean.
- AST scan for unused imports: clean.

---

## 7. Test Coverage & CI

**Zero tests. Zero CI.** Same scope question as the other scraper-style tools — but at the project's current maturity, an integration test against a fixture HTML page (saved snapshot of Vudu's current My Movies page) would catch selector regressions early.

### CI-1 — Add minimal GitHub Actions workflow: `ruff check`, `mypy` against the canonical file, `python -c "import vuduupdatedbyOpenAI"` smoke import.
### CI-2 — Snapshot-based test: capture an HTML page, run selectors against it, assert title count and shape.

---

## 8. Performance / Resource Notes

- The scrolling loop in `vuduupdatedbyOpenAI` is the main runtime determinant. Already capped at `--max-min 8` (default) and `--idle-rounds 3`.
- `time.sleep(0.2) * 5` polling pattern (N-5) is the main inefficiency.
- The `seen: Set[str]` deduplicates correctly inline (linear over title count). Fine.

---

## 9. Cleanup / Tech-Debt

- **Two deprecated variants** kept around (N-12).
- **Missing `.py` extension** on canonical (N-2).
- **No requirements.txt** (N-3).
- **Stale README** (N-15).
- **Rebrand acknowledgment missing** (N-1).
- **Hardcoded placeholder password** in legacy (N-11).
- **No LICENSE** (SEC-1).

---

## 10. Ideas — Additions (in scope)

**ADD-1 · S — Execute the CLAUDE.md-recommended cleanup.**
- `git rm vudu.py vuduupdatedbyclaude.py` (or move to `archive/`).
- `git mv vuduupdatedbyOpenAI vudu_scraper.py`.
- Update README, CLAUDE.md, branch / file references.
- Add `requirements.txt`.
- Add `LICENSE`.
- *First step*: one PR titled "Repo cleanup: archive deprecated variants, fix canonical extension".

**ADD-2 · S — Re-validate against current Fandango at Home site.**
- Open the current site in the user's browser; capture login URL, library URL, login form selectors, library item selectors.
- Update the constants in the canonical file.
- Document the verification date in CLAUDE.md.

**ADD-3 · M — Diff mode (`--diff against previous-run.csv`).**
- Already on CLAUDE.md's "Future Enhancement Ideas" list. Detects newly-purchased and (more usefully) movies that disappear from your library (Vudu has historically had license-expiration issues).
- *First step*: load previous CSV, sort both, print added/removed.

**ADD-4 · M — Metadata enrichment.**
- For each title, hit TMDb / OMDB API for release year, genre, runtime. Output as enriched CSV (title, year, genre, runtime).
- Closes the loop for cataloging-as-personal-knowledge-base.

**ADD-5 · S — Headless mode is the default; require explicit `--no-headless` to disable.**
- Most users running it locally don't need to watch the browser. Flipping the default reduces accidental "I left this open for an hour" cases.

**ADD-6 · S — Replace fixed 5×0.2s wait with `WebDriverWait` on count-changed (N-5).**

---

## 11. Ideas — New Directions (out of scope but interesting)

**DIR-1 · Generalize into a "owned media catalog" toolkit.**
- *Pitch*: VuduList is one of N "what movies do I own" sources. Same shape applies to Apple TV purchases, Google Play movies, Amazon Prime Video, Microsoft Store. A pluggable scraper architecture (per-provider plugin) → unified library output → de-dup across providers ("I own this movie 3 times").
- *What changes*: extract Selenium driver setup, login pattern, scroll-and-collect heuristics into a base class; each provider implements `LOGIN_URL`, `LIBRARY_URL`, and selectors.
- *Why it's worth considering*: many users have movies scattered across multiple ecosystems. A consolidated CSV is genuinely useful for migration / inventory.

**DIR-2 · Plug into the homelab — Stash / Plex import.**
- *Pitch*: instead of a CSV, output a metadata file (NFO XML, JSON-LD) that Plex / Stash / Sonarr can consume. The output becomes "rentable" inventory — pair with the user's media-stack to verify "I own this; download from a different source rather than re-purchasing".
- *What changes*: output format becomes per-target; the rest stays the same.
- *Why it's worth considering*: aligns with the user's existing media-stack (`hexos-homepage-config`) and the `sqlite-renamer` work. Closes the "what do I own vs what's on disk" gap.

---

## 12. Recommended Next Actions

### Must-fix (correctness / functionality)

1. **N-1 / ADD-2** — Verify the script still works against Fandango at Home; update URLs and selectors. Without this, the rest of the audit is irrelevant.
2. **N-2 / ADD-1** — Rename `vuduupdatedbyOpenAI` → `vudu_scraper.py`.
3. **N-6** — Fix the `os.makedirs("")` bug when out_path has no directory component.

### Should-fix (DX / hygiene)

4. **N-3 / DEP-1** — Add `requirements.txt`.
5. **N-12 / ADD-1** — Delete (or archive) `vudu.py` and `vuduupdatedbyclaude.py`.
6. **N-15 / DOC-1** — Rewrite README around the canonical CLI.
7. **N-11** — Either delete `vudu.py` or replace placeholder password with `os.getenv()`.
8. **N-7** — Strengthen login post-condition check.
9. **N-5 / ADD-6** — Replace fixed sleeps with `WebDriverWait`.
10. **SEC-1 / DOC-3** — Add `LICENSE`.
11. **CI-1** — Minimal GitHub Actions: `ruff` + smoke-import.

### Nice-to-have (cleanup / ideas)

12. **N-8** — Make `--no-sandbox` conditional.
13. **N-10** — Document "prefer env vars over CLI for password".
14. **N-4** — Fail-fast assertion that selector finds elements after navigation.
15. **ADD-3** — Diff mode against previous CSV.
16. **ADD-4** — TMDB / OMDB enrichment.
17. **ADD-5** — Flip default to headless.
18. **DIR-1 / DIR-2** — Future-form directions.

---

## Appendix: How this audit was produced

- Read `README.md`, `CLAUDE.md` (full), `vudu.py`, `vuduupdatedbyclaude.py` (file list only, not content), `vuduupdatedbyOpenAI` (full) in full.
- Inspected `.gitignore`, `git log -10`, file structure.
- No code modifications were made.
