# Project Analysis

Review snapshot: 2026-07-14 local time. Code was reviewed at commit 4a8b2365f82743b1a50bb357e85af008b5bbf51d on remote main. The originally inspected commit fd7fde0916bf81966b4aee20ea4dc9e3df0ec1ef has the same Git tree. This document is the evidence-based source for current planning.

## Executive Summary

Fandango at Home Library Exporter is a small local Python command-line tool intended to sign in to a personal Fandango at Home (formerly Vudu) account, collect movie titles from the browser-rendered library, and export them as CSV. Its intended user is an individual who wants a personal ownership inventory, not a hosted or multi-user audience.

The repository is best described as a maintenance-stage prototype rather than a release-ready utility. The current implementation has several good foundations: environment-variable credentials, explicit Selenium waits, bounded scrolling, clear exit codes, configurable driver selection, CSV output, and browser cleanup. The repository as a whole is held back by a missing dependency manifest, no tests or CI, stale public documentation, three competing executable implementations, and a fragile dependency on a third-party consumer website whose current authenticated DOM contract has not been verified.

The highest-priority direction is stabilization, not feature expansion: first verify the live account flow and selectors with an authorized test account; then create one named canonical entry point, declarative dependencies, automated tests, and a minimal CI check. Two confirmed correctness defects should be fixed in that work: an empty but valid library currently becomes a timeout, and the login success predicate is not a meaningful post-login assertion.

| Area | Assessment |
| --- | --- |
| Current health | Needs stabilization before public promotion or feature work |
| Strongest code | The extensionless current implementation, vuduupdatedbyOpenAI |
| Largest operational risk | Vudu/Fandango at Home URL, login, and selector assumptions are unverified external contracts |
| Largest repository risk | A clean user cannot install and run the tool from repository metadata |
| Security result | No validated reportable vulnerability in the local single-user threat model; two hardening/privacy improvements are warranted |
| Recommended direction | Consolidate, validate live behavior, establish reproducible setup and tests, then add product features |

## Project Overview

### Purpose and audience

The implemented purpose is narrower than some repository text implies: export a personal movie library to a one-column CSV. The GitHub description mentions movies and TV shows, but the current code only navigates the movies collection. The user is expected to run Chrome locally and provide their own credentials.

### Verified features

- Reads credentials from VUDU_USER and VUDU_PASS, or from --user and --pass command-line arguments.
- Creates a local Chrome Selenium session using a local ChromeDriver path or optional webdriver-manager.
- Visits a legacy Vudu login URL and a My Movies URL.
- Collects non-empty alt attributes from image elements selected by .border .gwt-Image.
- Scrolls until a configurable idle-round count or a time limit is reached.
- Deduplicates and alphabetically sorts titles.
- Writes UTF-8 CSV, one title per row, to a configurable output path.
- Returns distinct process outcomes for missing credentials, expected browser failures, and successful completion.

### Technology stack and architecture

| Layer | Evidence |
| --- | --- |
| Language | Python 3; the local baseline used Python 3.14.6 |
| Browser automation | Selenium 4-style APIs in the current script; obsolete Selenium APIs in vudu.py |
| Browser and driver | Chrome plus local ChromeDriver or optional webdriver-manager |
| Data output | Python csv module |
| Configuration | argparse, environment variables, frozen Config dataclass |
| External service | Vudu/Fandango at Home rendered website; no published API is used |
| Hosting/deployment | None tracked; this is a local CLI, not a deployed service |

Data flow:

1. The local user supplies credentials, output path, timeouts, and browser options through environment variables or CLI arguments.
2. parse_args builds Config and build_driver starts Chrome.
3. login fills named form elements and submits the form.
4. navigate_to_my_movies loads the library page; collect_visible_titles reads image alt text as the title source.
5. infinite_scroll_collect deduplicates titles in memory; write_csv overwrites the selected output file.

The current architecture is a single script with functions that are reasonably separable, but it has no package boundary, importable module name, fixtures, test seam around site selectors, or release boundary.

### Current maturity and lifecycle

| Lifecycle concern | Observed state |
| --- | --- |
| Build | Python source compiles, but there is no package/build configuration |
| Installation | No requirements.txt, pyproject.toml, lockfile, or documented supported Python version |
| Testing | No tracked tests; unittest discovery finds none |
| Linting/type checking | No configured tool or tracked configuration |
| CI/CD | No .github workflow is tracked and no GitHub Actions runs were found |
| Deployment | Not applicable; local browser CLI |
| Releases | No tags or releases were found |

## Repository Structure

