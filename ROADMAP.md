# Project Roadmap

This roadmap is derived from the verified findings in ANALYSIS.md, not from unverified historical TODOs. It assumes the project remains a small, local, personal media-library export tool. It deliberately separates confirmed defects and maintenance from optional product ideas.

## Roadmap Principles

Work is prioritized using the following order:

1. User impact and correctness: a trustworthy export matters more than new formats or integrations.
2. Risk reduction: resolve incorrect login/empty-state behavior, insecure defaults, and accidental data exposure early.
3. Dependency order: validate the external site contract before freezing selectors, tests, docs, or feature behavior.
4. Reproducibility: make setup, tests, and CI work before relying on manual knowledge.
5. Maintainability: one canonical module and one planning source are better than parallel scripts and duplicated roadmaps.
6. Strategic fit: grow from a reliable personal inventory tool only when the core purpose is proven.
7. Explicit authority: do not automate third-party account flows, delete legacy history, choose a license, or change GitHub settings without the required owner decision.

Priority labels:

- P0: release or safety gate; start immediately when authorized.
- P1: necessary for a stable, maintainable first release.
- P2: valuable after the stable baseline exists.
- P3: optional or strategic.

Effort labels are relative: S means a focused change, M means several coordinated changes with tests, and L means a multi-stage product decision.

## Phase 0: Immediate Safety and Repository Hygiene

There is no confirmed critical exploit, secret leak, broken source compilation, or default-branch migration need. This phase is intentionally small and can be completed without feature expansion.

| ID | Work | Why now | Completion criteria |
| --- | --- | --- | --- |
| PRIV-001 | Protect personal export files | Default and legacy CSV outputs are not ignored and can be staged accidentally | Output policy is documented; selected local export patterns or directory are ignored; a test/check proves exports are not staged by default |
| SEC-001 | Remove unconditional Chrome --no-sandbox | It weakens normal desktop browser containment, even though the security scan found no reportable exploit path | Normal desktop runs do not set --no-sandbox; any container exception is explicit, opt-in, and documented |
| PRIV-002 | Reduce credential exposure through CLI | --pass can expose credentials in shell history and local process arguments | README prefers VUDU_PASS or a no-echo prompt; compatibility behavior and warning are tested |
| BR-001 | Resolve obsolete local master branch safely | Local master is fully merged into origin/main and tracks a gone remote | After local use is confirmed absent, delete only the local master branch; do not rewrite history |
| GH-001 | Confirm main protection manually | main is already the remote default, but settings were not inspectable | Repository owner verifies protection/ruleset and chooses required checks after CI exists |

Do not remove any secret in this phase: the legacy credential strings were verified placeholders, not a discovered secret. Do not delete branches automatically.

## Phase 1: Stabilization

This phase makes the existing movie export trustworthy and reproducible. It is the prerequisite for a public README refresh or meaningful feature work.

| ID | Work | Scope | Success criteria |
| --- | --- | --- | --- |
| REL-001 | Authorize and perform a live site-contract check | Validate current Fandango at Home login URL, authentication transition, movie-library URL, empty state, item selectors, and scroll completion with a controlled account | Dated, sanitized evidence exists; no credentials/cookies/page-private data are committed; selector contract is explicitly accepted |
| BUG-001 | Correct valid empty-library behavior | Separate collection-ready state from the presence of a movie tile | An authorized empty or fixture-based empty library writes a successful empty CSV with a clear message |
| BUG-002 | Make authentication postcondition meaningful | Replace the existing URL-substring predicate with an authenticated state or post-login marker | Invalid credentials, MFA/challenge, and changed login UI fail distinctly; successful login is verified before scraping |
| SETUP-001 | Define reproducible installation | Add one authoritative dependency/project configuration and supported Python policy | A clean environment installs declared dependencies; help and a smoke test run without manual package discovery |
| ARCH-001 | Establish one canonical importable module | Rename the extensionless current script, preserve CLI behavior intentionally, and archive/remove the two legacy copies in a reviewed change | Normal Python import and tools work; README names one supported command; legacy status is unambiguous |
| TEST-001 | Build a baseline automated test suite | Cover pure logic, CLI validation, CSV output, login/empty DOM states, and selector failure fixtures | Tests run locally with no account and have meaningful failure-path assertions |
| UX-001 | Validate CLI ranges and output policy | Reject negative/zero timeouts where invalid and make overwrite behavior deliberate | Invalid input returns actionable errors; output overwrite behavior is documented and tested |

Phase 1 exit gate:

