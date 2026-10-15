# Written part

## Taking this to production: 200 casinos, updated daily

**Where it runs:** One small VPS (for example Hetzner, about 4 GB RAM) running
Ubuntu. It is cheap, simple, and I control everything on it. The site is mostly
static pages plus one daily job, so I don't need a big cloud setup. If traffic
grows I can add a bigger server or a CDN later.

**Database:** PostgreSQL on the same server (or a managed one if the budget
allows). I would store casinos, bonuses, and a history of each scrape so I can see
changes over time. Backups: a nightly `pg_dump` copied to off-server storage
(such as Backblaze B2 or S3), keep 14 days, and test restoring a backup
from time to time.

**Domain, DNS, HTTPS:** Buy the domain at a registrar, point DNS to Cloudflare
(free), and use Let's Encrypt (Caddy or Certbot) so HTTPS renews automatically.

**Daily scraper run:** A cron job on the server runs the scraper every night.
Each casino is scraped on its own, so one failure does not stop the others.
To know when it fails: the job pings Healthchecks.io when it finishes, and if
no ping arrives I get an email. I would also log how many casinos succeeded and
flag any result with a missing or strange value (for example no wagering number),
since the language model can make mistakes.

**Language model assumption:** The prototype uses Groq's hosted API, so the
scraper needs a Groq API key and internet access. The cost estimate below
assumes a production plan is selected and actual model pricing is checked before
launch. Avoid sending sensitive page content to a third party unless the
provider's data terms and the site's terms allow it.

The current screening-task version makes one model request per casino. At larger
scale, I would avoid repeating calls for unchanged page content and confirm the
provider's rate limits and pricing against the expected daily workload.

**Rough monthly cost:**

| Item | Cost |
|---|---|
| VPS | about 5-10 EUR |
| Backup storage | about 1-2 EUR |
| Domain | about 1-2 EUR (yearly fee spread out) |
| Language model calls (200 casinos/day, only re-reading pages that changed) | about 5-15 EUR |
| Cloudflare, Let's Encrypt, Healthchecks (free tiers) | 0 EUR |
| **Total** | **about 15-30 EUR per month** |

*(These are estimates. Check current prices before quoting.)*

---

# Brief approach note

> The scraper checks the source site's robots rules before fetching the page,
> extracts visible text, and uses Groq's language-model API to structure the
> welcome-bonus details as JSON. It validates the model's response and writes a
> file that a responsive HTML page displays. I kept the implementation small,
> avoided bypassing access controls, and documented the need to verify extracted
> values against the original offer.

Edit this note if it does not accurately describe the version you submit.

## AI-use disclosure

1. **Tool used:** Copilot in VS Code (an AI coding assistant).
2. **What I used it for:** I used it to help implement and document the scraper,
   connect the extraction step to Groq, and test the generated results.
3. **One mistake observed:** The first Groq model ID configured in the project,
   `llama-3.3-70b-versatile`, was not available to the Groq account used for the
   test.
4. **How I verified it:** I queried Groq's authenticated models endpoint. That
   model ID was absent from the returned list; `openai/gpt-oss-20b` was present.
   I then verified the replacement with a live extraction request and a full
   scraper run.
5. **What I changed:** I changed the default model to `openai/gpt-oss-20b`,
   updated the local `.env` model setting and setup documentation, and added a
   startup check that confirms the configured model is available before
   scraping.

## Time taken

I did not track the time closely enough to give a reliable duration. Before
submission, the generated bonus values still need to be checked against the
casinos' current terms; the shared Groq API key should also be revoked and
replaced.
