# Fandango at Home Library Exporter Agent Guide

## Project state

Fandango at Home Library Exporter is a local, personal-use Python/Selenium command-line tool intended to export movie titles from a Fandango at Home (formerly Vudu) library to CSV. It is a maintenance-stage prototype, not a release-ready product.

`ANALYSIS.md` is the current evidence record and `ROADMAP.md` is the canonical execution tracker. Verify both against the current code and Git state before treating an item as open.

## Current implementation

- `vuduupdatedbyOpenAI` is the current reference implementation, although its extensionless name is a known maintenance issue (`ARCH-001`).
- `vudu.py` and `vuduupdatedbyclaude.py` are legacy variants. Do not add features or fixes to them; migrate deliberately to one conventional module only after the site contract is validated.
- The project has no dependency manifest, automated tests, or CI. Selenium is required at runtime; `webdriver-manager` is optional when a local ChromeDriver path is supplied.

## Working rules

- Do not commit credentials, cookies, personal CSV exports, or authenticated page captures.
- Treat Fandango at Home URLs, selectors, authentication behavior, MFA, and scrolling as an external contract. Do not change them based on guesswork; authorize and document a sanitized live check first (`REL-001`).
- Do not add automation intended to bypass MFA, CAPTCHA, bot controls, or third-party access restrictions.
- Preserve history and uncommitted user work. Do not delete legacy branches or source variants without explicit approval and an identified target.
- Keep verified defects, maintenance, committed plan, optional enhancements, and speculative directions distinct in planning documents.

## Verification

- The existing scripts can be syntax-checked with `python3 -m py_compile vudu.py vuduupdatedbyclaude.py vuduupdatedbyOpenAI`.
- Do not represent `--help`, browser startup, login, or live scraping as verified until declared dependencies are installed and a controlled account check is explicitly authorized.
- When documentation changes, keep `ANALYSIS.md`, `ROADMAP.md`, and this guide consistent with the code and Git state.