- An authorized live contract check has been completed and documented.
- The project installs from a clean environment.
- Tests cover populated, empty, failed-login, and changed-selector paths without credentials.
- The movie export has a clear success, empty, and failure state.

## Phase 2: Maintainability and Developer Experience

| ID | Work | Value | Success criteria |
| --- | --- | --- | --- |
| TEST-002 | Add sanitized DOM fixtures and refresh procedure | Prevent silent selector regressions without storing private account data | Fixtures describe populated, empty, login-failed, and selector-missing states; refresh steps are documented |
| CI-001 | Add a minimal GitHub Actions workflow | Makes regression checks visible on pull requests | Supported Python matrix, formatter/linter or static checks, and tests pass on every PR |
| DX-001 | Add structured logging and diagnostic mode | Makes browser and selector failures actionable without leaking credentials | Log levels, redaction rules, and optional screenshots/DOM diagnostics are documented and tested |
| PERF-001 | Replace fixed scroll sleeps with page-state waiting | Improves speed and reliability based on observed behavior | Scroll completion waits on a measurable condition; the time limit remains bounded; behavior is tested with fakes/fixtures |
| REL-002 | Add constrained retries and rate limiting | Handles transient browser/network failures responsibly | Retry policy has limits, backoff, logging, and no behavior intended to evade third-party controls |
| DOC-001 | Rewrite README for the supported CLI | Align public onboarding with the working product | README includes install, safe credential setup, run examples, output/privacy, limitations, and troubleshooting |
| DOC-002 | Maintain repository-default agent guidance | Reduce drift between agent instructions, code, and plans | AGENTS.md is the project-wide default; CLAUDE.md contains only Claude-specific guidance and points to AGENTS.md |
| DOC-003 | Create a site-contract/testing note | Centralizes changing external assumptions | A dated docs/site-contract.md or equivalent explains only sanitized selectors, fixtures, and refresh conditions |
| GH-002 | Add contribution and issue workflow files | Makes maintenance predictable | Issue forms/template and PR template exist if outside contributions are invited; CONTRIBUTING.md matches actual local setup |
| DEP-001 | Add dependency update and vulnerability review policy | Makes browser/dependency updates deliberate | Dependabot or a documented manual cadence exists after a manifest and CI are stable |

## Phase 3: Product Improvements

These initiatives fit the verified product identity, but must wait for the Phase 1 exit gate.

| ID | Initiative | Value | Dependencies | Notes |
| --- | --- | --- | --- | --- |
| FEAT-001 | TV-show or content-type export | Brings product scope in line with the public description if selected | REL-001, TEST-002, documented content-specific contract | Do not claim support until verified |
| FEAT-002 | Diff mode against a prior export | Shows added or removed titles over time | Stable CSV schema, output policy, test fixtures | Define how renamed/duplicate titles are handled |
| FEAT-003 | JSON or SQLite export | Enables personal cataloging and downstream tools | Stable domain/schema design | Prefer a versioned schema over ad hoc columns |
| FEAT-004 | Progress reporting and resume support | Improves long-library runs | Reliable scroll state, logging, output design | Add only if measured runs show a need |
| FEAT-005 | Metadata enrichment | Adds release years, genres, and other catalog fields | API selection, terms, attribution, rate limits, data model | Treat as a separate integration project |

## Phase 4: Strategic Expansion

| ID | Direction | Value | Complexity and risk | Decision gate |
| --- | --- | --- | --- | --- |
| STRAT-001 | Multi-provider owned-media catalog | Consolidates inventories across vendors | L: provider contracts, auth, data normalization, long-term maintenance | Select and validate a second provider only after Vudu/Fandango flow is stable |
| STRAT-002 | Media-library integration | Reconciles owned inventory with a Plex or similar library | L: privacy, data format, integration support | Define user workflow and local-data boundary before implementation |
| STRAT-003 | Packaged distribution | Easier installation for non-developers | M to L: browser dependencies, signing, support matrix | Pursue only after repeatable CLI release process works |

## Exploratory Ideas

These are research items, not accepted implementation work:

- Compare Selenium and Playwright only if the verified site contract exposes Selenium-specific reliability or compatibility problems.
- Validate whether the service has a sanctioned export or account-data route before increasing automation scope.
- Research whether CSV formula neutralization should be enabled if data sources broaden or users share/import title lists.
- Evaluate a stable title identifier strategy before building diff, metadata, or multi-provider features.
- Determine whether users actually need headless-by-default behavior after observing live login, MFA, and debugging flows.

## Deferred or Rejected Ideas

