# Project guide

## Purpose and assessment scope

This project retrieves welcome-bonus information for three configured casinos,
uses a language model to extract structured fields, writes JSON, and displays
the results in a responsive web page. It is a small screening-task prototype,
not a production data service.

The assessment asks for one public source, three casinos, four output fields,
an LLM-assisted extraction step, a single-command scraper, and respectful
handling of `robots.txt`, access restrictions, and blocks. The four output
fields are casino name, welcome bonus, wagering multiplier, and minimum deposit;
the source URL is included as traceability metadata.

## Files

| File | Purpose |
|---|---|
| `scrape.py` | Checks robots rules, fetches source text, calls Groq, validates the extracted fields, and writes JSON. |
| `index.html` | Fetches `bonuses.json` over HTTP and renders a responsive table. |
| `requirements.txt` | Python packages used by the scraper. |
| `.env.example` | Template Groq API key and model settings. |
| `WRITTEN_PART.md` | Production discussion and personal submission sections to complete honestly. |
| `bonuses.json` | Generated output; created when the scraper completes. |

## Runtime requirements

- Python 3.9 or newer.
- The packages listed in `requirements.txt`.
- A valid Groq API key stored in the ignored `.env` file.
- Access to the selected Groq model. The default is `openai/gpt-oss-20b`.
- Network access to the configured source page.

This version uses Groq's hosted API. It does not require downloading a local
model, but it does require internet access and a Groq API key. The source-page
text is sent to Groq for processing. Keep the key in `.env` and do not commit
that file or share its contents.

## Implementation and verification notes

Groq was configured for this project, with the provided credential stored only
in the ignored local `.env` file, not in source code or documentation. The key
and selected model were verified with Groq, a live sample extraction was run,
and the scraper completed for all three configured casinos. It wrote three
records to `bonuses.json`, which were displayed by the local web page. The page
text limit is 8,000 characters, and rate-limit handling uses a bounded number
of retries. Four API-adapter tests and Python compilation passed.

The API key was shared in chat. Revoke it in Groq and replace it in `.env` with
a newly generated key before further use. Do not include the key in this guide,
source files, or screenshots.

## Windows quick start

Run these commands in PowerShell from the project folder:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Copy the example environment file, then add your Groq key:

```powershell
Copy-Item .env.example .env
```

Open `.env` and set `GROQ_API_KEY`. The default model is
`openai/gpt-oss-20b`; optionally set `GROQ_MODEL` to another model enabled
for your account.

Run the scraper:

```powershell
python .\scrape.py
```

Alternatively, without activating the virtual environment:

```powershell
.\.venv\Scripts\python.exe .\scrape.py
```

## macOS and Linux quick start

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cp .env.example .env
# Edit .env and set GROQ_API_KEY before continuing.
python scrape.py
```

The scraper requires internet access to Groq and the configured source website.

## Processing flow

1. The scraper loads `.env` from the project directory if it exists.
2. It checks that the Groq API key is present and queries the available model
   list. It exits with an actionable error if the key or selected model is not
   accepted.
3. For each casino, it checks the source host's `robots.txt`. A denied URL or
   an unreadable robots file is skipped. A missing robots file (HTTP 404 or 410)
   is treated as having no published rules.
4. It downloads the page once per distinct URL, using a per-run cache. HTTP
   401, 403, and 429 responses are treated as blocked; the scraper does not
   attempt to bypass those restrictions.
5. It removes common non-content elements and sends at most 8,000 characters
   of page text to Groq. The prompt instructs the model not to guess and to
   treat page text as data, not instructions.
6. It requests JSON and validates the three extracted bonus fields. Rate-limit
   responses are retried briefly; invalid model responses are reported for that
   casino, and the remaining casinos can still be processed.
7. It writes results to `bonuses.json` beside `scrape.py`, regardless of the
   terminal's current directory.

The JSON entries have this shape:

```json
{
  "casino": "Example Casino",
  "welcome_bonus": "100% up to 500 EUR",
  "wagering_multiplier": 35,
  "min_deposit": "20 EUR",
  "source_url": "https://example.com/bonuses"
}
```

Unavailable bonus values are represented as `null`. `wagering_multiplier` is
stored as a non-negative number or `null`.

## View the results

The page uses `fetch("bonuses.json")`, so serve the project directory over HTTP
rather than opening the HTML file directly:

```powershell
python -m http.server 8000
```

Then open <http://localhost:8000/>. Alternatively, use VS Code Live Server.
Run the scraper before opening the page; until `bonuses.json` exists, the page
shows an explanatory error.

## Troubleshooting

| Message or symptom | What to check |
|---|---|
| `Missing GROQ_API_KEY` | Add a valid Groq key to `GROQ_API_KEY` in the project's `.env` file. |
| `Groq rejected the API key` | Check that the key is active and copied correctly; never paste it into source code. |
| `Groq model '…' is unavailable` | Set `GROQ_MODEL` to an ID available to your Groq account. |
| Groq returns HTTP 429 | The scraper makes a few short retries. If the limit persists, wait for the account quota window to reset or check your Groq limits. |
| Groq returns HTTP 429 | The scraper makes a few short retries. If the limit persists, wait for the account quota window to reset or check your Groq limits. |
| A casino is skipped for robots rules | Use a different public source that permits access; do not bypass the site's rules. |
| Blocked response or request failure | Check the URL and site availability; if the source blocks automated access, select another source. |
| Model response is rejected | Choose an instruction-following Groq model that supports JSON response mode. |
| Page says it cannot load `bonuses.json` | Run the scraper successfully and serve the same project folder over HTTP. |
| The page loads but shows `n/a` | Check the source text and model output; missing fields are intentionally not guessed. |

## Limitations and review checklist

- The three casino entries currently all point to the same comparison-page URL.
  Verify that the page actually includes current terms for each named casino;
  otherwise change the source or casino list before presenting any results.
- Pages that populate content with browser-side JavaScript may not expose the
  bonus text to this HTTP scraper.
- Up to the first 8,000 characters are sent to Groq. Avoid using this with
  pages that contain sensitive data, and review the provider's data terms.
- Groq request limits may cause some casino records to fail during a run. Check
  the console output and rerun after the quota window resets if needed.
- A valid JSON response does not guarantee that the extracted offer is correct.
  Compare each value with the original terms and record the actual result before
  describing model mistakes in the written submission.
- The scraper writes its output only after processing all configured sources.
  If all sources are skipped, the output is an empty JSON array.
- The browser page validates response status and basic JSON structure, but the
  source data still needs human review.
- The production proposal in `WRITTEN_PART.md` is an estimate, not a quote;
  verify current provider prices and operational costs before submitting it.

Before submission:

1. Configure a valid Groq API key and model, then run the scraper.
2. Check `bonuses.json` against the source page; do not invent results.
3. Open the page over HTTP and check desktop and narrow-screen layouts.
4. Complete the personal AI-use and time-taken sections in `WRITTEN_PART.md`
   truthfully. Do not leave example text or placeholders in the submitted copy.
5. Include the source files and a short approach note with the submission.

## Brief approach note

> The scraper checks the source site's robots rules before fetching the page,
> extracts visible text, and uses Groq's language-model API to structure the
> welcome-bonus details as JSON. It validates the model's response and writes a
> file that a responsive HTML page displays. I kept the implementation small,
> avoided bypassing access controls, and documented the need to verify extracted
> values against the original offer.

Edit this note if it does not accurately describe the version you submit.
