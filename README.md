# Gupy Scraper (Python): Gupy vagas to JSON and CSV

[![Run on Apify](https://img.shields.io/badge/Run%20on-Apify-00A67E?logo=apify&logoColor=white)](https://apify.com/data_daemon/gupy-jobs-scraper)
![Output](https://img.shields.io/badge/output-JSON%20%7C%20CSV%20%7C%20Excel-blue)
![License: MIT](https://img.shields.io/badge/license-MIT-green)

Scrape **Gupy job postings (vagas)** into JSON or CSV from Python: title, company and career page, city and state, workplace type (presencial, híbrido, remoto), job type (CLT, estágio, PJ, jovem aprendiz), PcD flag, posting date, application deadline and the **full description**. Search by term, state and city, or pull the whole portal.

> **Em português:** extraia as **vagas da Gupy** em JSON ou CSV com Python: título, empresa, cidade, estado, modelo de trabalho, tipo de vaga, vagas PcD, prazo de inscrição e descrição completa.

[Gupy](https://portal.gupy.io) is Brazil's largest applicant-tracking system and job portal (portal.gupy.io). This repository is a small, tested Python client for the hosted
**[Gupy Jobs Scraper | Brazil Jobs with Descriptions (Gupy Portal)](https://apify.com/data_daemon/gupy-jobs-scraper)** on Apify: no browser, proxy, login or scraper maintenance on your side. Price:
**$0.80 per 1,000 jobs, description included, plus $0.001 per run**; Apify's free plan ($5 of credit every month) covers about 6,000 jobs.

## Quick start

```bash
pip install apify-client
export APIFY_TOKEN=your_token      # Apify Console > Settings > Integrations
python scrape.py --keyword "analista de dados" --max 50
```

`scrape.py` starts the Actor, waits for it and writes `gupy-jobs.json` (every field) and `gupy-jobs.csv`
(the main columns). For any other option, put the Actor input in a file and pass `--input`:

```bash
python scrape.py --input examples/input.json
```

```json
{
  "searchTerms": [
    "analista de dados"
  ],
  "maxResults": 5
}
```

## Sample output

A real run of [`examples/input.json`](examples/input.json) on 5 November 2026
([full JSON](examples/sample-output.json), [CSV](examples/sample-output.csv); descriptions shortened):

| postedAt | title | company | city | workplaceType | jobType |
|---|---|---|---|---|---|
| 2026-11-04 | ANALISTA DE COMPRAS SR DADOS - SALVADOR/BA | LM Mobilidade | Salvador | on-site | effective |
| 2026-11-04 | ANALISTA DE ORÇAMENTO E DADOS JR | @novarotadooeste | Cuiabá | on-site | effective |
| 2026-11-04 | Analista de Dados Pleno  - Inteligência de Mercado | ICONIC | Rio de Janeiro | hybrid | effective |
| 2026-11-04 | 22334-Analista de Dados e BI | Hitss Brasil | São Paulo | on-site | effective |
| 2026-11-04 | PESSOA ANALISTA DE PROTEÇÃO DE DADOS JR | Hospital Alemão Oswaldo Cruz - Vagas Corporativas, Administr | São Paulo | on-site | effective |

## Output fields

| Field | Description |
|---|---|
| `id` | Gupy job id (stable; use it for de-duplication and skipIds). |
| `title` | Job title as posted. |
| `url` | Public job page on the company's Gupy career site (apply there). |
| `company` | Company (Gupy career page name). |
| `companyUrl` | The company's Gupy career site. |
| `companyLogo` | Company logo URL. |
| `careerPage` | Career-site slug (the subdomain of the job URL), e.g. vemproitau. |
| `jobType` | Gupy vacancy type: effective (regular CLT job), internship, apprentice, temporary, legal_entity (PJ), freelancer, autonomous, lecturer, associate, talent_pool, volunteer. |
| `jobTypeRaw` | The vacancy type as Gupy sends it, e.g. vacancy_type_effective. |
| `workplaceType` | on-site, hybrid or remote. |
| `isRemote` | True for remote (home office) jobs. |
| `city` | City, when the employer set one. |
| `state` | Brazilian state, when the employer set one. |
| `locationText` | "City, State", or "Remote" for remote jobs without a place. |
| `acceptsPwd` | The employer marked the job as open to people with disabilities (PcD). |
| `postedAt` | When the job was published on Gupy (ISO 8601, UTC). |
| `applicationDeadline` | Last day to apply (ISO 8601, UTC), when set. |
| `description` | Full description as plain text (null when includeDescription is off). |
| `searchTerm` | The keyword that found the job (null for a no-keyword run). |
| `scrapedAt` | When this run saved the job (ISO 8601, UTC). |

## Daily feed of only new jobs

```bash
python scrape.py --keyword "analista de dados" --only-new gupy-daily --max 0
```

With `--only-new <name>` the Actor remembers which jobs it has delivered under that name and returns only new ones;
skipped jobs are not charged. Run it from cron, a GitHub Actions schedule or an Apify schedule:

```yaml
# .github/workflows/gupy-daily.yml
on:
  schedule: [{cron: "30 2 * * *"}]
  workflow_dispatch:
jobs:
  scrape:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install apify-client && python scrape.py --keyword "analista de dados" --only-new gupy-daily --max 0
        env: {APIFY_TOKEN: "${{ secrets.APIFY_TOKEN }}"}
      - uses: actions/upload-artifact@v4
        with: {name: gupy-jobs, path: "gupy-jobs.*"}
```

## Other ways to call it

**curl**, one request that runs the Actor and returns the rows:

```bash
curl -X POST "https://api.apify.com/v2/acts/data_daemon~gupy-jobs-scraper/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"searchTerms": ["analista de dados"], "maxResults": 5}'
```

**Apify CLI:** `apify call data_daemon/gupy-jobs-scraper --input-file examples/input.json`

**No code:** open the [Actor page](https://apify.com/data_daemon/gupy-jobs-scraper), fill the form and export to CSV/Excel, or connect it to Google Sheets,
Zapier, Make or n8n. AI agents can call it through the Apify MCP server (`https://mcp.apify.com`).

## FAQ

### Does Gupy have a public jobs API?

Gupy's developer API is for employers who use Gupy as their ATS (their own jobs and candidates). Searching every company's vagas needs the public portal, which is what the hosted Actor behind this repo reads.

### How many jobs are on Gupy?

On 30 October 2026 a full pull returned 9,879 jobs from 1,571 employers published over four days; about 2,700 new jobs a day.

### Can I filter PcD, estágio or remote jobs?

Yes: `pwdOnly`, `jobTypes` (`internship` for estágio, `effective` for CLT, `legal_entity` for PJ) and `workplaceTypes` (`remote`, `hybrid`, `on-site`) in the input.

### Is it legal to scrape Gupy?

The Actor reads only public job listings, without logging in, at a polite request rate, and returns job and company
data, not personal profiles. Check the rules that apply to your use of the data in your jurisdiction.

## Related

- [Zap Imóveis & VivaReal Scraper | Brazil Property Listings API](https://apify.com/data_daemon/zap-imoveis-vivareal-scraper)
- [Welcome to the Jungle Jobs Scraper | WTTJ API & New Job Feed](https://apify.com/data_daemon/welcome-to-the-jungle-jobs-scraper)
- [Career Site Jobs API – Workday, Greenhouse, Lever, Ashby](https://apify.com/data_daemon/career-site-jobs-scraper)
- [India Job Alerts](https://github.com/datadmon/india-job-alerts): new jobs from seven Indian boards every morning via GitHub Actions
- [All Daemon Data Actors](https://apify.com/data_daemon)

Unofficial project, not affiliated with Gupy. Issues and questions: open an issue here or on the
[Actor's Issues tab](https://apify.com/data_daemon/gupy-jobs-scraper/issues).
