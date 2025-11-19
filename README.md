# khyte-n8n

**Blueprint library for reusable n8n workflows and subflows**

This repository contains documented patterns, specifications, and example configurations for n8n automation workflows designed for AI-powered consulting and agency work.

## What is this?

`khyte-n8n` is a collection of ready-to-use workflow blueprints that you can adapt and deploy in n8n for yourself or your clients. It's **not** a running n8n instance — it's a library of battle-tested patterns with clear documentation.

Think of it as your automation cookbook: pick a recipe, follow the spec, customize for your needs, and deploy.

## What's inside?

This repository contains two major components:

### 1. n8n Workflow Blueprints

**4 Reusable Subflows**

Building blocks you can plug into any workflow:

- **safe-http-call** — Resilient HTTP calls with retries, timeouts, and normalized error handling
- **ai-call-wrapper** — Consistent AI API calls with JSON parsing and error recovery
- **notification-dispatch** — Standard notifications via Slack/email with severity levels
- **audit-log** — Simple event logging to sheets/databases for workflow tracking

**10 Core Workflows**

End-to-end automation patterns for common business needs:

1. **Lead Enrichment Pipeline** — Enrich raw leads with AI + API data
2. **Weekly Report Autodraft** — Generate client performance reports automatically
3. **Meeting Summary & Tasks** — Turn transcripts into summaries and task lists
4. **Smart Inbox Triage** — AI-powered email routing and prioritization
5. **Document Intake to DB** — Extract structured data from PDFs and forms
6. **CRM & Sheet Sync** — Keep CRM and spreadsheets synchronized
7. **Ops Digest** — Daily/weekly operational summary emails
8. **LinkedIn Lead Organizer** — Score and prioritize exported LinkedIn leads (ToS-safe)
9. **Proposal & Followup Automation** — Never forget to follow up on proposals
10. **Sponsorship Local Research Helper** — Score potential sponsors for local orgs

### 2. Automation Encyclopedia

**[`encyclopedia/`](encyclopedia/)** — A comprehensive reference library of **218+ automation ideas** across 10 business domains.

Use the encyclopedia for:
- **Client discovery** — Identify automation opportunities during initial calls
- **Proposal building** — Bundle multiple automations into packages
- **Ideation** — Avoid blank page syndrome when designing new workflows
- **Cross-reference** — Find similar patterns when facing new problems

The encyclopedia is **tool-agnostic** (works with any automation platform), while the workflow blueprints are **n8n-specific** implementations. Together they provide both broad ideas and detailed execution plans.

## How to use this repo

1. **Browse** the `workflows/` or `subflows/` directories
2. **Read** the `spec.md` in the folder you're interested in
3. **Review** the `example.json` or `example-config.json` to see the n8n structure
4. **Adapt** the workflow to your specific tools and client needs
5. **Import** or rebuild it in your n8n instance
6. **Configure** environment variables and credentials (see `docs/environment-and-secrets.md`)

## Documentation

- [`docs/overview.md`](docs/overview.md) — Conceptual overview and usage guide
- [`docs/architecture-and-conventions.md`](docs/architecture-and-conventions.md) — Design patterns and naming conventions
- [`docs/environment-and-secrets.md`](docs/environment-and-secrets.md) — How to handle credentials and configuration

## Important notes

### This is a blueprint library, not a running system

You need your own n8n instance to use these workflows. This repo provides the **what** and **how**, not the infrastructure.

### Legal & ethical use

All workflows follow platform Terms of Service, especially:

- **No scraping** of LinkedIn or other platforms via headless browsers
- **Only use exported data** or publicly available information
- **Respect rate limits** and API usage policies

The LinkedIn workflow specifically uses CSV exports or manually curated lists — never automated scraping.

### Generic by design

Workflows reference generic services (e.g. "CRM", "Email provider", "Spreadsheet") with common examples. You choose the specific tools:

- CRM: HubSpot, Pipedrive, Salesforce, etc.
- Email: Gmail, Outlook, etc.
- Sheets: Google Sheets, Excel, Airtable, etc.
- Database: Supabase, PostgreSQL, MySQL, etc.

### Never commit secrets

All examples use environment variable placeholders like `AI_API_KEY`, `SLACK_WEBHOOK_URL`, etc. Configure these in your n8n credentials, never hardcode them.

## About

Created by **Hai** at **Khyte** (Borås, Sweden) for AI workflow and automation consulting.

These patterns are designed for small-to-mid-sized agencies and local businesses looking to save time, reduce errors, and eliminate manual chaos — without the hype.

## License

Feel free to use, adapt, and deploy these patterns for your own work or client projects.
