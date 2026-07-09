# CLAUDE.md - AI Assistant Guide for VuduList

## Project Overview

**VuduList** is a Python-based web scraping tool that automates the extraction of movie titles from a user's Vudu account. The program uses Selenium WebDriver to:
1. Log into a Vudu account
2. Navigate to the "My Movies" section
3. Scrape movie titles from dynamically-loaded content
4. Export the data to a CSV file

**Primary Use Case:** Cataloging personal movie collections for backup, organization, or migration purposes.

---

## Repository Structure

```
VuduList/
├── .git/                          # Git version control
├── .gitignore                     # Python-specific gitignore
├── README.md                      # User-facing documentation
├── CLAUDE.md                      # This file - AI assistant guide
├── vudu.py                        # Original implementation (deprecated)
├── vuduupdatedbyclaude.py         # Claude-updated version (intermediate)
└── vuduupdatedbyOpenAI            # OpenAI-updated version (most recent)
```

### File Purposes

#### 1. `vudu.py` (Original - DEPRECATED)
- **Status:** Legacy code, not maintained
- **Issues:**
  - Uses deprecated Selenium methods (`find_element_by_*`)
  - Hardcoded credentials (security risk)
  - Hardcoded Windows chromedriver path
  - Fixed sleep times instead of WebDriverWait
  - No error handling
  - Duplicate detection happens post-collection (inefficient)
- **Line Count:** 92 lines
- **Do NOT use this as reference for new code**

#### 2. `vuduupdatedbyclaude.py` (Intermediate)
- **Status:** Improved but superseded
- **Improvements over original:**
  - Modern Selenium 4 syntax (`By.NAME`, `By.CSS_SELECTOR`)
  - WebDriverWait for dynamic elements
  - Duplicate prevention during collection
  - Try-except-finally for error handling
  - Browser cleanup in finally block
  - UTF-8 encoding for CSV output
- **Remaining issues:**
  - Still has hardcoded credentials
  - Still has hardcoded chromedriver path
  - Limited configuration options
- **Line Count:** 87 lines

#### 3. `vuduupdatedbyOpenAI` (Current Reference Implementation)
- **Status:** Current reference implementation in this repository
- **Major improvements:**
  - **Modular architecture** with dataclasses and type hints
  - **CLI argument parsing** with argparse
  - **Environment variable support** (VUDU_USER, VUDU_PASS, CHROMEDRIVER)
  - **Optional webdriver-manager** integration (auto-downloads chromedriver)
  - **Configurable parameters** (timeouts, idle rounds, max duration)
  - **Robust scrolling algorithm** with stagnation detection
  - **Comprehensive error handling** with specific exception types
  - **Headless mode support**
  - **Detailed inline documentation** with "Why:" comments
  - **Exit codes** for shell integration
- **Line Count:** 229 lines
- **Architecture:** Function-based with clear separation of concerns
- **This should be the reference implementation**

---

## Code Evolution Timeline

Based on git history:

```
4e4c199 Initial commit
  └─> vudu.py created (original version)

16e2df1 Update vudu.py
  └─> Minor improvements

132ed3a Master Update
  └─> README improvements

bc7bc8e Update README.md
  └─> Documentation updates

bdc664b Fixed code formatting (including tabs and comments)
  └─> Code cleanup

dcddcb0 Updated the code using Claude AI
  └─> vuduupdatedbyclaude.py created (Selenium 4 syntax, error handling)

437b49b Rename vuduupdatedbyclaude to vuduupdatedbyclaude.py
  └─> File extension fix

48a0761 Updated Vudu.py using OpenAI
  └─> vuduupdatedbyOpenAI created (production-grade refactor)
```

---

## Technical Stack

### Core Dependencies
- **Python:** 3.7+ (uses type hints, dataclasses)
- **Selenium:** 4.x (uses new `By` syntax)
- **Chrome/Chromium:** Required for webdriver
- **ChromeDriver:** Matching Chrome version (auto-managed if webdriver-manager installed)

### Optional Dependencies
- **webdriver-manager:** Auto-downloads and manages chromedriver versions

### Standard Library
- `argparse` - CLI argument parsing
- `csv` - CSV file writing
- `os` - Environment variables and path operations
- `sys` - Exit codes
- `time` - Sleep and timing
- `dataclasses` - Configuration objects
- `typing` - Type annotations

---

## Key Conventions & Patterns

### 1. Code Style
- **Indentation:** 4 spaces (PEP 8 compliant)
- **Imports:** Grouped by standard library, third-party, local
- **Type hints:** Extensive use in the current reference implementation (`vuduupdatedbyOpenAI`)
- **Docstrings:** Present on module and key functions
- **Comments:** Inline with "Why:" explanations for non-obvious decisions

