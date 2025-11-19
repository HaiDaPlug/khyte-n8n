# Workflow: Sponsorship Local Research Helper

## Summary

Automates research for local business sponsorship opportunities and community partnerships. Searches for events, organizations, and initiatives in your target area, evaluates fit based on your criteria, and compiles research reports with contact information and outreach suggestions. Perfect for businesses looking to build local presence through strategic sponsorships.

**Time saved:** 3-4 hours weekly → 20 minutes reviewing opportunities
**Best for:** Local businesses, consultancies building regional presence, community-focused brands

## Use Cases

1. **Local event sponsorships**
   - Find upcoming business events in Borås/region
   - Research organizer credibility and audience
   - Suggest sponsorship tier and outreach approach

2. **Community organization partnerships**
   - Identify nonprofits and associations aligned with values
   - Evaluate partnership opportunities
   - Compile contact details and pitch angles

3. **Industry conference presence**
   - Track relevant conferences in target regions
   - Analyze speaker opportunities vs. booth vs. sponsorship
   - Budget and ROI estimation

## Inputs & Dependencies

**Required:**
- **Search parameters:** Location, industry focus, budget range
- **Web search access:** Google Custom Search API or similar
- **AI API access:** OpenAI or Anthropic (for evaluation and research)
- **Data sources:** One or more of:
  - Eventbrite API (events)
  - Meetup API (local groups)
  - Google search (general)
  - Local business directories
  - Social media APIs (Twitter/LinkedIn for event announcements)
- **n8n credentials:**
  - Google Custom Search API key
  - Eventbrite API token
  - OpenAI/Anthropic API key
  - Optional: Social media API credentials

**Search criteria:**
- Geographic area (city, region, radius)
- Event types or organization focus
- Budget range for sponsorships
- Target audience demographics
- Timing preferences (upcoming months)

## High-Level Flow

```
1. Trigger (scheduled weekly or manual)
   ↓
2. Define search parameters (location, event types, date range)
   ↓
3. Parallel research collection:
   3a. Search Eventbrite for local events
   3b. Search Meetup for networking groups
   3c. Google search for "[city] business events [month]"
   3d. Check local chambers of commerce
   3e. Search nonprofit directories
   ↓
4. Aggregate and deduplicate results
   ↓
5. For each opportunity:
   5a. Extract details (date, venue, organizer, audience)
   5b. Find contact information
   5c. Research organizer credibility
   5d. Call AI to evaluate:
       - Fit score for your business
       - Expected ROI/visibility
       - Sponsorship tier recommendation
       - Target audience alignment
       - Outreach strategy
   ↓
6. Filter and prioritize:
   6a. High fit (immediate action)
   6b. Medium fit (consider)
   6c. Low fit (archive)
   ↓
7. Compile research report:
   7a. Top opportunities with details
   7b. Budget estimates
   7c. Contact information
   7d. Suggested outreach messages
   ↓
8. Save to opportunities database
   ↓
9. Send weekly digest with top picks
   ↓
10. Log completion
```

## Node-by-Node Implementation

### 1. Trigger: Schedule Weekly (Schedule Trigger node)

```
Trigger: Cron - every Monday at 9:00 AM
Expression: 0 9 * * 1
```

### 2. Define Search Parameters (Function node)

```javascript
// Configure your search criteria
const searchConfig = {
  // Geographic area
  location: {
    city: process.env.TARGET_CITY || 'Borås',
    region: process.env.TARGET_REGION || 'Västra Götaland',
    country: 'Sweden',
    radius: 50,  // km
    coordinates: {
      lat: 57.7210,  // Borås coordinates
      lng: 12.9401
    }
  },

  // Time range
  dateRange: {
    start: new Date(),
    end: new Date(Date.now() + 90 * 24 * 60 * 60 * 1000)  // Next 90 days
  },

  // Event types
  eventTypes: [
    'business networking',
    'industry conference',
    'tech meetup',
    'startup event',
    'chamber of commerce',
    'community festival',
    'charity event'
  ],

  // Target industries/topics
  topics: [
    'automation',
    'AI',
    'digital transformation',
    'entrepreneurship',
    'local business',
    'sustainability'
  ],

  // Budget range
  sponsorshipBudget: {
    min: 5000,  // SEK
    max: 50000  // SEK
  },

  // Organization types
  organizationTypes: [
    'nonprofit',
    'business association',
    'industry group',
    'educational institution'
  ]
};

return {
  searchConfig,
  searchTimestamp: new Date().toISOString(),
  searchId: `research-${Date.now()}`
};
```

