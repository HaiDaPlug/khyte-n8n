# Overview — khyte-n8n

## What is khyte-n8n?

`khyte-n8n` is a library of **blueprints and patterns** for n8n automation workflows. It's designed to help you (or your clients) quickly implement proven automation patterns without starting from scratch every time.

This is **not** a running n8n instance. Think of it as a cookbook:

- Each workflow is a recipe
- Each subflow is a reusable technique
- The docs explain the ingredients and methods
- You bring your own kitchen (n8n instance) and customize to taste

## Who is this for?

This library is built for:

- **AI/automation consultants** who need to deliver workflows quickly for multiple clients
- **Agency owners** who want to automate recurring operations
- **Small business operators** drowning in manual admin and communication
- **Anyone** who uses n8n and wants battle-tested patterns instead of reinventing wheels

## Two types of components

### 1. Subflows (Reusable Building Blocks)

Subflows are **small, focused patterns** you can plug into any workflow. They solve common problems like:

- Making HTTP calls safely with retries
- Calling AI APIs consistently
- Sending notifications
- Logging events

**Location:** `subflows/`

**What's in each subflow folder:**

- `spec.md` — What it does, when to use it, inputs/outputs, behavior
- `example.json` — A sample n8n export showing the node structure

**Current subflows:**

1. **safe-http-call** — Resilient HTTP requests with retry logic and error normalization
2. **ai-call-wrapper** — Consistent AI API calls with JSON mode and parse recovery
3. **notification-dispatch** — Standard Slack/email notifications with severity levels
4. **audit-log** — Simple event logging to track workflow execution

### 2. Workflows (End-to-End Use Cases)

Workflows are **complete automation scenarios** that solve real business problems. Each one combines multiple steps, often using subflows as building blocks.

**Location:** `workflows/`

**What's in each workflow folder:**

- `spec.md` — Full specification including:
  - Summary and use cases
  - Input requirements and dependencies
  - High-level flow diagram (text)
  - Node-by-node implementation outline
  - Where subflows are used
  - Customization tips for different clients/tools
- `example-config.json` — A sample n8n workflow export showing structure and wiring

**Current workflows:**

1. **Lead Enrichment Pipeline** — Take raw leads and enrich with AI + API data
2. **Weekly Report Autodraft** — Generate performance reports for clients automatically
3. **Meeting Summary & Tasks** — Turn meeting transcripts into summaries and task lists
4. **Smart Inbox Triage** — AI-powered email classification and routing
5. **Document Intake to DB** — Extract structured data from PDFs and forms
6. **CRM & Sheet Sync** — Keep CRM and spreadsheets in sync
7. **Ops Digest** — Daily/weekly operational summaries
8. **LinkedIn Lead Organizer** — Score and prioritize exported LinkedIn leads (ToS-safe)
9. **Proposal & Followup Automation** — Never miss a proposal follow-up
10. **Sponsorship Local Research Helper** — Score potential sponsors for local organizations

## How to use this library

### Step 1: Identify your need

Browse the `workflows/` directory or the workflow list above. Find the pattern that matches what you're trying to automate.

**Not sure which workflow fits?** Read the Summary and Use Cases sections in each `spec.md`.

### Step 2: Read the spec

Open the workflow's `spec.md` and read through:

- What it does
- What inputs it needs (sheets, APIs, email, etc.)
- How it works (high-level flow)
- Where you can customize it

### Step 3: Review the example

Look at the `example-config.json` to see:

- What n8n nodes are used
- How they're connected
- Where environment variables and credentials go

This is **illustrative**, not a perfect working export. Use it as a reference to understand the structure.

### Step 4: Identify subflows

Check which subflows the workflow uses (listed in the spec). You may want to build those subflows first as reusable components in your n8n instance.

### Step 5: Build in n8n

Now you're ready to implement:

1. Open your n8n instance
2. Create a new workflow
3. Add nodes following the spec
4. Configure credentials and environment variables (see `docs/environment-and-secrets.md`)
5. Connect nodes as shown in the example JSON
6. Test with sample data
7. Adjust for your specific tools and needs

