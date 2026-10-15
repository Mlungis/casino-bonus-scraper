# Casino Bonus Scraper

Scrape welcome-bonus information for three casinos, use Groq to structure the
results, save them as JSON, and view them in a responsive web page.

## Requirements

- Python 3.9 or newer
- A Groq API key and internet access
- A browser for viewing the results

No Ollama installation or local model download is required. The scraper sends
up to 8,000 characters of page text to Groq for extraction.

## Install

Open PowerShell in this project folder (the folder containing `scrape.py`) and
run:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

On macOS or Linux, use:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

## Configure Groq

1. Create an API key in your Groq account.
2. Copy `.env.example` to `.env` in the project folder.
3. Open `.env` and replace `your_groq_api_key` with your key.

The example file selects `openai/gpt-oss-20b` as the model. If that model is
not available to your account, replace `GROQ_MODEL` with an available model ID.

**Keep the key private.** `.env` is excluded by `.gitignore`; do not paste the
key into source code, commit it, or include it in screenshots. If you have
shared a key, revoke it in Groq and create a new one.

## Run the scraper

From the project folder:

```powershell
.\.venv\Scripts\python.exe .\scrape.py
```

On macOS or Linux:

```bash
.venv/bin/python scrape.py
```

The scraper checks the Groq key and model, checks the source page's
`robots.txt`, fetches the page once, and asks Groq to extract each casino's
welcome bonus, wagering multiplier, and minimum deposit. The source URL is
included with each result. It retries rate limits briefly and reports failures
without attempting to bypass access restrictions.

Successful results are written to `bonuses.json` beside `scrape.py`. Missing
values are `null`. Check each result against the actual casino terms; model
output can be incorrect.

## View the results

The page fetches `bonuses.json`, so run a local web server instead of opening
`index.html` directly. In a second PowerShell window, from the project folder,
run:

```powershell
.\.venv\Scripts\python.exe -m http.server 8000
```

Open <http://localhost:8000/> in your browser. Stop the server with `Ctrl+C`.
On macOS or Linux, run `.venv/bin/python -m http.server 8000` instead.

## Deploy the webpage to Vercel

The Vercel deployment serves only the static `index.html` and `bonuses.json`;
it does not run the Python scraper. The root `vercel.json` explicitly declares
those static files so Vercel does not try to start the Python project as a web
application.

If this project is inside a subfolder of your Git repository, set Vercel's
**Root Directory** to `casino-bonus-scraper` (the folder containing this
README, `index.html`, and `vercel.json`). Leave the Framework Preset as
**Other**, then redeploy. For updated bonus data, run the scraper locally and
commit the refreshed `bonuses.json` before deploying.

Do not add the Groq API key to Vercel: it is only needed when running the
scraper locally, and the deployment is limited to the static page and data.

## Project files

- `scrape.py` — scraper and Groq integration
- `bonuses.json` — generated results
- `index.html` — responsive results page
- `vercel.json` — deploys the page and JSON as static files
- `requirements.txt` — Python dependencies
- `.env.example` — Groq settings template; copy to the ignored `.env`
- `PROJECT_GUIDE.md` — detailed processing notes, troubleshooting, and review checklist
- `WRITTEN_PART.md` — assessment response and production discussion

## Troubleshooting

- **Python command not found:** Install Python 3.9+ and reopen the terminal.
- **Missing `GROQ_API_KEY`:** Check that `.env` exists beside `scrape.py` and
  contains your key.
- **Groq rejected the key:** Check it in your Groq account. Revoke and replace
  any key that has been exposed.
- **Model unavailable:** Set `GROQ_MODEL` in `.env` to a model enabled for your
  account.
- **HTTP 429:** Groq rate limits apply. The scraper retries a few times; if the
  limit persists, wait and try again later.
- **Page cannot load `bonuses.json`:** Run the scraper successfully, then make
  sure the web server is serving this same project folder.
- **Casino skipped or blocked:** Respect the site's robots rules and access
  restrictions; choose another permitted public source if necessary.

## Important limitations

All three configured casinos currently use the same comparison-page URL. Verify
that it contains current information for each casino before relying on the
results. Review the generated values against the source page before submission.
For further details, see [PROJECT_GUIDE.md](./PROJECT_GUIDE.md).