### 3a. Search Eventbrite (HTTP Request node)

```
Method: GET
URL: https://www.eventbriteapi.com/v3/events/search/
Headers:
  Authorization: Bearer {{ $env.EVENTBRITE_TOKEN }}
Parameters:
  location.latitude: {{ $('Define Search Parameters').item.json.searchConfig.location.coordinates.lat }}
  location.longitude: {{ $('Define Search Parameters').item.json.searchConfig.location.coordinates.lng }}
  location.within: {{ $('Define Search Parameters').item.json.searchConfig.location.radius }}km
  start_date.range_start: {{ $('Define Search Parameters').item.json.searchConfig.dateRange.start.toISOString() }}
  start_date.range_end: {{ $('Define Search Parameters').item.json.searchConfig.dateRange.end.toISOString() }}
  q: business OR networking OR tech
```

Parse Eventbrite results:

```javascript
const events = $input.item.json.events || [];
const searchConfig = $('Define Search Parameters').item.json.searchConfig;

const parsedEvents = events.map(event => ({
  source: 'eventbrite',
  type: 'event',
  name: event.name.text,
  description: event.description.text,
  url: event.url,
  startDate: event.start.local,
  endDate: event.end.local,
  venue: event.venue?.name,
  address: event.venue?.address?.localized_address_display,
  organizer: event.organizer?.name,
  organizerId: event.organizer_id,
  capacity: event.capacity,
  isFree: event.is_free,
  category: event.category?.name
}));

return parsedEvents;
```

### 3b. Search Meetup Groups (HTTP Request node)

```
Method: GET
URL: https://api.meetup.com/find/groups
Parameters:
  lat: {{ $('Define Search Parameters').item.json.searchConfig.location.coordinates.lat }}
  lon: {{ $('Define Search Parameters').item.json.searchConfig.location.coordinates.lng }}
  radius: {{ $('Define Search Parameters').item.json.searchConfig.location.radius }}
  topic_id: 1452 (technology), 21441 (entrepreneurship)
```

### 3c. Google Search for Local Events (HTTP Request - Google Custom Search)

```javascript
const searchConfig = $('Define Search Parameters').item.json.searchConfig;
const month = new Date().toLocaleDateString('sv-SE', { month: 'long' });

const searchQueries = [
  `${searchConfig.location.city} business events ${month}`,
  `${searchConfig.location.city} tech meetup sponsorship`,
  `${searchConfig.location.region} industry conference`,
  `${searchConfig.location.city} startup community events`,
  `${searchConfig.location.city} chamber of commerce events`
];

return searchQueries.map(query => ({ query }));
```

For each query:

```
Method: GET
URL: https://www.googleapis.com/customsearch/v1
Parameters:
  key: {{ $env.GOOGLE_SEARCH_API_KEY }}
  cx: {{ $env.GOOGLE_SEARCH_ENGINE_ID }}
  q: {{ $json.query }}
  dateRestrict: m3  // Last 3 months
  num: 10
```

### 3d. Search Nonprofit Databases (HTTP Request or Web Scraping)

**For Swedish organizations (Bolagsverket):**

```javascript
// Search for local nonprofits/associations
const searchTerms = [
  'automation',
  'digitalisering',
  'företagande',
  searchConfig.location.city
];

// Use official registry or directory
// Note: Respect robots.txt and ToS
```

