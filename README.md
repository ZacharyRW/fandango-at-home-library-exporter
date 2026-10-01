# Fandango at Home Library Exporter

Export the list of movies and TV shows you own on Fandango at Home (formerly Vudu) to a CSV file.

## How it works

The script opens Chrome with Selenium, logs in to your Fandango at Home account, scrolls through your library to load the dynamically-rendered titles, then writes the deduplicated, alphabetically-sorted titles to a CSV file.

## Which script to use

This repo holds three versions of the scraper, oldest first:

| File | Status |
| ---- | ------ |
| `vudu.py` | Original (2017). Written for Selenium 3 — it uses the `find_element_by_*` API that was removed in Selenium 4, so it will not run on modern Selenium. |
| `vuduupdatedbyclaude.py` | Updated to Selenium 4 syntax (`By`, `WebDriverWait`). Credentials are still hard-coded placeholders at the top of the file. |
| `vuduupdatedbyOpenAI` | Most current. Selenium 4, command-line options (`--out`, `--headless`), credentials via `--user` / `--pass` flags or the `VUDU_USER` / `VUDU_PASS` environment variables, and optional `webdriver-manager` support so you don't need a manually installed chromedriver. Note: the file is missing its `.py` extension — rename it to `vuduupdatedbyOpenAI.py` before running. |

**Recommended:** use `vuduupdatedbyOpenAI` (renamed with a `.py` extension).

## Requirements

- Python 3
- Google Chrome
- `pip install selenium` (plus `webdriver-manager` if you want automatic chromedriver handling with the OpenAI variant)

The two older scripts additionally need a manually downloaded [ChromeDriver](https://chromedriver.chromium.org/) matching your Chrome version, pointed to by the `chromedriver = 'C:\\chromedriver.exe'` line at the top.

## Usage (recommended variant)

```bash
# rename once — the file is missing its .py extension in the repo
mv vuduupdatedbyOpenAI vuduupdatedbyOpenAI.py

# run it
python vuduupdatedbyOpenAI.py --out movies.csv --headless
# or with env vars instead of flags:
# VUDU_USER=you@example.com VUDU_PASS=secret python vuduupdatedbyOpenAI.py --out movies.csv
```

For `vudu.py` / `vuduupdatedbyclaude.py`, edit the `USERNAME` and `PASSWORD` placeholders at the top of the file first — and never commit your real credentials.

## Caveats

- These scripts were written against the Vudu-era site layout; pages and selectors have changed since, so if the scraper can't find the login fields or movie tiles, the CSS selectors near the top of the file are the first thing to update.
- Automated logins can trip bot protection. If you hit a CAPTCHA or login wall, wait a while and retry, or log in manually in the same Chrome profile first.
