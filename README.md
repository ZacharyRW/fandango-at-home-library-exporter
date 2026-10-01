# Fandango at Home Library Exporter

Export the list of movies you own on Fandango at Home (formerly Vudu) to a CSV file.

## How it works

The script opens Chrome with Selenium, logs in to your Fandango at Home account, scrolls through your **My Movies** library to load the dynamically-rendered titles, then writes the deduplicated, alphabetically-sorted titles to a CSV file. TV shows are not exported — the script only navigates the movies library.

## Which script to use

This repo holds three versions of the scraper, oldest first:

| File | Status |
| ---- | ------ |
| `vudu.py` | Original (2017). Written for Selenium 3 — it uses the `find_element_by_*` API that was removed in Selenium 4, so it will not run on modern Selenium. |
| `vuduupdatedbyclaude.py` | Updated to Selenium 4 syntax (`By`, `WebDriverWait`). Credentials are still hard-coded placeholders at the top of the file. |
| `vuduupdatedbyOpenAI` | Most current. Selenium 4, command-line options (`--out`, `--headless`, `--chromedriver`), credentials via `--user` / `--pass` flags or the `VUDU_USER` / `VUDU_PASS` environment variables. Requires `webdriver-manager` unless you pass `--chromedriver`. Note: the file is missing its `.py` extension — rename it to `vuduupdatedbyOpenAI.py` before running. |

**Recommended:** use `vuduupdatedbyOpenAI` (renamed with a `.py` extension).

## Requirements

- Python 3
- Google Chrome
- `pip install selenium webdriver-manager`

`webdriver-manager` is required for the recommended setup: without it (and without a `--chromedriver` path), the script raises `RuntimeError: webdriver-manager not installed and no --chromedriver path provided` before Chrome starts. If you'd rather manage the driver yourself, download a [ChromeDriver](https://chromedriver.chromium.org/) matching your Chrome version and pass `--chromedriver /path/to/chromedriver` (or set the `CHROMEDRIVER` environment variable) instead of installing `webdriver-manager`.

The two older scripts additionally need a manually downloaded ChromeDriver pointed to by the `chromedriver = 'C:\\chromedriver.exe'` line at the top.

## Usage (recommended variant)

```bash
# rename once — the file is missing its .py extension in the repo
mv vuduupdatedbyOpenAI vuduupdatedbyOpenAI.py

# install dependencies
pip install selenium webdriver-manager

# run it
python vuduupdatedbyOpenAI.py --out movies.csv --headless
# or with env vars instead of flags:
# VUDU_USER=you@example.com VUDU_PASS=secret python vuduupdatedbyOpenAI.py --out movies.csv
```

For `vudu.py` / `vuduupdatedbyclaude.py`, edit the `USERNAME` and `PASSWORD` placeholders at the top of the file first — and never commit your real credentials.

## Caveats

- These scripts were written against the Vudu-era site layout, and Fandango pages have changed since. If no titles are exported, check in this order: (1) the login actually succeeded — watch for 2FA/MFA prompts or changed terms (the script raises `TimeoutException` when login likely fails), (2) the library isn't empty, (3) the Fandango URLs haven't changed, and only then (4) whether the CSS selectors near the top of the file need updating. Don't change selectors on guesswork — verify against the live page first.
- Automated logins can trip bot protection. If you hit a CAPTCHA or login wall, wait a while and retry, or log in manually in the same Chrome profile first.