### 4. Aggregate and Deduplicate Results (Function node)

```javascript
const eventbriteEvents = $('Search Eventbrite').all().map(e => e.json);
const meetupGroups = $('Search Meetup Groups').all().map(e => e.json);
const googleResults = $('Google Search for Local Events').all().flatMap(batch => batch.json.items || []);

// Combine all sources
let allOpportunities = [
  ...eventbriteEvents,
  ...meetupGroups,
  ...googleResults.map(result => ({
    source: 'google',
    type: 'search_result',
    name: result.title,
    description: result.snippet,
    url: result.link
  }))
];

// Deduplicate by URL or name similarity
const seen = new Set();
const deduplicated = allOpportunities.filter(opp => {
  const key = opp.url || opp.name.toLowerCase().trim();
  if (seen.has(key)) return false;
  seen.add(key);
  return true;
});

return {
  opportunities: deduplicated,
  totalFound: deduplicated.length,
  bySource: {
    eventbrite: eventbriteEvents.length,
    meetup: meetupGroups.length,
    google: googleResults.length
  }
};
```

### 5. Split Opportunities for Processing (Split In Batches node)

```
Batch Size: 10
Input: {{ $json.opportunities }}
```

### 6. Extract and Enrich Opportunity Details (Function node)

```javascript
const opportunity = $input.item.json;
const searchConfig = $('Define Search Parameters').item.json.searchConfig;

// Extract date if available
let eventDate = opportunity.startDate || null;

// Extract contact info from description or URL
const emailPattern = /[\w.-]+@[\w.-]+\.\w+/g;
const phonePattern = /(\+46|0)[\d\s-]{8,}/g;

const emails = (opportunity.description || '').match(emailPattern) || [];
const phones = (opportunity.description || '').match(phonePattern) || [];

// Estimate audience size
let estimatedAttendees = opportunity.capacity || null;
if (!estimatedAttendees && opportunity.description) {
  // Try to extract from description
  const attendeeMatch = opportunity.description.match(/(\d+)\s*(attendees|participants|deltagare)/i);
  if (attendeeMatch) {
    estimatedAttendees = parseInt(attendeeMatch[1]);
  }
}

return {
  ...opportunity,
  extractedEmails: emails,
  extractedPhones: phones,
  estimatedAttendees,
  eventDate,
  daysUntilEvent: eventDate ? Math.ceil((new Date(eventDate) - new Date()) / (1000 * 60 * 60 * 24)) : null
};
```

### 7. Research Organizer Credibility (HTTP Request + AI)

Fetch organizer website if available:

```javascript
// Try to find organizer website
const organizerWebsite = opportunity.organizerWebsite ||
                        opportunity.url?.match(/https?:\/\/[^\/]+/)?.[0];

if (!organizerWebsite) {
  return null;  // Skip if no website
}

return { url: organizerWebsite };
```

```
Method: GET
URL: {{ $json.url }}
Response Format: Text
Continue On Fail: Yes
```

**Use WebFetch tool to analyze organizer:**

```
URL: {{ $json.url }}
Prompt: Extract information about this organization:
- Organization name and type
- Mission/focus
- Credibility indicators (years active, partners, past events)
- Contact information
- Recent activity or news
Return concise summary.
```

### 8. Build AI Evaluation Prompt (Function node)

```javascript
const opportunity = $input.item.json;
const searchConfig = $('Define Search Parameters').item.json.searchConfig;
const organizerInfo = $('Research Organizer Credibility').item?.json || {};

const yourBusiness = {
  name: process.env.YOUR_BUSINESS_NAME || 'Your Company',
  focus: 'Automation consulting and AI implementation',
  targetAudience: 'SMBs, agencies, consultants',
  location: searchConfig.location.city,
  sponsorshipGoals: [
    'Build local brand awareness',
    'Generate qualified leads',
    'Position as automation experts',
    'Network with decision-makers'
  ],
  budget: searchConfig.sponsorshipBudget
};

const prompt = `Evaluate this sponsorship opportunity for ${yourBusiness.name}.