| Path | Role | Status |
| --- | --- | --- |
| vuduupdatedbyOpenAI | Current reference implementation according to CLAUDE.md; extensionless Python executable | Keep temporarily; rename and package in a stabilization change |
| vudu.py | Original 2017 Selenium script using static credentials, static driver path, magic scroll counts, and deprecated Selenium APIs | Historical legacy code; archive or remove after a migration decision |
| vuduupdatedbyclaude.py | Intermediate modernization with static ChromeDriver path | Historical legacy code; archive or remove after a migration decision |
| README.md | Public quick-start document | Materially outdated; describes legacy interactive behavior rather than the current CLI |
| CLAUDE.md | Extensive AI/developer handoff and planning document | Valuable source material but stale and oversized for the current project |
| AGENTS.md | Repository-default guidance | Keep concise and aligned with the current code, analysis, and roadmap |
| .gitignore | Ignores environment files, virtual environments, IDE state, and bytecode | Reasonable base; does not ignore personal CSV exports |
| .github/ | Not present | Missing CI, templates, dependency automation, and contribution metadata |
| tests/ | Not present | Missing unit, integration, and selector regression coverage |
| Dependency/project files | Not present | Missing reproducible installation and tooling definition |

## Validation Results

Validation was intentionally recorded before any repair. No dependency installation was attempted because the repository has no authoritative dependency manifest or pinned version policy; selecting and installing arbitrary current packages would have changed the baseline without a defined project contract.

| Check | Command | Result |
| --- | --- | --- |
| Interpreter | python3 --version | Passed: Python 3.14.6 |
| Syntax compilation | python3 -m py_compile vudu.py vuduupdatedbyclaude.py vuduupdatedbyOpenAI | Passed |
| Recursive compilation | python3 -m compileall vudu.py vuduupdatedbyclaude.py vuduupdatedbyOpenAI | Passed |
| Dependency presence | python3 -m pip show selenium webdriver-manager | Failed baseline: neither package is installed |
| Current CLI help | python3 vuduupdatedbyOpenAI --help | Failed before argument parsing: ModuleNotFoundError for selenium |
| Normal Python import | python3 -c import vuduupdatedbyOpenAI | Failed: the extensionless filename is not importable by normal Python module discovery |
| Test discovery | python3 -m unittest discover -v | Failed with exit 5: Ran 0 tests; NO TESTS RAN |
| Lint/type/format/security tools | ruff, mypy, pytest, black, pip-audit | Not installed and not configured |
| Output path claim from prior audit | Python evaluation of dirname(abspath(movies.csv)) | Passed: parent is the repository directory, not an empty path |
| Export privacy hygiene | git check-ignore -v vudu_movies.csv Example2.csv | Neither default nor legacy CSV export is ignored |
| Tree consistency | git diff --quiet fd7fde0 4a8b236 | Passed: the two commits have the same tree |
| External endpoint observation | Anonymous request to the configured My Movies URL | Redirected to a Fandango at Home browse page; this does not verify authenticated selectors |

The documented setup also fails as written:

- README.md says the program prompts for credentials and a CSV file name. The current script instead requires CLI arguments or environment variables and defaults to vudu_movies.csv.
- The current script imports Selenium before it can show --help, so basic help cannot run in a clean checkout.
- The current script's docstring names tools/vudu_scrape_my_movies.py, but that path does not exist.

Environmental limitations:

- No authorized Vudu/Fandango at Home account session was supplied; the live login, MFA, selector, and collection behavior were not exercised.
- The third-party page is a changing external UI, not a stable API.
- GitHub repository settings such as branch protection, rulesets, social preview, topics, website, funding, and Actions permissions were not exposed by the available access surface.

## Existing Issue Verification

### Sources searched

This review searched all tracked files, Git history, local and remote branch metadata, public GitHub metadata available through the connected service, closed issues and pull requests, and existing planning language. No source TODO, FIXME, HACK, BUG, XXX, disabled test, skipped test, placeholder implementation, or commented-out feature block was found outside the documents and legacy code described below.

### Reconciled prior findings

