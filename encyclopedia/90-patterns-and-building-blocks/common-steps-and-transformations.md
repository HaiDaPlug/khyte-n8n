# Common Steps and Transformations

Reusable transformation patterns that appear in many automations.

## Data Extraction and Parsing

**Pattern**: Extract structured data from unstructured sources

**Common applications**:
- Email parsing (extract order number, amount from email body)
- PDF data extraction (invoice details, contract terms)
- Web scraping (prices, contact info, event details)
- Natural language processing (sentiment, entities, intent)

**Implementation approaches**:
- **Regex patterns**: Fast, predictable, brittle (breaks when format changes)
- **AI/LLM extraction**: Flexible, handles variations, costs money
- **Structured APIs**: Most reliable when available
- **Hybrid**: Try structured first, fall back to AI if fails

**Best practices**:
- Validate extracted data (does it make sense?)
- Handle missing data gracefully (what if amount isn't in email?)
- Log failures for pattern improvement
- Keep extraction logic separate from business logic

**Example**:
```
Extract from email:
"Your invoice #INV-1234 for $500.00 is due on 2024-03-15"
→ {invoice: "INV-1234", amount: 500.00, due_date: "2024-03-15"}
```

---

## Data Enrichment

**Pattern**: Enhance existing data with additional information from external sources

**Common applications**:
- Contact enrichment (email → full profile, company, title)
- Company enrichment (domain → size, industry, revenue)
- Location enrichment (address → lat/long, timezone, demographics)
- Content enrichment (URL → metadata, preview, keywords)

**Implementation approaches**:
- **Free APIs**: Google, OpenStreetMap (limited data, rate limits)
- **Paid APIs**: Clearbit, Apollo, Hunter (comprehensive, costs per lookup)
- **Web scraping**: LinkedIn, company websites (legal gray area, fragile)
- **AI inference**: Use LLM to infer missing data (less accurate, creative)

**Best practices**:
- Cache enriched data (don't re-enrich same record)
- Respect rate limits and quotas
- Store data source and enrichment date (for freshness tracking)
- Handle API failures gracefully (partial data > no data)

**Example**:
```
Input: {email: "john@acme.com"}
Enrichment APIs →
Output: {
  email: "john@acme.com",
  name: "John Smith",
  title: "VP Sales",
  company: "Acme Corp",
  company_size: "50-200",
  industry: "SaaS"
}
```

---

## Data Validation and Cleaning

**Pattern**: Verify data quality and fix common issues

**Common applications**:
- Email validation (format, domain exists, not disposable)
- Phone number normalization (format consistently)
- Address validation (real address, correct format)
- Duplicate detection (same person entered twice)

**Implementation approaches**:
- **Format validation**: Regex checks (fast, catches obvious errors)
- **Existence validation**: API checks (email has MX record, phone is real number)
- **Normalization**: Convert to standard format (US phone → +1-555-123-4567)
- **Deduplication**: Hash comparison, fuzzy matching

**Best practices**:
- Validate early (at data entry, not days later)
- Provide clear error messages
- Auto-fix when possible (trim whitespace, title case names)
- Quarantine bad data, don't delete (might be recoverable)

**Example validation rules**:
- Email: Valid format AND domain has MX record AND not in disposable email list
- Phone: Valid format AND correct country code AND not obviously fake (555-1212)
- Amount: Positive number AND has exactly 2 decimals (for currency)

---

## Conditional Logic and Routing

**Pattern**: Make decisions and route data to different paths

**Common applications**:
- Lead routing (size → enterprise rep, geography → local rep)
- Approval routing (amount → approver tier)
- Priority assignment (keywords → urgent vs normal)
- Content categorization (topic → department)

**Implementation approaches**:
- **Rule-based**: IF/THEN conditions (transparent, maintainable)
- **Lookup tables**: Match input to predefined mappings (flexible, non-technical can edit)
- **AI classification**: Machine learning or LLM (handles ambiguity, needs training)
- **Scoring systems**: Calculate score, route based on threshold

**Best practices**:
- Start with simple rules, add complexity only when needed
- Document decision logic clearly
- Provide default/fallback path (what if nothing matches?)
- Log which path was taken and why

**Example**:
```
IF company_size > 200 AND deal_value > $50k
  → Route to enterprise team
ELSE IF location in ["CA", "NY", "TX"]
  → Route to named account rep
ELSE
  → Round-robin to general team
```

---

## Data Aggregation and Summarization

**Pattern**: Combine multiple data points into summary or report

**Common applications**:
- Daily/weekly summaries (aggregate metrics from multiple sources)
- Dashboard updates (compile KPIs)
- Report generation (sales performance, campaign results)
- Trend analysis (compare periods, identify changes)

**Implementation approaches**:
- **Database queries**: SQL aggregations (fast, powerful)
- **API data pulls**: Collect from multiple services, combine
- **Spreadsheet formulas**: Sum, average, pivot
- **AI summarization**: Natural language summary of data

**Best practices**:
- Cache intermediate results (don't re-query same data)
- Handle missing data (what if one source fails?)
- Visualize when helpful (charts > tables of numbers)
- Include context (not just "500" but "500 (up 20% from last week)")

**Example**:
```
Aggregate:
- Revenue: $50k (Google Ads) + $30k (Organic) + $20k (Referral)
- Total leads: 100 (conversion rate 15%)
Summary: "Generated $100k revenue from 100 leads. Google Ads was top source (50%). Conversion rate improved 3% vs last month."
```

---

## Text Generation and Formatting

**Pattern**: Create formatted text output from data

**Common applications**:
- Email composition (personalized outreach, notifications)
- Report formatting (structured documents)
- Message creation (Slack, SMS)
- Document generation (contracts, proposals)

**Implementation approaches**:
- **Templates with variables**: "Hello {{name}}, your invoice {{id}} for {{amount}}..."
- **Conditional sections**: Include paragraph only if condition met
- **AI generation**: LLM writes entire message from data/context
- **Markdown/HTML**: Structured formatting

**Best practices**:
- Keep templates separate from logic (easier to edit)
- Preview before sending to customer
- Personalize beyond just {{FirstName}} (reference real context)
- Test with real data (edge cases: long names, special characters)

**Example template**:
```
Hi {{first_name}},

{{#if returned_customer}}
Welcome back! We're excited to work with you again.
{{else}}
Thanks for choosing us!
{{/if}}

Your {{service}} project is scheduled for {{date}}.
{{#if needs_preparation}}
Please complete the pre-project questionnaire by {{prep_deadline}}.
{{/if}}

Best,
Team
```

---

## Error Handling and Retries

**Pattern**: Gracefully handle failures

**Common applications**:
- API call failures (service down, rate limited)
- Data quality issues (missing required field)
- External dependency unavailable (website unreachable)
- Unexpected formats (data structure changed)

**Implementation approaches**:
- **Retry with backoff**: Try again after 1min, 5min, 15min
- **Fallback to alternative**: If API fails, use cached data or simpler method
- **Human escalation**: Create task for manual review
- **Graceful degradation**: Continue with partial data

**Best practices**:
- Log all errors with context (what failed, why, what data)
- Don't retry forever (max 3 attempts)
- Alert humans for critical failures
- Distinguish temporary vs permanent failures

**Retry pattern**:
```
Try:
  Call API
If fails:
  Wait 60 seconds
  Try again
If fails again:
  Wait 300 seconds
  Try again
If fails third time:
  Alert human
  Create manual task
  Stop retrying
```

---

## Scheduling and Delays

**Pattern**: Time-based workflows and waiting

**Common applications**:
- Follow-up sequences (wait 3 days, send email)
- Reminder systems (send reminder 24 hours before)
- Batch processing delays (collect data for 1 hour, then process)
- Rate limiting (don't send more than 1 email per minute)

**Implementation approaches**:
- **Platform delays**: Built-in "wait X time" steps
- **Scheduled tasks**: Create future task, run at specific time
- **Queue systems**: Add to queue with delay parameter
- **Database + cron**: Store future action, cron job executes when due

**Best practices**:
- Be explicit about timezone
- Handle cancellations (if trigger condition changes, don't send)
- Account for weekends/holidays (don't send at 2am Saturday)
- Allow override (urgent case needs immediate send)

**Example sequence**:
```
1. Email sent
2. Wait 3 days
3. Check if replied
4. If no reply, send follow-up
5. Wait 7 days
6. If still no reply, create task for manual outreach
```

---

## Pattern Composition

Most real automations combine multiple patterns:

**Example: Lead capture and routing**
1. **Trigger**: Form submitted (webhook)
2. **Extraction**: Parse form data
3. **Validation**: Verify email format, required fields
4. **Enrichment**: Look up company data
5. **Scoring**: Calculate lead score
6. **Conditional routing**: Route based on score and geography
7. **Notification**: Send to assigned rep
8. **Logging**: Record in CRM

The art is knowing which patterns to use, in what order, with appropriate error handling at each step.