**Opportunity:**
- Name: ${opportunity.name}
- Type: ${opportunity.type}
- Date: ${opportunity.eventDate || 'Not specified'}
- Estimated attendees: ${opportunity.estimatedAttendees || 'Unknown'}
- Description: ${opportunity.description?.substring(0, 500)}
- Organizer: ${opportunity.organizer || 'Unknown'}

**Organizer Info:**
${organizerInfo.summary || 'Limited information available'}

**Your Business:**
- Focus: ${yourBusiness.focus}
- Target audience: ${yourBusiness.targetAudience}
- Location: ${yourBusiness.location}
- Sponsorship goals: ${yourBusiness.sponsorshipGoals.join(', ')}
- Budget range: ${yourBusiness.budget.min}-${yourBusiness.budget.max} SEK

**Evaluate and return JSON:**

{
  "fitScore": 1-10 (alignment with goals and audience),
  "fitReason": "Specific explanation of fit score",
  "audienceAlignment": "excellent|good|moderate|poor",
  "audienceDescription": "Who will attend and why they matter to you",
  "credibilityAssessment": "high|medium|low|unknown",
  "expectedROI": {
    "brandAwareness": "high|medium|low",
    "leadGeneration": "high|medium|low",
    "networking": "high|medium|low"
  },
  "sponsorshipTier": "title|platinum|gold|silver|bronze|booth|speaking|none",
  "suggestedBudget": number (in SEK),
  "budgetJustification": "Why this amount",
  "visibilityOpportunities": ["opportunity 1", "opportunity 2"],
  "risks": ["potential concern 1", "potential concern 2"],
  "outreachStrategy": "How to approach organizer (specific)",
  "pitchAngle": "One-sentence value proposition for this event",
  "priority": "high|medium|low",
  "recommendedAction": "Specific next step"
}

Be realistic about ROI. Consider audience size, quality, and cost-effectiveness.`;

return {
  prompt,
  opportunity,
  yourBusiness
};
```

### 9. Call AI for Opportunity Evaluation (AI Call Wrapper subflow)

```json
{
  "prompt": "={{ $json.prompt }}",
  "mode": "json",
  "systemInstructions": "You are a marketing strategist evaluating sponsorship opportunities. Be honest about fit and ROI. Don't oversell weak opportunities. Focus on strategic alignment and measurable outcomes.",
  "temperature": 0.3,
  "model": "gpt-4-turbo"
}
```

### 10. Extract and Merge Evaluation (Function node)

```javascript
const evaluation = $input.item.json.outputJson;
const opportunity = $('Build AI Evaluation Prompt').item.json.opportunity;

return {
  ...opportunity,
  // AI evaluation
  fitScore: evaluation.fitScore,
  fitReason: evaluation.fitReason,
  audienceAlignment: evaluation.audienceAlignment,
  audienceDescription: evaluation.audienceDescription,
  credibility: evaluation.credibilityAssessment,
  expectedROI: evaluation.expectedROI,
  sponsorshipTier: evaluation.sponsorshipTier,
  suggestedBudget: evaluation.suggestedBudget,
  budgetJustification: evaluation.budgetJustification,
  visibilityOpportunities: evaluation.visibilityOpportunities.join('; '),
  risks: evaluation.risks.join('; '),
  outreachStrategy: evaluation.outreachStrategy,
  pitchAngle: evaluation.pitchAngle,
  priority: evaluation.priority,
  recommendedAction: evaluation.recommendedAction,

  // Metadata
  evaluatedAt: new Date().toISOString(),
  status: 'evaluated'
};
```

### 11. Filter by Priority (Function node)