| Existing item | Source | Current status | Verification | Still relevant? | Recommended action |
| --- | --- | --- | --- | --- | --- |
| N-1: Vudu rebrand and stale URLs/selectors | Prior review | Partially confirmed | Configured My Movies URL redirected anonymously to Fandango at Home; no authenticated selector test was authorized | Yes | Perform an authorized live contract check before code changes or release |
| N-2: canonical file lacks .py extension | Prior review | Confirmed | vuduupdatedbyOpenAI is extensionless and normal Python import fails | Yes | Rename to a conventional module name during consolidation |
| N-3: no requirements file | Prior review | Confirmed | No requirements.txt, pyproject.toml, lockfile, or dependency metadata exists; Selenium is absent locally | Yes | Add a single authoritative dependency definition and supported-version policy |
| N-4: alt-only title extraction is fragile | Prior review | Partially confirmed | collect_visible_titles reads only image alt; live failure was not observed | Yes | Validate current DOM and add selector fallbacks plus fixture coverage |
| N-5: fixed polling sleep in scrolling | Prior review | Confirmed design risk | Current loop sleeps five times for 0.2 seconds after every scroll | Yes | Replace or augment with a count-change/ready-state wait and measure behavior |
| N-6: os.makedirs empty-parent bug | Prior review | Already fixed / false positive | Current code uses dirname(abspath(out_path)), which is non-empty for a bare file name | No | Remove from active backlog; retain only a test for output paths |
| N-7: login does not verify success | Prior review | Partially confirmed, with a confirmed weak predicate | The post-submit predicate only checks that the URL contains vudu.com, which is already true on the login URL | Yes | Wait for a post-authenticated library element or a known authenticated URL/state |
| N-8: unconditional --no-sandbox | Prior review | Confirmed hardening concern | Current driver options always add --no-sandbox | Yes | Remove on normal desktops or make an explicit documented compatibility option |
| N-9: --pass name is a reserved word concern | Prior review | Obsolete as a defect | argparse deliberately maps it to password; Python's reserved word does not cause a functional error | No | Prefer a clearer --password alias only as a compatibility UX improvement |
| N-10: password in CLI arguments | Prior review | Confirmed local privacy concern | --pass is accepted and documented; shell history and local process viewers may expose it | Yes | Prefer environment variable or secure prompt input; document the tradeoff |
| N-11: legacy placeholder credentials | Prior review | Partially confirmed | vudu.py contains example values, not a real secret; retaining an editable legacy script invites future accidental commits | Yes | Archive/remove legacy scripts and add a secret-scanning check once tooling exists |
| N-12: three executable variants | Prior review | Confirmed | All three scripts remain executable source candidates with inconsistent behavior | Yes | Establish one canonical entry point and archive the rest |
| N-13: legacy static ChromeDriver path | Prior review | Confirmed | Intermediate script retains a Windows-specific ChromeDriver path | Yes, only until legacy code is archived | Do not modernize the legacy copy; archive/remove it |
| N-14: legacy magic scrolling values | Prior review | Confirmed | vudu.py still uses fixed nested scroll counts | Yes, only until legacy code is archived | Do not tune legacy behavior; archive/remove it |
| N-15: README describes legacy interface | Prior review | Confirmed | README says credentials and CSV name are prompted; current script uses CLI/env configuration | Yes | Rewrite README after deciding the canonical name and install method |
| SEC-1: no license | Prior review | Resolved 2026-10-01 | No LICENSE was tracked | Yes — owner selected MIT on 2026-10-01 (explicit direction); LICENSE added in PR #9 | Done |
| SEC-4: CAPTCHA/2FA handling | Prior review | Unable to verify | No live login was run and external service behavior is outside repository control | Possibly | Document expected manual authentication behavior; do not treat absence as a security vulnerability |
| SEC-5: anti-detection flag may conflict with terms | Prior review | Risk requiring external review | --disable-blink-features=AutomationControlled is present; no terms source was verified | Yes | Review current service terms manually and remove unnecessary anti-detection behavior |
| SEC-6: legacy output overwrites | Prior review | Confirmed legacy behavior | vudu.py uses a fixed Example2.csv name; current script lets the caller choose but still overwrites selected output | Yes, as UX policy | Resolve through legacy archival and an intentional overwrite policy for the current tool |
| DOC-2: project guidance was too large/stale | Prior review | Resolved | Project-wide guidance now lives in concise AGENTS.md; CLAUDE.md is a Claude-specific pointer | No | Keep AGENTS.md current with code and planning documents |
| DOC-5: repository link validity | Prior review | Already fixed | GitHub repository resolves through the connected service | No | Remove from active backlog |
| DOC-6: no rebrand acknowledgment | Prior review | Partially confirmed | Docs use Vudu only; anonymous navigation shows Fandango at Home branding | Yes | Update naming and note that selectors require validation |
| CI-1: add lint/type/smoke CI | Prior review | Confirmed need | No workflow or tool configuration exists; proposed import check cannot work until filename/module issue is fixed | Yes | Add CI after packaging and tests are defined |
| CI-2: selector snapshot test | Prior review | Confirmed need | No fixtures or tests exist; third-party DOM is an external contract | Yes | Capture authorized, sanitized fixtures and test selectors without credentials |

### Reconciled additions and directions