### 2. Naming Conventions
- **Functions:** `snake_case` (e.g., `build_driver`, `infinite_scroll_collect`)
- **Constants:** `UPPER_SNAKE_CASE` (e.g., `LOGIN_URL`, `EMAIL_NAME`)
- **Variables:** `snake_case`
- **Classes:** `PascalCase` (e.g., `Config` dataclass)

### 3. Architecture Patterns
- **Separation of concerns:** Each function has single responsibility
- **Configuration objects:** Use dataclasses for grouped settings
- **Explicit is better than implicit:** No magic numbers, use named constants
- **Fail fast:** Validate inputs early (credentials check at startup)

### 4. Error Handling
- **Specific exceptions:** Catch `TimeoutException`, `WebDriverException` separately
- **Always cleanup:** Use try-finally for browser.quit()
- **User-friendly messages:** Print clear errors to stderr
- **Exit codes:** 0 for success, 1 for errors, 2 for invalid args

### 5. Selenium Best Practices
- **WebDriverWait over sleep:** Use explicit waits for reliability
- **Modern selectors:** Use `By.CSS_SELECTOR`, `By.NAME` (not `find_element_by_*`)
- **Page load strategy:** Set to "normal" for heavy pages
- **Headless options:** Use `--headless=new` for modern Chrome
- **Anti-detection:** Set `--disable-blink-features=AutomationControlled`

---

## Security Considerations

### Critical Security Issues

1. **NEVER commit credentials**
   - Original files contain example credentials (safe)
   - When testing, use environment variables or CLI args
   - Add `*.env` to `.gitignore` (already present)

2. **Credential handling precedence:**
   ```python
   # Correct order (from vuduupdatedbyOpenAI):
   1. CLI arguments (--user, --pass)
   2. Environment variables (VUDU_USER, VUDU_PASS)
   3. Exit if neither provided (never hardcode)
   ```

3. **CSV output security:**
   - Files created in current directory by default
   - UTF-8 encoding prevents data corruption
   - No sensitive data logged (credentials never printed)

4. **Browser security:**
   - ChromeDriver path can be set via `--chromedriver` or `CHROMEDRIVER` env var
   - Headless mode prevents screen observation
   - Always quit browser to prevent orphaned processes

---

## Development Workflows

### For AI Assistants Working on This Repo

#### Adding New Features
1. **Always use `vuduupdatedbyOpenAI` as the base**
2. **Maintain the modular structure:**
   - Add new CLI args to `parse_args()`
   - Add new config fields to `Config` dataclass
   - Create focused helper functions
3. **Follow existing patterns:**
   - Use type hints
   - Add "Why:" comments for non-obvious code
   - Handle errors explicitly
4. **Test scenarios to consider:**
   - Empty movie collections
   - Network timeouts
   - Changed Vudu UI selectors
   - Invalid credentials

#### Bug Fixes
1. **Identify which file is affected:**
   - If in `vudu.py` or `vuduupdatedbyclaude.py`: Consider migrating fix to `vuduupdatedbyOpenAI`
   - If in `vuduupdatedbyOpenAI`: Fix in place
2. **Common issues:**
   - Selector changes (Vudu UI updates)
   - Timeout too short (increase default wait_timeout)
   - Infinite loop (check idle_rounds_limit logic)

#### Refactoring Guidelines
1. **Maintain backward compatibility** in CLI interface
2. **Add deprecation warnings** before removing features
3. **Update docstrings** if function signatures change
4. **Preserve exit code contract** (0 = success, 1 = error, 2 = invalid input)

#### Testing Checklist
Since there are no automated tests, manually verify:
- [ ] Runs with `--headless` flag
- [ ] Respects `VUDU_USER` and `VUDU_PASS` env vars
- [ ] Creates CSV with correct encoding (UTF-8)
- [ ] Handles invalid credentials gracefully
- [ ] Stops scrolling when no new items load
- [ ] Respects `--max-min` timeout
- [ ] Works with both local chromedriver and webdriver-manager

---

## Common Tasks for AI Assistants

### Task 1: Update Selenium Selectors (UI Changes)
**When:** Vudu changes their website structure

**Where to look:**
- `LOGIN_URL` (line 34 in vuduupdatedbyOpenAI)
- `MY_MOVIES_URL` (line 35)
- `EMAIL_NAME`, `PASSWORD_NAME` (lines 37-38)
- `SUBMIT_SELECTOR` (line 39)
- `MOVIE_IMG_SELECTOR` (line 41)