```javascript
const opportunities = $input.all();

const highPriority = opportunities.filter(o =>
  o.json.priority === 'high' &&
  o.json.fitScore >= 7 &&
  o.json.suggestedBudget <= 50000
);

const mediumPriority = opportunities.filter(o =>
  o.json.priority === 'medium' ||
  (o.json.fitScore >= 4 && o.json.fitScore < 7)
);

const lowPriority = opportunities.filter(o =>
  o.json.priority === 'low' ||
  o.json.fitScore < 4
);

return {
  highPriority,
  mediumPriority,
  lowPriority,
  summary: {
    total: opportunities.length,
    high: highPriority.length,
    medium: mediumPriority.length,
    low: lowPriority.length,
    totalBudgetNeeded: highPriority.reduce((sum, o) => sum + (o.json.suggestedBudget || 0), 0)
  }
};
```

### 12. Build Research Report (Function node)

```javascript
const { highPriority, mediumPriority, summary } = $input.item.json;
const searchConfig = $('Define Search Parameters').item.json.searchConfig;

const report = `# Sponsorship Research Report
${searchConfig.location.city} Area - ${new Date().toLocaleDateString()}

---

## Executive Summary

**Opportunities found:** ${summary.total}
- High priority: ${summary.high}
- Medium priority: ${summary.medium}
- Low priority: ${summary.low}

**Estimated total budget for high-priority:** ${summary.totalBudgetNeeded.toLocaleString()} SEK

---

## High Priority Opportunities (Take Action)

${highPriority.map((opp, i) => `
### ${i + 1}. ${opp.json.name}

**Fit Score:** ${opp.json.fitScore}/10 - ${opp.json.priority.toUpperCase()}
**Date:** ${opp.json.eventDate ? new Date(opp.json.eventDate).toLocaleDateString() : 'TBD'}
**Estimated Attendees:** ${opp.json.estimatedAttendees || 'Unknown'}

**Why This Opportunity:**
${opp.json.fitReason}

**Audience:**
${opp.json.audienceDescription}

**Expected ROI:**
- Brand Awareness: ${opp.json.expectedROI.brandAwareness}
- Lead Generation: ${opp.json.expectedROI.leadGeneration}
- Networking: ${opp.json.expectedROI.networking}

**Recommended Sponsorship:**
- Tier: ${opp.json.sponsorshipTier}
- Budget: ${opp.json.suggestedBudget.toLocaleString()} SEK
- Justification: ${opp.json.budgetJustification}

**Visibility Opportunities:**
${opp.json.visibilityOpportunities.split('; ').map(v => `- ${v}`).join('\n')}

**Potential Risks:**
${opp.json.risks.split('; ').map(r => `- ${r}`).join('\n')}

**Outreach Strategy:**
${opp.json.outreachStrategy}

**Pitch Angle:**
"${opp.json.pitchAngle}"

**Contact Information:**
${opp.json.extractedEmails.length > 0 ? `- Email: ${opp.json.extractedEmails[0]}` : ''}
${opp.json.extractedPhones.length > 0 ? `- Phone: ${opp.json.extractedPhones[0]}` : ''}
- Website: ${opp.json.url}

**Next Step:** ${opp.json.recommendedAction}

---
`).join('\n')}

## Medium Priority Opportunities (Consider)

${mediumPriority.slice(0, 5).map((opp, i) => `
${i + 1}. **${opp.json.name}**
   - Fit: ${opp.json.fitScore}/10
   - Budget: ${opp.json.suggestedBudget?.toLocaleString() || 'TBD'} SEK
   - Date: ${opp.json.eventDate ? new Date(opp.json.eventDate).toLocaleDateString() : 'TBD'}
   - Why: ${opp.json.fitReason}
   - Link: ${opp.json.url}
`).join('\n')}

${mediumPriority.length > 5 ? `\n*(+ ${mediumPriority.length - 5} more medium priority opportunities in database)*` : ''}

---

## Recommendations

1. **Prioritize high-fit events** in the next 30-60 days to maximize preparation time
2. **Total estimated investment** for all high-priority: ${summary.totalBudgetNeeded.toLocaleString()} SEK
3. **Focus areas:** Look for speaking opportunities at high-fit events (better ROI than pure sponsorship)
4. **Quick wins:** Smaller events with highly aligned audiences often outperform large generic events