### Step 6: Customize

Every workflow is designed to be **generic**. You'll need to:

- Choose specific tools (which CRM? which email provider? which sheet tool?)
- Adjust field mappings
- Tweak AI prompts
- Set trigger schedules

The spec's **Customization Tips** section will guide you.

## Design philosophy

### Generic by default

Workflows don't assume you use HubSpot or Gmail or Google Sheets. They describe patterns like:

- "Fetch from CRM" (could be HubSpot, Pipedrive, Salesforce, etc.)
- "Send email" (could be Gmail, Outlook, SendGrid, etc.)
- "Write to sheet" (could be Google Sheets, Excel, Airtable, etc.)

This makes patterns reusable across different client stacks.

### Environment variables for everything

Never hardcode:

- API keys
- Webhook URLs
- Email addresses
- Database credentials

All examples use placeholders like `AI_API_KEY`, `SLACK_WEBHOOK_URL`, etc. Configure these in n8n's credentials system.

### Subflows for common patterns

Instead of repeating "HTTP call with retry logic" in every workflow, we abstract it into `safe-http-call`. This means:

- Less duplication
- Consistent error handling
- Easier maintenance

### ToS-compliant

All workflows respect platform Terms of Service. Specifically:

- **No scraping** via headless browsers (Puppeteer, Playwright, etc.)
- **No bypassing** login, captcha, or rate limits
- **Only use** exported data or official APIs

The LinkedIn workflow, for example, uses CSV exports or manually curated lists — never automated scraping.

## When to use each type

### Use a subflow when...

- You need a small, focused piece of functionality
- You'll use the same pattern in multiple workflows
- You want to standardize error handling, retries, or formatting

**Example:** Every workflow that calls an AI API should use `ai-call-wrapper` instead of building it from scratch each time.

### Use a full workflow when...

- You have a complete business process to automate
- You need multiple steps coordinated together
- You're solving a specific use case end-to-end

**Example:** Automating proposal follow-ups requires tracking, scheduling, drafting emails, and updating status — that's a full workflow.

## Adapting for clients

Most workflows need customization. Here's what typically changes:

**Data sources:**
- Sheet URL/range
- CRM endpoint
- Database connection
- Email inbox/label

**Tools:**
- Swap Google Sheets for Excel
- Swap Slack for email
- Swap HubSpot for Pipedrive

**Business logic:**
- Field mappings (your "lead score" might be different from client to client)
- AI prompts (tone, language, specific instructions)
- Trigger timing (daily vs weekly, morning vs evening)

**Output destinations:**
- Where results go (sheet, CRM, email, Notion, etc.)

The spec's **Customization Tips** will highlight the most common adaptation points.

## What's NOT in this repo

**This repo does NOT include:**

- A running n8n instance
- Pre-configured credentials
- Real data or secrets
- Platform-specific implementations (you choose the tools)
- Perfect, production-ready JSON (the examples are illustrative)

**You need to bring:**

- Your own n8n instance (cloud or self-hosted)
- Access to the tools you want to integrate (CRM, email, sheets, etc.)
- API keys and credentials
- Your specific business logic and data

## Getting help

If something's unclear:

1. Check the workflow's `spec.md` for details
2. Review `docs/architecture-and-conventions.md` for patterns
3. Look at `docs/environment-and-secrets.md` for credential setup
4. Examine the `example.json` to see node structure

Still stuck? Most n8n patterns are well-documented in the [n8n documentation](https://docs.n8n.io) and community forum.

## Philosophy: calm, practical automation

These workflows are built with a simple goal: **save time, reduce errors, eliminate manual chaos.**

No hype. No "10x your productivity" nonsense. Just practical patterns that work for real businesses dealing with real problems.

If a workflow saves you 30 minutes a day, that's a win. If it catches leads you would have missed, that's a win. If it makes sure you never forget a follow-up, that's a win.

That's the standard.