**How to debug:**
1. Run with `--headless` flag disabled
2. Add screenshots: `driver.save_screenshot('debug.png')`
3. Print element counts: `print(f"Found {len(elements)} elements")`
4. Check browser console for JavaScript errors

### Task 2: Add New Export Formats
**Current:** CSV only

**To add JSON/Excel/etc:**
1. Add to `Config` dataclass: `export_format: str`
2. Add CLI arg: `--format {csv,json,xlsx}`
3. Create new writer function: `write_json(rows, path)`
4. Update `main()` to dispatch based on format

### Task 3: Improve Scrolling Algorithm
**Current logic:** (lines 123-160 in vuduupdatedbyOpenAI)
- Scrolls to bottom
- Waits 1 second (5 × 0.2s)
- Tracks consecutive "no new items" rounds
- Stops after 3 idle rounds or max_minutes

**Potential improvements:**
- Adaptive wait times based on network speed
- Track viewport position to detect end-of-page
- Use Intersection Observer API if Vudu supports it

### Task 4: Add Pagination Support
If Vudu adds pagination instead of infinite scroll:
1. Detect pagination buttons
2. Click "next" until disabled
3. Aggregate results across pages

### Task 5: Add TV Shows / Other Content Types
Currently hardcoded to `#my_vudu/my_movies`

**To extend:**
1. Add `--content-type {movies,tv,all}` CLI arg
2. Map to URL fragments
3. Adjust selectors for different content types
4. Consider separate CSV files or multi-column output

---

## Git Workflow

### Branch Naming Convention
- Feature branches have historically used names like `claude/claude-md-<hash>-<session-id>`.
- Do not hardcode a current branch in this file. Before making branch-dependent claims or push decisions, run:

```bash
git branch --show-current
git rev-parse --short HEAD
git status --short
```

### Commit Message Style
Based on recent commits:
- Imperative mood: "Update", "Fix", "Add"
- Reference tool: "Updated the code using Claude AI"
- Descriptive: "Fixed code formatting (including tabs and comments)"

### When to Commit
1. After implementing a complete feature
2. After fixing a bug (with descriptive message)
3. Before making risky refactors (checkpoint)
4. When user explicitly requests

### Push Strategy
- Push only when the user explicitly requests it.
- Push from the branch confirmed at runtime; do not assume a historical `claude/*` branch is still current.
- Never force push without user permission.

---

## File Cleanup Recommendations

### For Repository Maintainer
Consider the following cleanup (discuss with user first):

1. **Archive deprecated files:**
   ```bash
   mkdir archive/
   git mv vudu.py archive/
   git mv vuduupdatedbyclaude.py archive/
   ```

2. **Rename production file:**
   ```bash
   git mv vuduupdatedbyOpenAI vudu_scraper.py
   ```

3. **Add dependencies file:**
   ```bash
   # Create requirements.txt:
   selenium>=4.0.0
   webdriver-manager>=3.8.0  # optional
   ```

4. **Add example .env file:**
   ```bash
   # Create .env.example:
   VUDU_USER=your_email@example.com
   VUDU_PASS=your_password
   CHROMEDRIVER=/path/to/chromedriver  # optional
   ```

---

## Quick Reference

### Running the Scraper

**With environment variables:**
```bash
export VUDU_USER="email@example.com"
export VUDU_PASS="password"
python vuduupdatedbyOpenAI --headless --out my_movies.csv
```

**With CLI args:**
```bash
python vuduupdatedbyOpenAI \
  --user "email@example.com" \
  --pass "password" \
  --out movies.csv \
  --headless \
  --max-min 10 \
  --idle-rounds 5
```

**With local chromedriver:**
```bash
python vuduupdatedbyOpenAI \
  --user "$VUDU_USER" \
  --pass "$VUDU_PASS" \
  --chromedriver /usr/local/bin/chromedriver
```

### Key Configuration Options

| Flag | Default | Description |
|------|---------|-------------|
| `--user` | `$VUDU_USER` | Vudu account email |
| `--pass` | `$VUDU_PASS` | Vudu account password |
| `--out` | `vudu_movies.csv` | Output CSV file path |
| `--headless` | `False` | Run Chrome in headless mode |
| `--pageload` | `60` | Page load timeout (seconds) |
| `--wait` | `30` | Element wait timeout (seconds) |
| `--idle-rounds` | `3` | Consecutive rounds with no new items before stopping |
| `--max-min` | `8` | Hard stop after N minutes |
| `--chromedriver` | `$CHROMEDRIVER` | Path to chromedriver binary |