---

*This report generated automatically from web research and AI analysis*
`;

return {
  report,
  reportTitle: `Sponsorship Research - ${searchConfig.location.city} - ${new Date().toLocaleDateString()}`,
  summary
};
```

### 13. Save Report to Google Docs (Google Docs node)

```
Operation: Create Document
Document Name: {{ $json.reportTitle }}
Folder ID: {{ $env.SPONSORSHIP_RESEARCH_FOLDER_ID }}
Content: {{ $json.report }}
```

### 14. Write to Opportunities Database (Google Sheets node)

```
Operation: Append Rows
Sheet ID: {{ $env.SPONSORSHIP_DATABASE_SHEET_ID }}
Sheet: Opportunities

Rows: {{ $('Extract and Merge Evaluation').all() }}

Columns:
  Opportunity Name: {{ $json.name }}
  Type: {{ $json.type }}
  Source: {{ $json.source }}
  Event Date: {{ $json.eventDate }}
  Organizer: {{ $json.organizer }}
  URL: {{ $json.url }}
  Fit Score: {{ $json.fitScore }}
  Priority: {{ $json.priority }}
  Suggested Budget: {{ $json.suggestedBudget }}
  Sponsorship Tier: {{ $json.sponsorshipTier }}
  Expected ROI Brand: {{ $json.expectedROI.brandAwareness }}
  Expected ROI Leads: {{ $json.expectedROI.leadGeneration }}
  Pitch Angle: {{ $json.pitchAngle }}
  Outreach Strategy: {{ $json.outreachStrategy }}
  Contact Email: {{ $json.extractedEmails[0] }}
  Status: evaluated
  Evaluated At: {{ $json.evaluatedAt }}
```

### 15. Send Weekly Digest (Notification Dispatch subflow)

```json
{
  "severity": "info",
  "message": "New sponsorship opportunities researched",
  "context": {
    "Location": "={{ $('Define Search Parameters').item.json.searchConfig.location.city }}",
    "Opportunities found": "={{ $('Filter by Priority').item.json.summary.total }}",
    "High priority": "={{ $('Filter by Priority').item.json.summary.high }}",
    "Estimated budget needed": "={{ $('Filter by Priority').item.json.summary.totalBudgetNeeded.toLocaleString() }} SEK",
    "Research report": "={{ $('Save Report to Google Docs').item.json.documentUrl }}"
  },
  "workflowName": "sponsorship-local-research-helper",
  "channels": ["email", "slack"]
}
```

### 16. Log Completion (Audit Log subflow)

```json
{
  "workflowName": "sponsorship-local-research-helper",
  "eventType": "success",
  "payloadSummary": "Researched {{ $('Filter by Priority').item.json.summary.total }} sponsorship opportunities",
  "details": {
    "highPriority": "={{ $('Filter by Priority').item.json.summary.high }}",
    "totalBudget": "={{ $('Filter by Priority').item.json.summary.totalBudgetNeeded }}"
  }
}
```

## Subflows Used

- **ai-call-wrapper:** For opportunity evaluation (step 9)
- **notification-dispatch:** For weekly digest (step 15)
- **audit-log:** For workflow logging (step 16)
- **webfetch:** For organizer research (step 7)

## Customization Tips

### Add specific industry filters

Focus on your niche:

```javascript
const industryKeywords = [
  'automation',
  'AI',
  'SaaS',
  'digital transformation'
];

// Filter opportunities by keywords in description
const relevant = opportunities.filter(opp =>
  industryKeywords.some(keyword =>
    opp.description.toLowerCase().includes(keyword.toLowerCase())
  )
);
```

### Track competitor sponsorships

Monitor where competitors are sponsoring:

```javascript
const competitors = ['Competitor A', 'Competitor B'];

// Search for competitor mentions at events
const competitorPresence = await searchGoogle(
  `"${competitor}" sponsor ${searchConfig.location.city} event`
);
```

### ROI tracking

After sponsoring, track actual results:

```javascript
// Add to database
opportunity.actualBudget = 15000;
opportunity.leadsGenerated = 8;
opportunity.meetingsBooked = 3;
opportunity.dealsWon = 1;
opportunity.costPerLead = opportunity.actualBudget / opportunity.leadsGenerated;
```

### Speaker opportunity prioritization

Favor speaking over sponsoring:

```javascript
// Check if speaking opportunities available
if (opportunity.description.includes('call for speakers') ||
    opportunity.description.includes('speaker proposals')) {
  opportunity.speakerOpportunity = true;
  opportunity.fitScore += 2;  // Boost fit score
}
```

### Integration with calendar

Auto-add high-priority event dates:

```javascript
if (opportunity.priority === 'high' && opportunity.eventDate) {
  await createCalendarEvent({
    title: `Consider: ${opportunity.name}`,
    date: opportunity.eventDate,
    description: `Sponsorship opportunity - Budget: ${opportunity.suggestedBudget} SEK`
  });
}
```

### Multi-region research

Expand beyond one city:

```javascript
const regions = ['Borås', 'Göteborg', 'Stockholm', 'Malmö'];

regions.forEach(async city => {
  const opportunities = await researchSponsorships(city);
  // Process each region
});
```

## Environment Variables Needed

```bash
# Search configuration
TARGET_CITY=Borås
TARGET_REGION=Västra Götaland
SPONSORSHIP_MIN_BUDGET=5000
SPONSORSHIP_MAX_BUDGET=50000

# Your business info
YOUR_BUSINESS_NAME=Your Company Name

# APIs
EVENTBRITE_TOKEN=your-eventbrite-token
MEETUP_API_KEY=your-meetup-key
GOOGLE_SEARCH_API_KEY=your-google-api-key
GOOGLE_SEARCH_ENGINE_ID=your-custom-search-engine-id

# AI
AI_MODEL=gpt-4-turbo
AI_TEMPERATURE=0.3

# Storage
SPONSORSHIP_RESEARCH_FOLDER_ID=google-drive-folder-id
SPONSORSHIP_DATABASE_SHEET_ID=your-sheet-id

# Notifications
SLACK_WEBHOOK_URL=your-slack-webhook
NOTIFICATION_EMAIL=your@email.com

# Audit
AUDIT_LOG_SHEET_ID=your-audit-sheet-id
```

## Performance Notes

- **Processing time:** 5-10 minutes per research run (depends on results found)
- **API costs:**
  - Google Custom Search: $5 per 1000 queries (100 free/day)
  - AI evaluation: ~$0.10-0.20 per opportunity (GPT-4)
- **Recommended frequency:** Weekly (Monday mornings)
- **Results volume:** Typically 10-30 opportunities per region per week

**Optimization:**
- Cache previously researched events (deduplicate)
- Skip low-quality results early (before AI evaluation)
- Use GPT-3.5 for initial filtering, GPT-4 for final evaluation

## Common Issues & Solutions

**Issue: Too many irrelevant results**
- Solution: Tighten search keywords, add industry-specific terms, improve filtering

**Issue: Missing contact information**
- Solution: Add manual research step for high-priority opportunities, use LinkedIn search

**Issue: Budget estimates vary widely**
- Solution: Collect actual sponsorship prices over time, build pricing database

**Issue: Events already past application deadline**
- Solution: Increase search frequency, add deadline tracking, notify sooner

**Issue: Same events show up repeatedly**
- Solution: Improve deduplication, track processed opportunities, skip if already evaluated

**Issue: Difficulty assessing organizer credibility**
- Solution: Manual verification for high-budget opportunities, check social proof (attendee testimonials)

This workflow transforms sponsorship research from scattered manual work into a systematic, data-driven process that identifies the best opportunities for your local market presence.