| Existing item | Current status | Verification and recommended action |
| --- | --- | --- |
| ADD-1: archive legacy files, rename canonical file, add dependencies/license | Partially confirmed | Consolidation and dependency work are necessary; license choice made by owner (MIT, 2026-10-01; LICENSE added in PR #9). Do this after a live contract check, not as a blind rename. |
| ADD-2: revalidate current Fandango at Home site | Confirmed top priority | Anonymous redirect demonstrates branding change; only an authorized browser session can validate login and selectors. |
| ADD-3: diff mode | Optional enhancement | Fits the product once reliable CSV output and tests exist. |
| ADD-4: metadata enrichment | Optional enhancement | Useful but adds API, attribution, rate-limit, and data-quality dependencies. Defer. |
| ADD-5: headless by default | Product choice, not verified defect | Avoid changing the default before live debugging and authentication behavior are understood. |
| ADD-6: use WebDriverWait for loading | Confirmed maintenance improvement | The fixed sleep is present; use measurable page-state conditions. |
| DIR-1: owned-media catalog toolkit | Speculative direction | A coherent long-term expansion only after this single-provider tool is stable. |
| DIR-2: Plex/Stash/media-stack import | Speculative direction | Requires a defined target format and product/privacy decision; do not place in the stabilization backlog. |

### CLAUDE.md task inventory

| Existing item | Current status | Verification and next action |
| --- | --- | --- |
| Task 1: update Selenium selectors | Confirmed conditional work | Needed only after authorized live contract validation identifies current selectors. |
| Task 2: add JSON/Excel exports | Optional enhancement | CSV is the only implemented format. Defer until the core export is reliable. |
| Task 3: improve scrolling algorithm | Confirmed maintenance opportunity | Fixed sleep is present; prioritize after live behavior is measured. |
| Task 4: pagination support | Conditional / speculative | No evidence that the service uses pagination today. Keep as a contingency, not active backlog. |
| Task 5: TV shows / content types | Confirmed absent feature | GitHub description overstates the product today; add only after movie scraping is stabilized. |
| Future high 1: automated tests | Confirmed need | No tests exist. Prioritize in stabilization. |
| Future high 2: requirements.txt | Confirmed need | No dependency declaration exists. Prioritize in stabilization. |
| Future high 3: logging | Confirmed improvement | Current print-based feedback is limited. Add structured logging after test seams exist. |
| Future high 4: TV shows | Optional enhancement | Not implemented; defer behind movie reliability. |
| Future high 5: resume capability | Optional enhancement | Useful for very large collections, but no failed-run evidence exists. Explore after baseline telemetry. |
| Future medium 6: rate limiting | Conditional improvement | Good etiquette and reliability control; define only after observing live behavior and service expectations. |
| Future medium 7: retry logic | Confirmed reliability opportunity | Current generic browser failure exits; retry policy should be constrained and observable. |
| Future medium 8: progress bar | Optional enhancement | Fits a long-running CLI but is secondary to correctness. |
| Future medium 9: multiple export formats | Optional enhancement | Same disposition as Task 2. |
| Future medium 10: proxy support | Deferred / not recommended now | Increases compliance, privacy, and support complexity without a verified user need. |
| Future low 11: GUI | Deferred | A GUI would multiply support surface before the CLI is stable. |
| Future low 12: Docker | Deferred | Browser automation containers require platform and sandbox decisions; no current need. |
| Future low 13: scheduled runs | Deferred | Requires reliable authentication, credential storage, and terms review first. |
| Future low 14: diff mode | Optional enhancement | A good post-stabilization feature; duplicate of ADD-3. |
| Future low 15: metadata enrichment | Optional enhancement | A good exploratory idea; duplicate of ADD-4. |
| Cleanup recommendation: archive vudu.py | Confirmed | Legacy version is materially obsolete and misleading. |
| Cleanup recommendation: archive vuduupdatedbyclaude.py | Confirmed | Legacy intermediate version is materially obsolete and misleading. |
| Cleanup recommendation: rename current script | Confirmed | Extensionless filename blocks normal import/tooling. |
| Cleanup recommendation: add dependencies and .env.example | Partially confirmed | Dependency metadata is required; an .env.example is useful only if it contains names/comments and no credential-like values. |

### GitHub history and closed-work inventory

| Existing item | Current status | Verification and next action |
| --- | --- | --- |
| Closed issue 1: Better Code Hub refactor suggestion | Not implemented in legacy file, but obsolete | The legacy file remains. Do not reopen automatically; archival is preferable to refactoring it. |
| Closed issue 2: Better Code Hub refactor suggestion | Not implemented in legacy file, but obsolete | Same disposition as issue 1. |
| Pull request 3: CLAUDE.md documentation | Merged | Useful documentation landed, but it now needs reconciliation with current code and branch state. |
| Pull request 4: PROJECT_AUDIT.md | Closed unmerged / historical duplicate | A dangling local object for its head exists. Preserve history; do not revive it as current work. |
| Pull request 5: master documentation merge | Merged | It placed the previous local master tree on remote main. |

## Newly Discovered Findings

No critical or high-severity defect was confirmed. Severity below reflects user impact and evidence, not the importance of eventual cleanup.

### Medium

#### BUG-001: An empty valid library is treated as a navigation timeout

| Field | Evidence |
| --- | --- |
| Category | Correctness and user experience |
| Affected component | vuduupdatedbyOpenAI, navigate_to_my_movies at lines 105-109 and main at lines 212-213 |
| Evidence | Navigation waits for at least one element matching the movie-image selector. A genuinely empty library has no such element, so execution times out before the later no-titles warning can run. |
| Impact | A user with no movies, a temporarily hidden library, or a new account gets a failure instead of a successful empty CSV and clear state. |
| Verification | Static control-flow review of the wait and the later warning branch. |
| Recommended fix | Wait for a library container or authenticated page marker, then distinguish an empty collection from a selector/login failure. Add tests for populated, empty, and selector-missing DOM fixtures. |
| Confidence | High |

#### BUG-002: Login success is not meaningfully checked

| Field | Evidence |
| --- | --- |
| Category | Correctness and reliability |
| Affected component | vuduupdatedbyOpenAI, login at lines 98-102 |
| Evidence | The post-submit condition is that current_url contains vudu.com. The configured login URL already contains vudu.com, so the predicate can pass immediately without proving a successful authentication transition. |
| Impact | Invalid credentials, MFA prompts, changed forms, or blocked automation can turn into a later, less diagnostic navigation failure. |
| Verification | Static review of LOGIN_URL and the lambda predicate. |
| Recommended fix | Wait for a post-login page marker, an authenticated account control, or a defined library URL. Surface a specific failure when the login form or challenge remains present. |
| Confidence | High |

#### SETUP-001: A clean checkout lacks an executable dependency contract

| Field | Evidence |
| --- | --- |
| Category | Build, reproducibility, and developer experience |
| Affected component | Repository root and setup documentation |
| Evidence | No requirements.txt, pyproject.toml, lockfile, or CI environment exists. selenium and webdriver-manager are absent locally; --help fails during the unconditional Selenium import. |
| Impact | New users and CI cannot reliably install, invoke, or verify the project. Dependency compatibility and supported Python versions are unspecified. |
| Verification | pip show, CLI help, and repository inventory. |
| Recommended fix | Add a single authoritative package/dependency definition, optional dependency policy, supported Python range, and a clean-install smoke test. |
| Confidence | High |

### Low

| ID | Title | Category | Evidence and impact | Recommended action | Confidence |
| --- | --- | --- | --- | --- | --- |
| ARCH-001 | Three conflicting executable implementations | Maintainability | Two legacy files and one current extensionless file use incompatible Selenium approaches, driver setup, and credential handling. Contributors can edit the wrong one. | Choose one canonical module; archive or remove the legacy copies in a reviewed migration. | High |
| PRIV-001 | Personal CSV exports are not ignored | Privacy and repository hygiene | Neither vudu_movies.csv nor Example2.csv is ignored. A normal git add operation could stage a personal ownership list. | Add documented export patterns to .gitignore or move exports to an ignored directory; explain the privacy implication. | High |
| SEC-HARD-001 | Chrome sandbox is disabled unconditionally | Security hardening | build_driver always adds --no-sandbox. The completed security review found no reportable lower-privileged attack path in this local tool, but this weakens normal desktop browser isolation. | Remove it by default; make any container-specific exception explicit and documented. | High |
| PRIV-002 | Password-bearing CLI path is documented | Privacy | --pass accepts secrets from shell history and process arguments. It is not a reportable repository vulnerability under the scan threat model, but it is avoidable exposure. | Prefer VUDU_PASS or a no-echo prompt; retain compatibility only with a clear warning if needed. | High |
| UX-001 | Numeric CLI values are not range validated | Reliability | --wait, --pageload, --idle-rounds, and --max-min accept arbitrary integers, including zero or negative values. | Validate positive bounds and present actionable argument errors. | High |
| DOC-001 | Current public documentation and source docstring disagree with code | Documentation | README describes the original interactive workflow; the current docstring names a non-existent tools path. | Rewrite README and update the module docstring after the canonical name is selected. | High |

### Informational and risks requiring verification

| ID | Observation | Evidence | Recommended handling |
| --- | --- | --- | --- |
| RISK-001 | External site contract may be stale | Anonymous navigation to the configured URL redirected to Fandango at Home; no authenticated DOM review was authorized | Treat live selector validation as a release gate, not a confirmed current breakage |
| RISK-002 | Formula-looking CSV values are preserved | Dynamic local serialization probe showed leading =, +, -, and @ values remain unchanged; normal attacker control of title data was not established | Reassess formula neutralization if title sources broaden beyond the owner-controlled account |
| INFO-001 | No reportable security vulnerabilities survived the completed scan | Repository-wide security scan covered all three scripts and candidate attack paths | Track hardening items as maintenance, not as an asserted exploit |
| INFO-002 | No release or packaging process exists | No tags, releases, packages, or build metadata were found | Establish a small release checklist only after installation and CI work land |

## Architecture Assessment

### Strengths

- The current script separates driver construction, login, navigation, collection, CSV writing, and CLI parsing into individual functions.
- Config is an immutable dataclass, which supports testing and avoids ambient globals in the current code.
- Explicit Selenium waits, bounded scroll termination, timeout configuration, specific browser exceptions, and driver cleanup are all better than the legacy scripts.
- The current script avoids checked-in real credentials and supports environment variables.

### Weaknesses and debt

- The single file is not a normal Python module, which blocks imports, test discovery, common linters, and static tools.
- Selenium DOM selectors, URL fragments, and login assumptions are embedded as constants without a testable page abstraction or sanctioned fixture.
- The scraper conflates library readiness with the existence of at least one movie item.
- Browser setup mixes normal desktop choices, container workarounds, optional driver installation, and anti-detection behavior with no platform policy.
- There is no dependency boundary or version policy, so Selenium/Chrome compatibility is ungoverned.
- Legacy copies create ambiguity instead of preserving useful version history, which Git already provides.

### Recommended architectural evolution

Keep the design intentionally small. Convert the current script into one importable module with a focused CLI wrapper rather than introducing a large framework. Establish interfaces around:

1. Configuration and credential acquisition.
2. Browser/session construction.
3. Site contract: login verification, collection-ready state, empty state, title extraction, and scroll completion.
4. Output writing and explicit overwrite policy.
5. Pure helper logic that can be tested without Chrome.

Do not extract a provider-plugin architecture until the single-provider contract is reliable and there is a selected second provider. Do not containerize or add a GUI before the local CLI has a repeatable test and release path.

## Test and Quality Assessment

There is no automated test coverage. The code compiles, but compilation does not exercise Selenium imports, browser creation, login, DOM selection, scrolling, CSV behavior, or CLI errors.

Recommended coverage layers:

| Layer | First tests |
| --- | --- |
| Unit | Argument bounds, credential precedence, CSV output and overwrite semantics, deduplication, sort behavior |
| DOM fixture | Populated library, empty library, changed selector, login failure/MFA state |
| Browser integration | Opt-in authorized smoke test against a test account, never in ordinary CI |
| Regression | Sanitized HTML fixtures from the live contract check |
| Quality automation | Formatter, linter, type checking appropriate to the selected package layout, and test execution |

The proposed old CI smoke import is invalid until the canonical source has a Python module filename. Tests must also avoid embedding any real credentials, cookies, library exports, or unredacted page snapshots.

## Security and Privacy Assessment

The completed repository-wide security scan found no validated reportable vulnerability in the local single-user threat model. It reviewed every executable script and preserved five candidates through validation and attack-path analysis: three CSV formula-handling instances, unconditional --no-sandbox, and CLI password arguments. All were rejected from reportable status because no realistic lower-privileged in-scope attacker path was established.

Confirmed hardening and privacy work remains appropriate:

- Do not disable Chrome sandboxing by default on a normal local desktop.
- Prefer environment variables or a no-echo prompt over password-bearing CLI arguments.
- Ignore or deliberately locate local CSV exports because ownership lists can be personal information.
- Archive legacy code containing credential placeholders, even though the checked-in values are not secrets.
- Add dependency pinning/version policy and secret scanning once project tooling exists.

Potential, not confirmed, risks:

- Spreadsheet formula interpretation becomes more important if title data can be influenced by other users, imports, shared libraries, or a compromised upstream source.
- webdriver-manager and an arbitrary local ChromeDriver path are supply-chain/local-environment trust boundaries; the repository alone cannot establish their runtime integrity.
- The anti-detection browser flag warrants a manual third-party terms review. This audit did not verify or assert a terms violation.

## Performance Assessment

No runtime profile was captured because the external authenticated collection was not exercised. The current in-memory set and final sort are appropriate for an ordinary personal library. The confirmed inefficiency is the fixed one-second polling delay after every scroll, independent of page readiness. Measure collection time, rendered element counts, and scroll rounds during the authorized contract check before selecting an adaptive wait or retry policy.

## Accessibility and User Experience Assessment

This is a command-line tool; native visual accessibility, touch targets, and screen-reader markup are not applicable to the repository's UI. The third-party website is not controlled by this project and was not audited for accessibility.

CLI experience still needs attention:

- Empty collection, bad credentials, MFA, selector changes, and network failures are not clearly distinguished.
- --help cannot run without Selenium installed.
- Parameter bounds and output overwrite behavior are unclear.
- README does not give a usable current setup or privacy-safe credential example.
- Headless behavior should remain a conscious debugging/product decision until live authentication is validated.

## Documentation Assessment

| Document | Status | Problems | Recommended action |
| --- | --- | --- | --- |
| README.md | Update / replace | Describes legacy prompting flow and static ChromeDriver assumptions; lacks current installation, CLI, privacy, troubleshooting, project status, and Fandango at Home context | Rewrite after the canonical module and dependency contract are selected |
| AGENTS.md | Keep | Repository-default agent guidance with current project constraints and validation limits | Keep project-wide instructions here and align it with current code, analysis, and roadmap |
| CLAUDE.md | Keep minimal | Claude-specific entry point only | Point to AGENTS.md; do not duplicate project-wide guidance |
| ANALYSIS.md | Keep | New canonical evidence-based audit | Update only after meaningful code, site-contract, or GitHub-state changes |
| ROADMAP.md | Keep | New actionable plan tied to verified findings | Use as the one current planning tracker until work lands |
| .gitignore | Update | Does not ignore local movie export CSV files | Add selected export patterns when output policy is decided |
| LICENSE | Done 2026-10-01 | Owner selected MIT (explicit direction); LICENSE added in PR #9 | Done |
| CONTRIBUTING.md | Create if accepting outside contributions | No contribution workflow or setup contract | Add after packaging, tests, and CI exist |
| SECURITY.md | Create if publicly maintained | No reporting path or scope statement | Add a short policy after deciding whether external reports are accepted |
| CHANGELOG.md | Conditional | No releases exist | Add when versioned releases begin; do not invent historical entries |
| Setup/testing/deployment docs | Create selectively | No reproducible setup or test instructions; deployment is not applicable | Put basic setup/testing in README first; avoid a deployment guide for a non-deployed CLI |

Recommended final documentation structure:

- README.md: product status, supported platform, install, safe credential setup, run examples, output/privacy, troubleshooting, and current limitations.
- AGENTS.md: repository-default code architecture, test guidance, and agent/developer conventions.
- CLAUDE.md: Claude-specific pointer only.
- ANALYSIS.md and ROADMAP.md: current audit and ordered work until superseded.
- docs/site-contract.md: dated, sanitized selector/login contract and fixture-refresh procedure once live validation is authorized.
- SECURITY.md and CONTRIBUTING.md only when their ownership and maintenance policies are decided. (LICENSE done: owner selected MIT on 2026-10-01.)

## GitHub Repository Assessment

### Public-facing state verified

| Area | Observation | Recommendation |
| --- | --- | --- |
| Description | It says the project scrapes movies and TV shows, while code only implements movies | Correct the description or implement TV support later |
| Default branch | GitHub API and remote metadata identify main as default | Keep main; no default-branch migration is needed |
| Issues and pull requests | No open issues or pull requests were found | Add issue and PR templates after the core workflow is defined |
| Closed issues | Two Better Code Hub refactor issues target legacy code | Leave closed; record their obsolete disposition rather than reopening |
| Releases/tags/packages | No releases, tags, or packages were found | Do not add release process until install/test/CI baseline exists |
| GitHub Actions | No workflow files or observed runs | Add a minimal test/lint workflow after packaging |
| Public README | It accurately reflects the stale local README | Rewrite it to improve evaluation and installation |
| Repository presentation | No verified topics, website, social preview, screenshots, or demo asset were available through access | Add a precise description, topics, status notice, and a sanitized screenshot/demo only after core function works |

The anonymous public page renderer showed an older-looking master view while GitHub API and remote Git metadata showed main as the default branch and the current code tree. Treat the API/remote state as authoritative, but manually check the branch selector and public page cache after the next push.

The following settings were not available to inspect and require a manual GitHub review:

- Branch protection or rulesets for main.
- Required checks, push restrictions, merge queue, and Actions permissions.
- Topics, homepage URL, social preview, funding, wiki, discussions, and Pages.
- Dependabot configuration, secret scanning, code scanning, and private vulnerability reporting.
- Issue forms, pull request template, labels, milestones, and project boards.

## Branch Assessment and Git Hygiene

Remote default branch is main. It already contains the code tree from the old local master through merged pull request 5. No branch rename or force operation is appropriate.

| Branch or object | Last activity | Merge status | Associated pull request | Unique commits versus origin/main | Recommended action | Reason |
| --- | --- | --- | --- | --- | --- | --- |
| origin/main | 2026-07-08 | Default active branch | Pull request 5 merged into it | N/A | Keep | Authoritative remote branch |
| docs/repository-audit | Created during this audit from origin/main | Unmerged local documentation branch | None yet | Audit documentation commits only | Keep until reviewed and merged | Isolated, reversible audit work |
| master | 2026-07-08 | Fully merged into origin/main | Pull request 5 | 0 | Review, then delete local branch after audit work is merged and no local workflow depends on it | Tracks a gone origin/master and duplicates main's tree |
| origin/master | Not present after remote pruning | N/A | Historical source branch | N/A | No action | It is no longer a remote branch |
| Dangling commit 8f8b5d7 | 2026 closed PR history | Unmerged | Pull request 4 | Not a branch | Preserve; do not garbage-collect as part of this audit | Historical PROJECT_AUDIT.md work may be recoverable if needed |

No branches were deleted. The working tree was clean before documentation changes, and no shared history was rewritten.

## Product and Feature Opportunities

### Near-term, evidence-based improvements

| Opportunity | Value | Complexity | Dependencies | Timing |
| --- | --- | --- | --- | --- |
| Live site-contract validation | Makes all scraping behavior trustworthy | Medium | Authorized account and manual browser test | Near-term |
| One canonical importable module | Reduces contributor mistakes and enables tools | Medium | Live contract decision | Near-term |
| Dependency definition, tests, and CI | Makes setup reproducible and prevents regressions | Medium | Canonical module layout | Near-term |
| Clear empty/login/error states | Reduces misleading failures | Low to medium | DOM fixture design | Near-term |
| Privacy-safe export policy | Reduces accidental personal-data commits | Low | Output naming decision | Near-term |

### Larger feature ideas that fit the product

| Idea | Value | Complexity and risk | Fit |
| --- | --- | --- | --- |
| TV-show export | Aligns product description with functionality | Medium; depends on live site contract and content-specific selectors | Good after movie stability |
| Diff against previous export | Identifies additions/removals in an ownership inventory | Low to medium; needs stable identifier and output policy | Strong fit |
| JSON/SQLite export | Supports downstream cataloging | Medium; adds schema/versioning decisions | Good after CSV contract is stable |
| Retry/progress/logging | Better long-running CLI usability | Medium; must avoid opaque or aggressive automation | Good fit |
| Metadata enrichment | More useful catalog data | Medium to high; adds APIs, terms, attribution, and data-quality work | Optional, later |

### Alternative directions and experiments

- A multi-provider owned-media catalog is a reasonable strategic direction only after a second provider is chosen and the current provider has a stable interface.
- Plex or other media-library integration could be useful for personal inventory reconciliation, but requires a product and privacy decision about data interchange.
- Evaluate Playwright only if Selenium/Chrome compatibility proves costly; do not switch toolchains before measuring the current failure mode.

### Ideas not recommended now

- Proxy support, aggressive anti-detection behavior, or automation intended to evade third-party controls.
- A GUI, Docker image, or scheduling system before authentication, selectors, dependencies, and tests are stable.
- Metadata APIs or platform integrations before the core CSV is demonstrably correct.

## Recommended Priorities

1. Obtain authorization and perform a live Fandango at Home contract check: current URLs, login outcome, empty state, item selectors, and scrolling behavior.
2. Fix BUG-001 and BUG-002 against that verified contract.
3. Select one conventional Python module/CLI name and archive or remove legacy variants in a focused migration.
4. Add an authoritative dependency/project definition and clean-install documentation.
5. Add DOM fixtures and tests, including empty, failed-login, and changed-selector behavior.
6. Add minimal CI using the new module name and declared tools.
7. Remove default --no-sandbox, reduce password CLI exposure, and ignore personal exports.
8. Rewrite README and keep AGENTS.md aligned with code, Git state, and the planning documents.
9. Improve GitHub metadata, templates, branch protection, and dependency automation.
10. Consider TV support and export enhancements only after the above is complete.

## Limitations

- No live third-party account, login, MFA, bot-detection, or authenticated selector behavior was tested.
- No dependency installation, browser launch, or external package vulnerability scan was performed because no canonical dependency contract exists.
- GitHub settings and some presentation metadata were inaccessible; no claim is made that branch protection, rulesets, dependency automation, secret scanning, or templates are absent in GitHub settings.
- The security conclusion covers the checked-in repository and its local threat model, not Chrome, drivers, package registries, the third-party service, or a compromised workstation.
- The public GitHub renderer disagreed with API/remote branch metadata, so public-page cache/branch presentation needs manual confirmation after the next push.