### Selector Reference

| Element | Selector | Purpose |
|---------|----------|---------|
| Email field | `name="email"` | Login form |
| Password field | `name="password"` | Login form |
| Login button | `.custom-button` | Submit login |
| Movie images | `.border .gwt-Image` | Extract titles from alt attribute |

---

## Troubleshooting Guide

### Problem: "Login likely failed or timed out"
**Causes:**
- Invalid credentials
- Two-factor authentication enabled
- Vudu terms of service update requiring acceptance
- IP rate limiting

**Solutions:**
1. Verify credentials manually in browser
2. Disable 2FA or handle TOTP in code
3. Accept ToS manually first
4. Increase `--wait` timeout

### Problem: "No titles were found"
**Causes:**
- UI selector changed
- Network too slow (images not loaded)
- Account has no movies

**Solutions:**
1. Run without `--headless` to visually inspect
2. Increase `--idle-rounds` and `--max-min`
3. Check browser console for errors
4. Update `MOVIE_IMG_SELECTOR`

### Problem: Script hangs indefinitely
**Causes:**
- Infinite loop in scroll logic
- Network stall
- ChromeDriver crash

**Solutions:**
1. Check `--max-min` is set (default 8 minutes)
2. Monitor network activity
3. Check for zombie chromedriver processes
4. Add debug print statements in `infinite_scroll_collect()`

### Problem: Duplicate movies in CSV
**Causes:**
- Set deduplication failed
- Same movie listed multiple times on Vudu (different versions)

**Solutions:**
1. Check `seen` set in `infinite_scroll_collect()` (line 128)
2. Vudu may list "HD" and "UHD" separately (expected behavior)
3. Post-process CSV: `sort -u movies.csv > unique.csv`

---

## Future Enhancement Ideas

### High Priority
1. **Add automated tests** (unittest or pytest)
2. **Create requirements.txt** for dependency management
3. **Add logging** (replace print statements with logging module)
4. **Support TV shows** (separate or combined with movies)
5. **Resume capability** (save progress, resume on crash)

### Medium Priority
6. **Rate limiting** (respect Vudu's servers)
7. **Retry logic** (network transient failures)
8. **Progress bar** (tqdm integration)
9. **Multiple export formats** (JSON, Excel, SQLite)
10. **Proxy support** (for privacy-conscious users)

### Low Priority
11. **GUI version** (Tkinter or web interface)
12. **Docker container** (easier deployment)
13. **Scheduled runs** (cron-like functionality)
14. **Diff mode** (detect new/removed movies since last run)
15. **Metadata enrichment** (IMDB ratings, release years from API)

---

## AI Assistant Reminders

### Before Making Changes
- [ ] Read this CLAUDE.md file completely
- [ ] Use `vuduupdatedbyOpenAI` as reference (not older versions)
- [ ] Check if feature already exists (read full file)
- [ ] Consider security implications (credentials, data exposure)

### During Development
- [ ] Maintain type hints and docstrings
- [ ] Add "Why:" comments for non-obvious code
- [ ] Use explicit waits (not time.sleep) for Selenium
- [ ] Handle errors specifically (not bare except)
- [ ] Test with `--headless` flag

### Before Committing
- [ ] Remove any hardcoded credentials
- [ ] Update docstrings if signatures changed
- [ ] Check exit codes still work (0/1/2)
- [ ] Verify backward compatibility (CLI args)
- [ ] Test manually (no automated tests exist)

### Communication Style
- Be clear about which file you're modifying
- Reference line numbers when discussing code
- Explain tradeoffs when multiple approaches exist
- Ask for clarification on security-sensitive changes

---

## Document Maintenance

**Last Updated:** 2025-11-19
**Author:** Claude (Anthropic AI)
**Repository:** https://github.com/ZacharyRW/VuduList
**Last verified:** 2026-07-09 against commit `dcd0ad8`. Re-run `git branch --show-current`, `git rev-parse --short HEAD`, and `git status --short` before making branch, release, or working-tree claims.

**Update Triggers:**
- Major code refactors
- New dependencies added
- Vudu UI/API changes requiring selector updates
- New features or CLI arguments
- Security vulnerabilities discovered
- Branch/repo structure changes

---

## Contact & Support

For issues or feature requests, users should:
1. Check this CLAUDE.md for common solutions
2. Review README.md for basic usage
3. Open GitHub issue at repository URL

AI assistants should:
1. Consult this guide before making assumptions
2. Ask user for clarification on ambiguous requirements
3. Suggest updates to this file when patterns change