| Idea | Disposition | Reason |
| --- | --- | --- |
| Proxy support | Deferred | Adds compliance, privacy, and support complexity without a verified need |
| Anti-detection or bypass-focused automation | Rejected | Does not fit a respectful personal-tool posture and could conflict with third-party controls |
| GUI before CLI stabilization | Deferred | Increases support surface without solving core correctness |
| Docker before a stable local release | Deferred | Browser sandbox and driver compatibility require additional platform work |
| Scheduled unattended runs | Deferred | Requires a safe credential-storage, authentication, reliability, and terms decision |
| Metadata enrichment now | Deferred | API/data-quality work would distract from reliable basic export |
| Refactoring legacy scripts in place | Rejected | Archive/remove them after migration instead of maintaining three implementations |
| Default-branch migration | Rejected as unnecessary | Remote default is already main |

## Documentation Plan

Complete documentation in this dependency order:

1. Keep ANALYSIS.md and ROADMAP.md as the current evidence and plan while the stabilization work is open.
2. After REL-001 and ARCH-001, rewrite README.md around the single supported command and dependency method.
3. Add a sanitized site-contract/testing document while creating DOM fixtures.
4. Keep AGENTS.md aligned with the current module, validation commands, branch facts, and links to the canonical analysis/roadmap; keep CLAUDE.md Claude-specific only.
5. Add CONTRIBUTING.md, SECURITY.md, issue templates, and a release checklist only when the maintenance policy and supported workflow are real.
6. Add CHANGELOG.md only when versioned releases begin.
7. Add LICENSE only after the owner selects a license; do not guess.

## GitHub Improvement Plan

1. Correct the repository description so it says movie export only, or wait to update it until FEAT-001 makes TV support real.
2. Add concise topics matching the actual technology and scope once the README is current.
3. Add a project-status notice and a sanitized screenshot or short demo only after a verified run exists.
4. Add CI after SETUP-001 and TEST-001, then manually configure main branch protection to require those checks.
5. Add Dependabot or a documented dependency-review cadence after the manifest is introduced.
6. Add issue forms, a PR template, CONTRIBUTING.md, and SECURITY.md only if external participation/reporting is desired.
7. Decide whether releases and release notes serve the project after the first reproducible version exists.
8. Manually review public branch presentation because the anonymous page renderer showed stale master-like data while API and Git remote metadata reported main.

## Branch Cleanup Plan

### Safe to delete now

None. No destructive branch change is authorized by this roadmap.

### Review before deletion

| Item | State | Required check | Proposed action |
| --- | --- | --- | --- |
| Local master | Fully merged into origin/main, tracks gone origin/master, zero unique commits versus main | Confirm no local worktree, script, deployment, or personal workflow still refers to master | Delete locally only after explicit owner confirmation |
| Dangling commit 8f8b5d7 | Historical closed unmerged pull request 4 head | Confirm whether its PROJECT_AUDIT.md history is wanted | Preserve; do not run garbage collection as audit cleanup |

### Keep

| Branch | Reason |
| --- | --- |
| origin/main | Current default and authoritative remote branch |

### Rename or migrate

No branch migration is needed. The desired default branch, main, is already in use. The later ARCH-001 filename/module migration is a code-layout change, not a branch migration.

### Manual GitHub action required

- Verify main remains the default in repository settings.
- Add appropriate protection/ruleset and required checks after CI exists.
- Confirm no external badge, deployment, webhook, or documentation reference still assumes master.
- Verify the public branch selector/cache displays main after the next push.

## Milestone Table

| ID | Initiative | Priority | Effort | Dependencies | Target phase | Success criteria |
| --- | --- | --- | --- | --- | --- | --- |
| REL-001 | Authorized live site-contract verification | P0 | M | Test account and explicit authorization | 1 | Current login, selectors, empty state, and scrolling behavior are documented without secrets |
| BUG-001 | Empty-library success path | P1 | S | REL-001 or representative fixture | 1 | Empty collection creates an empty CSV and a clear successful result |
| BUG-002 | Meaningful login postcondition | P1 | S | REL-001 or representative fixture | 1 | Failed login/challenge and success are distinguishable |
| SETUP-001 | Dependency and project definition | P1 | M | Canonical module decision | 1 | Clean install and CLI help work from documented steps |
| ARCH-001 | One importable canonical module | P1 | M | REL-001 recommendation | 1 | One named supported entry point; legacy implementations are clearly historical |
| TEST-001 | Baseline unit and CLI tests | P1 | M | ARCH-001 | 1 | Core logic and error paths run without browser/account access |
| UX-001 | Argument and overwrite policy | P1 | S | TEST-001 | 1 | Invalid values and overwrite behavior are explicit and tested |
| PRIV-001 | Ignore personal exports | P0 | S | Output policy | 0 | Default export cannot be staged accidentally |
| SEC-001 | Desktop-safe browser sandbox defaults | P0 | S | Browser platform policy | 0 | --no-sandbox is absent by default and any exception is opt-in |
| PRIV-002 | Credential entry hardening | P1 | S | README/update of CLI behavior | 0-1 | Safe credential path is default/documented and does not leak in examples |
| TEST-002 | Sanitized DOM fixture suite | P2 | M | REL-001, TEST-001 | 2 | Selector regressions and empty/login states are testable offline |
| CI-001 | Pull-request validation workflow | P2 | S | SETUP-001, TEST-001 | 2 | CI executes declared checks successfully |
| PERF-001 | Event-driven scroll waiting | P2 | M | REL-001, TEST-002 | 2 | Bounded, measurable loading wait replaces blind polling |
| DX-001 | Logging and diagnostics | P2 | M | ARCH-001, TEST-001 | 2 | Redacted diagnostics make failures actionable |
| DOC-001 | README rewrite | P1 | S | SETUP-001, ARCH-001 | 2 | Public docs accurately install and run the project |
| DOC-002 | Repository-default agent guidance | P2 | S | DOC-001 | 2 | AGENTS.md is current and Claude-specific guidance is isolated in CLAUDE.md |
| GH-001 | Main branch/manual settings review | P1 | S | CI-001 for required checks | 0-2 | Owner confirms default/protection/ruleset behavior |
| GH-002 | Contribution and issue workflow | P2 | S | DOC-001, CI-001 | 2 | Templates and guidance match actual maintenance practice |
| BR-001 | Local master cleanup | P3 | S | User confirmation | 0 or after merge | Only obsolete local master is deleted; no remote/shared history changes |
| FEAT-001 | TV-show export | P2 | M | Phase 1 exit gate | 3 | Product description and tested behavior match |
| FEAT-002 | Export diff mode | P2 | M | Stable output schema and tests | 3 | Added/removed behavior is documented and tested |
| FEAT-003 | JSON/SQLite output | P3 | M | Stable data model | 3 | Versioned schema and tests exist |
| STRAT-001 | Multi-provider catalog | P3 | L | Stable core plus second provider decision | 4 | A product brief and provider architecture are validated before coding |

## Success Metrics

The roadmap is improving the project when these are true:

- A clean environment can install the declared dependencies and show CLI help from the documented command.
- The complete automated test suite passes locally and in GitHub Actions.
- Tests cover populated library, valid empty library, login failure/challenge, selector absence, CSV output, and argument validation.
- A verified live test can distinguish successful login, empty collection, and site-contract failure.
- The repository has one supported implementation and no ambiguous executable legacy copy.
- No default export CSV is accidentally staged by Git.
- The default Chrome configuration does not disable sandboxing on a normal desktop.
- README instructions reproduce the observed setup and output behavior.
- main has appropriate manual protections and required checks after CI is added.
- Confirmed open defects shrink without turning speculative ideas into mandatory backlog work.

## Recommended Execution Order

1. Obtain explicit authorization and run REL-001 against a controlled account.
2. Record the accepted site contract and create sanitized DOM fixtures.
3. Implement BUG-001 and BUG-002 with tests.
4. Implement ARCH-001 and SETUP-001 together in one focused migration, preserving a documented CLI compatibility choice.
5. Add TEST-001, UX-001, PRIV-001, SEC-001, and PRIV-002.
6. Rewrite README and maintain AGENTS.md/CLAUDE.md guidance.
7. Add CI and configure GitHub main protections.
8. Review local master for safe removal after explicit confirmation.
9. Reassess FEAT-001 and FEAT-002 with real user value and the stable test baseline.

## Change Rules

- Preserve uncommitted user work and do not rewrite shared Git history.
- Do not force-push, delete unmerged branches, close issues/PRs, or garbage-collect dangling history as part of roadmap execution.
- Do not treat a historical TODO as current without checking code, Git state, and the accepted site contract.
- Keep verified defects, maintenance, committed plan, optional enhancement, and speculative direction visibly distinct.
- Keep changes small and focused: one purpose per branch/commit where practical.
- Do not commit credentials, cookies, private library exports, or unredacted authenticated page snapshots.
- Do not add automation intended to bypass MFA, CAPTCHA, bot controls, or third-party access restrictions.
- Update ANALYSIS.md and ROADMAP.md when evidence materially changes; archive rather than silently erase useful historical planning.
