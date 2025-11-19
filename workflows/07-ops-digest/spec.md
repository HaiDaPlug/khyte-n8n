# Workflow: Ops Digest

## Summary

Automatically compiles a daily operational digest by monitoring multiple systems: server health, error logs, deployment status, API usage, support tickets, and team activity. Generates a concise summary with AI-powered insights and alerts, delivered each morning.

**Time saved:** 30-45 minutes daily → 2 minutes reviewing digest
**Best for:** Technical leads, operations managers, startup founders

## Use Cases

1. **DevOps morning briefing**
   - Server uptime and performance metrics
   - Recent deployments and errors
   - API usage and rate limits
   - Incident summaries

2. **Customer success digest**
   - New support tickets and status
   - Customer health scores
   - Feature requests and bug reports
   - SLA compliance

3. **Product operations**
   - User activity and engagement metrics
   - Feature usage trends
   - Performance bottlenecks
   - Conversion funnel health

## Inputs & Dependencies

**Required:**
- **Monitoring sources:** At least 2 of:
  - Server monitoring (UptimeRobot, Pingdom, DataDog)
  - Error tracking (Sentry, Rollbar, LogRocket)
  - Support system (Zendesk, Intercom, Help Scout)
  - Analytics (Google Analytics, Mixpanel, Plausible)
  - CI/CD (GitHub Actions, GitLab CI, Vercel)
  - API monitoring (custom endpoint or APM)
- **AI API access:** OpenAI or Anthropic (for insights generation)
- **n8n credentials:**
  - API keys for each monitoring source
  - OpenAI/Anthropic API key
  - Email/Slack credentials

**Time range:**
- Last 24 hours (default)
- Configurable window

## High-Level Flow

```
1. Trigger (scheduled daily: 8am local time)
   ↓
2. Calculate time window (last 24 hours)
   ↓
3. Parallel data collection:
   3a. Fetch server/infrastructure status
   3b. Fetch error logs and incidents
   3c. Fetch support tickets
   3d. Fetch deployment activity
   3e. Fetch API/usage metrics
   3f. Fetch user analytics
   ↓
4. Aggregate all data sources
   ↓
5. Detect anomalies and issues:
   5a. Error rate spikes
   5b. Performance degradation
   5c. Unusual traffic patterns
   5d. SLA breaches
   ↓
6. For each section:
   6a. Calculate key metrics
   6b. Compare to baseline/previous day
   6c. Generate AI insights
   ↓
7. Compile digest:
   7a. Executive summary
   7b. Critical issues (if any)
   7c. Section-by-section details
   7d. Recommended actions
   ↓
8. Send digest via email and Slack
   ↓
9. If critical issues: Send urgent alert
   ↓
10. Log completion
```

## Node-by-Node Implementation

### 1. Trigger: Schedule Daily (Schedule Trigger node)

```
Trigger: Cron - every day at 8:00 AM
Expression: 0 8 * * *
Timezone: Europe/Stockholm (or your timezone)
```

### 2. Calculate Time Window (Function node)

```javascript
const now = new Date();

// Yesterday 8am to today 8am (24 hour window)
const windowEnd = now;
const windowStart = new Date(now);
windowStart.setDate(now.getDate() - 1);

// For comparison (day before yesterday)
const previousWindowStart = new Date(windowStart);
previousWindowStart.setDate(windowStart.getDate() - 1);
const previousWindowEnd = new Date(windowStart);

return {
  current: {
    start: windowStart.toISOString(),
    end: windowEnd.toISOString(),
    label: windowStart.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
  },
  previous: {
    start: previousWindowStart.toISOString(),
    end: previousWindowEnd.toISOString()
  },
  timestamp: now.toISOString()
};
```

### 3a. Fetch Server/Infrastructure Status (HTTP Request - UptimeRobot/DataDog)

**For UptimeRobot:**

```
Method: POST
URL: https://api.uptimerobot.com/v2/getMonitors
Headers:
  Content-Type: application/x-www-form-urlencoded
Body:
  api_key={{ $env.UPTIMEROBOT_API_KEY }}
  format=json
  logs=1
  log_start_date={{ Math.floor(new Date($('Calculate Time Window').item.json.current.start).getTime() / 1000) }}
  log_end_date={{ Math.floor(new Date($('Calculate Time Window').item.json.current.end).getTime() / 1000) }}
```

Parse response:

```javascript
const monitors = $input.item.json.monitors;

const totalMonitors = monitors.length;
const upMonitors = monitors.filter(m => m.status === 2).length;
const downMonitors = monitors.filter(m => m.status === 8 || m.status === 9).length;

const incidents = monitors.flatMap(m =>
  (m.logs || [])
    .filter(log => log.type === 1)  // Type 1 = down
    .map(log => ({
      monitor: m.friendly_name,
      type: 'downtime',
      startTime: new Date(log.datetime * 1000).toISOString(),
      duration: log.duration
    }))
);

const avgUptime = monitors.reduce((sum, m) => sum + parseFloat(m.all_time_uptime_ratio), 0) / totalMonitors;

return {
  section: 'infrastructure',
  totalMonitors,
  upMonitors,
  downMonitors,
  uptime: avgUptime.toFixed(2) + '%',
  incidents,
  incidentCount: incidents.length
};
```

**For DataDog:**

```
Method: GET
URL: https://api.datadoghq.com/api/v1/monitor
Headers:
  DD-API-KEY: {{ $env.DATADOG_API_KEY }}
  DD-APPLICATION-KEY: {{ $env.DATADOG_APP_KEY }}
Parameters:
  group_states: alert,warn,no data
```

### 3b. Fetch Error Logs (Sentry API)

```
Method: GET
URL: https://sentry.io/api/0/projects/{{ $env.SENTRY_ORG }}/{{ $env.SENTRY_PROJECT }}/issues/
Headers:
  Authorization: Bearer {{ $env.SENTRY_AUTH_TOKEN }}
Parameters:
  query: is:unresolved
  start: {{ $('Calculate Time Window').item.json.current.start }}
  end: {{ $('Calculate Time Window').item.json.current.end }}
  statsPeriod: 24h
```

Parse errors:

```javascript
const issues = $input.all();

const totalErrors = issues.reduce((sum, issue) =>
  sum + parseInt(issue.json.count || 0), 0
);

const criticalErrors = issues.filter(issue =>
  issue.json.level === 'error' || issue.json.level === 'fatal'
).length;

const topErrors = issues
  .sort((a, b) => parseInt(b.json.count) - parseInt(a.json.count))
  .slice(0, 5)
  .map(issue => ({
    title: issue.json.title,
    count: issue.json.count,
    url: issue.json.permalink,
    firstSeen: issue.json.firstSeen,
    level: issue.json.level
  }));

return {
  section: 'errors',
  totalErrors,
  uniqueIssues: issues.length,
  criticalErrors,
  topErrors
};
```

### 3c. Fetch Support Tickets (Zendesk/Intercom)

**For Zendesk:**

```
Method: GET
URL: https://{{ $env.ZENDESK_SUBDOMAIN }}.zendesk.com/api/v2/search.json
Headers:
  Authorization: Basic {{ Buffer.from($env.ZENDESK_EMAIL + '/token:' + $env.ZENDESK_API_TOKEN).toString('base64') }}
Parameters:
  query: type:ticket created>={{ $('Calculate Time Window').item.json.current.start }}
```

Parse tickets:

```javascript
const tickets = $input.item.json.results;

const newTickets = tickets.filter(t => t.status === 'new').length;
const openTickets = tickets.filter(t => t.status === 'open').length;
const solvedTickets = tickets.filter(t => t.status === 'solved').length;

const urgentTickets = tickets.filter(t => t.priority === 'urgent' || t.priority === 'high');

const avgResponseTime = tickets
  .filter(t => t.assignee_id && t.updated_at)
  .reduce((sum, t) => {
    const created = new Date(t.created_at);
    const updated = new Date(t.updated_at);
    return sum + (updated - created);
  }, 0) / tickets.length;

const avgResponseHours = (avgResponseTime / (1000 * 60 * 60)).toFixed(1);

return {
  section: 'support',
  totalTickets: tickets.length,
  newTickets,
  openTickets,
  solvedTickets,
  urgentTickets: urgentTickets.length,
  avgResponseTime: avgResponseHours + ' hours',
  topIssues: urgentTickets.slice(0, 3).map(t => ({
    id: t.id,
    subject: t.subject,
    priority: t.priority,
    status: t.status
  }))
};
```

### 3d. Fetch Deployment Activity (GitHub API)

```
Method: GET
URL: https://api.github.com/repos/{{ $env.GITHUB_REPO }}/deployments
Headers:
  Authorization: token {{ $env.GITHUB_TOKEN }}
  Accept: application/vnd.github.v3+json
Parameters:
  environment: production
  per_page: 20
```

Parse deployments:

```javascript
const deployments = $input.all();
const timeWindow = $('Calculate Time Window').item.json.current;

const recentDeployments = deployments.filter(d => {
  const deployedAt = new Date(d.json.created_at);
  return deployedAt >= new Date(timeWindow.start) && deployedAt <= new Date(timeWindow.end);
});

const successfulDeployments = recentDeployments.filter(d =>
  d.json.statuses?.[0]?.state === 'success'
).length;

const failedDeployments = recentDeployments.filter(d =>
  d.json.statuses?.[0]?.state === 'failure'
).length;

return {
  section: 'deployments',
  totalDeployments: recentDeployments.length,
  successful: successfulDeployments,
  failed: failedDeployments,
  deployments: recentDeployments.map(d => ({
    sha: d.json.sha.substring(0, 7),
    environment: d.json.environment,
    status: d.json.statuses?.[0]?.state || 'pending',
    createdAt: d.json.created_at
  }))
};
```

### 3e. Fetch API Usage Metrics (Custom endpoint or Stripe for usage-based)

**Custom API metrics endpoint:**

```
Method: GET
URL: {{ $env.API_METRICS_URL }}/metrics
Headers:
  Authorization: Bearer {{ $env.API_METRICS_TOKEN }}
Parameters:
  start: {{ $('Calculate Time Window').item.json.current.start }}
  end: {{ $('Calculate Time Window').item.json.current.end }}
```

**Or query from logs/database:**

```javascript
// If you track API calls in a database
const query = `
  SELECT
    COUNT(*) as total_requests,
    COUNT(DISTINCT user_id) as unique_users,
    AVG(response_time_ms) as avg_response_time,
    COUNT(CASE WHEN status_code >= 500 THEN 1 END) as errors_5xx,
    COUNT(CASE WHEN status_code >= 400 AND status_code < 500 THEN 1 END) as errors_4xx
  FROM api_logs
  WHERE timestamp >= $1 AND timestamp <= $2
`;

const result = await db.query(query, [timeWindow.start, timeWindow.end]);

return {
  section: 'api',
  totalRequests: result.rows[0].total_requests,
  uniqueUsers: result.rows[0].unique_users,
  avgResponseTime: result.rows[0].avg_response_time + 'ms',
  serverErrors: result.rows[0].errors_5xx,
  clientErrors: result.rows[0].errors_4xx,
  errorRate: ((result.rows[0].errors_5xx + result.rows[0].errors_4xx) / result.rows[0].total_requests * 100).toFixed(2) + '%'
};
```

### 3f. Fetch User Analytics (Google Analytics or Plausible)

**For Plausible (privacy-friendly):**

```
Method: GET
URL: https://plausible.io/api/v1/stats/aggregate
Headers:
  Authorization: Bearer {{ $env.PLAUSIBLE_API_KEY }}
Parameters:
  site_id: {{ $env.PLAUSIBLE_SITE_ID }}
  period: custom
  date: {{ $('Calculate Time Window').item.json.current.start }},{{ $('Calculate Time Window').item.json.current.end }}
  metrics: visitors,pageviews,bounce_rate,visit_duration
```

Parse analytics:

```javascript
const stats = $input.item.json.results;

return {
  section: 'analytics',
  visitors: stats.visitors.value,
  pageviews: stats.pageviews.value,
  bounceRate: stats.bounce_rate.value + '%',
  avgSessionDuration: Math.round(stats.visit_duration.value) + 's'
};
```

### 4. Aggregate All Sections (Merge node)

```
Mode: Merge By Position
Inputs: All data collection nodes (3a-3f)
```

### 5. Detect Anomalies (Function node)

```javascript
const sections = $input.all();
const infrastructure = sections.find(s => s.json.section === 'infrastructure')?.json;
const errors = sections.find(s => s.json.section === 'errors')?.json;
const support = sections.find(s => s.json.section === 'support')?.json;
const deployments = sections.find(s => s.json.section === 'deployments')?.json;
const api = sections.find(s => s.json.section === 'api')?.json;

const anomalies = [];
const criticalIssues = [];

// Infrastructure checks
if (infrastructure?.downMonitors > 0) {
  criticalIssues.push({
    severity: 'critical',
    area: 'Infrastructure',
    issue: `${infrastructure.downMonitors} monitor(s) down`,
    details: infrastructure.incidents
  });
}

if (infrastructure?.incidentCount > 3) {
  anomalies.push({
    area: 'Infrastructure',
    metric: 'Incidents',
    value: infrastructure.incidentCount,
    threshold: 3,
    message: 'Higher than normal incident count'
  });
}

// Error checks
const errorRateThreshold = 100;
if (errors?.totalErrors > errorRateThreshold) {
  anomalies.push({
    area: 'Errors',
    metric: 'Total errors',
    value: errors.totalErrors,
    threshold: errorRateThreshold,
    message: 'Error rate above threshold'
  });
}

if (errors?.criticalErrors > 0) {
  criticalIssues.push({
    severity: 'high',
    area: 'Errors',
    issue: `${errors.criticalErrors} critical errors`,
    details: errors.topErrors
  });
}

// Support checks
if (support?.urgentTickets > 5) {
  criticalIssues.push({
    severity: 'medium',
    area: 'Support',
    issue: `${support.urgentTickets} urgent tickets`,
    details: support.topIssues
  });
}

// Deployment checks
if (deployments?.failed > 0) {
  criticalIssues.push({
    severity: 'high',
    area: 'Deployments',
    issue: `${deployments.failed} failed deployment(s)`,
    details: deployments.deployments.filter(d => d.status === 'failure')
  });
}

// API checks
const apiErrorRateThreshold = 5;  // 5%
if (api && parseFloat(api.errorRate) > apiErrorRateThreshold) {
  anomalies.push({
    area: 'API',
    metric: 'Error rate',
    value: api.errorRate,
    threshold: apiErrorRateThreshold + '%',
    message: 'API error rate elevated'
  });
}

return {
  hasCriticalIssues: criticalIssues.length > 0,
  criticalIssues,
  anomalies,
  allSections: sections.map(s => s.json)
};
```

### 6. Generate AI Insights (Function node + AI Call Wrapper)

```javascript
const { allSections, anomalies, criticalIssues } = $input.item.json;

const infrastructure = allSections.find(s => s.section === 'infrastructure');
const errors = allSections.find(s => s.section === 'errors');
const support = allSections.find(s => s.section === 'support');
const deployments = allSections.find(s => s.section === 'deployments');
const api = allSections.find(s => s.section === 'api');
const analytics = allSections.find(s => s.section === 'analytics');

const prompt = `Analyze this operational data and provide insights for a daily ops digest.

**Infrastructure:**
- Uptime: ${infrastructure?.uptime || 'N/A'}
- Monitors: ${infrastructure?.upMonitors}/${infrastructure?.totalMonitors} up
- Incidents: ${infrastructure?.incidentCount || 0}

**Errors:**
- Total errors: ${errors?.totalErrors || 0}
- Unique issues: ${errors?.uniqueIssues || 0}
- Critical: ${errors?.criticalErrors || 0}
- Top errors: ${JSON.stringify(errors?.topErrors || [])}

**Support:**
- New tickets: ${support?.newTickets || 0}
- Open: ${support?.openTickets || 0}
- Solved: ${support?.solvedTickets || 0}
- Avg response time: ${support?.avgResponseTime || 'N/A'}

**Deployments:**
- Total: ${deployments?.totalDeployments || 0}
- Successful: ${deployments?.successful || 0}
- Failed: ${deployments?.failed || 0}

**API:**
- Requests: ${api?.totalRequests || 0}
- Error rate: ${api?.errorRate || 'N/A'}
- Avg response time: ${api?.avgResponseTime || 'N/A'}

**Analytics:**
- Visitors: ${analytics?.visitors || 0}
- Pageviews: ${analytics?.pageviews || 0}

**Anomalies detected:**
${anomalies.length > 0 ? JSON.stringify(anomalies, null, 2) : 'None'}

**Critical issues:**
${criticalIssues.length > 0 ? JSON.stringify(criticalIssues, null, 2) : 'None'}

Provide:
1. executiveSummary: 2-3 sentence overview (health status, key wins, concerns)
2. keyInsights: Array of 3-5 notable insights or trends
3. recommendedActions: Array of 2-3 specific actions to take (if issues found)
4. overallStatus: "healthy" | "concerning" | "critical"

Return JSON.`;

return {
  prompt,
  allSections,
  anomalies,
  criticalIssues
};
```

Call AI:

```json
{
  "prompt": "={{ $json.prompt }}",
  "mode": "json",
  "systemInstructions": "You are an operations analyst reviewing daily system health. Be concise but insightful. Focus on actionable information. If everything looks good, say so. If there are concerns, be specific about what to investigate.",
  "temperature": 0.3,
  "model": "gpt-4-turbo"
}
```

### 7. Build Digest Email/Message (Function node)

```javascript
const aiInsights = $input.item.json.outputJson;
const data = $('Detect Anomalies').item.json;
const timeWindow = $('Calculate Time Window').item.json.current;

const statusEmoji = {
  healthy: '✅',
  concerning: '⚠️',
  critical: '🚨'
};

const digest = `# Operations Digest - ${timeWindow.label}

${statusEmoji[aiInsights.overallStatus]} **Status: ${aiInsights.overallStatus.toUpperCase()}**

---

## Executive Summary

${aiInsights.executiveSummary}

---

## Key Metrics

**Infrastructure**
- Uptime: ${data.allSections.find(s => s.section === 'infrastructure')?.uptime || 'N/A'}
- Incidents: ${data.allSections.find(s => s.section === 'infrastructure')?.incidentCount || 0}

**Errors**
- Total: ${data.allSections.find(s => s.section === 'errors')?.totalErrors || 0}
- Critical: ${data.allSections.find(s => s.section === 'errors')?.criticalErrors || 0}

**Support**
- New tickets: ${data.allSections.find(s => s.section === 'support')?.newTickets || 0}
- Avg response: ${data.allSections.find(s => s.section === 'support')?.avgResponseTime || 'N/A'}

**Deployments**
- Successful: ${data.allSections.find(s => s.section === 'deployments')?.successful || 0}
- Failed: ${data.allSections.find(s => s.section === 'deployments')?.failed || 0}

**API**
- Requests: ${data.allSections.find(s => s.section === 'api')?.totalRequests || 0}
- Error rate: ${data.allSections.find(s => s.section === 'api')?.errorRate || 'N/A'}

---

## Insights

${aiInsights.keyInsights.map((insight, i) => `${i + 1}. ${insight}`).join('\n')}

${data.criticalIssues.length > 0 ? `\n---\n\n## 🚨 Critical Issues\n\n${data.criticalIssues.map(issue => `**${issue.area}:** ${issue.issue}`).join('\n')}` : ''}

${aiInsights.recommendedActions.length > 0 ? `\n---\n\n## Recommended Actions\n\n${aiInsights.recommendedActions.map((action, i) => `${i + 1}. ${action}`).join('\n')}` : ''}

---

*Generated automatically at ${new Date().toLocaleString()}*
`;

return {
  digest,
  subject: `Ops Digest ${timeWindow.label} - ${aiInsights.overallStatus.toUpperCase()}`,
  overallStatus: aiInsights.overallStatus,
  hasCriticalIssues: data.hasCriticalIssues
};
```

### 8. Send Digest Email (Email node)

```
To: {{ $env.OPS_DIGEST_EMAIL }}
Subject: {{ $json.subject }}
Email Type: Text (or HTML if formatted)
Body: {{ $json.digest }}
```

### 9. Send Digest to Slack (Slack node)

```
Channel: #ops-digest
Message: {{ $json.digest }}
```

**Or use blocks for better formatting:**

```json
{
  "channel": "#ops-digest",
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "{{ $json.subject }}"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "{{ $json.digest }}"
      }
    }
  ]
}
```

### 10. Send Critical Alert (IF + Notification Dispatch)

```
Condition: {{ $json.hasCriticalIssues === true }}
True: Send alert
False: Skip
```

```json
{
  "severity": "urgent",
  "message": "Critical issues detected in ops digest",
  "context": {
    "Status": "={{ $('Build Digest Email/Message').item.json.overallStatus }}",
    "Critical issues": "={{ $('Detect Anomalies').item.json.criticalIssues.length }}",
    "Details": "Check #ops-digest channel"
  },
  "workflowName": "ops-digest",
  "channels": ["slack", "sms"]
}
```

### 11. Log Completion (Audit Log subflow)

```json
{
  "workflowName": "ops-digest",
  "eventType": "success",
  "payloadSummary": "Digest generated - Status: {{ $('Build Digest Email/Message').item.json.overallStatus }}",
  "details": {
    "criticalIssues": "={{ $('Detect Anomalies').item.json.criticalIssues.length }}",
    "anomalies": "={{ $('Detect Anomalies').item.json.anomalies.length }}"
  }
}
```

## Subflows Used

- **ai-call-wrapper:** For insights generation (step 6)
- **notification-dispatch:** For critical alerts (step 10)
- **audit-log:** For workflow logging (step 11)

Optional:
- **safe-http-call:** For API requests (steps 3a-3f)

## Customization Tips

### Add custom metrics

Track business-specific KPIs:

```javascript
// Revenue metrics
const dailyRevenue = await getStripeRevenue(timeWindow);

// User engagement
const activeUsers = await getActiveUsers(timeWindow);

// Custom events
const conversions = await getConversions(timeWindow);
```

### Trend analysis

Compare to previous periods:

```javascript
const change = ((current - previous) / previous * 100).toFixed(1);
const trend = change > 0 ? '📈' : '📉';

insights.push(`Error rate ${trend} ${Math.abs(change)}% vs yesterday`);
```

### Conditional sections

Only include relevant sections:

```javascript
if (deployments.totalDeployments === 0) {
  // Skip deployments section
  return null;
}
```

### Integration with incident management

Auto-create incidents for critical issues:

```javascript
if (criticalIssues.length > 0) {
  // Create PagerDuty/Opsgenie incident
  await createIncident({
    title: `Ops Digest: ${criticalIssues.length} critical issues`,
    details: criticalIssues
  });
}
```

### Weekly rollup

On Mondays, include weekly summary:

```javascript
const today = new Date().getDay();
if (today === 1) {  // Monday
  // Fetch last 7 days data
  // Generate weekly trends
}
```

### Custom alert thresholds

Configure per-metric thresholds:

```javascript
const thresholds = {
  errorRate: process.env.ERROR_RATE_THRESHOLD || 100,
  responseTime: process.env.RESPONSE_TIME_THRESHOLD || 1000,
  uptimeMin: process.env.UPTIME_MIN || 99.9
};
```

## Environment Variables Needed

```bash
# Monitoring services
UPTIMEROBOT_API_KEY=your-uptimerobot-key
DATADOG_API_KEY=your-datadog-key
DATADOG_APP_KEY=your-datadog-app-key
SENTRY_ORG=your-org
SENTRY_PROJECT=your-project
SENTRY_AUTH_TOKEN=your-sentry-token

# Support
ZENDESK_SUBDOMAIN=your-subdomain
ZENDESK_EMAIL=your-email
ZENDESK_API_TOKEN=your-token

# Version control
GITHUB_TOKEN=your-github-token
GITHUB_REPO=owner/repo

# Analytics
PLAUSIBLE_API_KEY=your-plausible-key
PLAUSIBLE_SITE_ID=your-site-id

# Custom metrics
API_METRICS_URL=your-metrics-endpoint
API_METRICS_TOKEN=your-token

# AI
AI_MODEL=gpt-4-turbo
AI_TEMPERATURE=0.3

# Notifications
OPS_DIGEST_EMAIL=ops-team@company.com
SLACK_OPS_CHANNEL=#ops-digest
SMS_ALERT_NUMBER=your-number  # for critical alerts

# Alert thresholds
ERROR_RATE_THRESHOLD=100
RESPONSE_TIME_THRESHOLD=1000
UPTIME_MIN=99.9

# Audit
AUDIT_LOG_SHEET_ID=your-audit-sheet-id
```

## Performance Notes

- **Processing time:** 1-3 minutes (depends on number of sources)
- **API calls:** 5-10 external APIs per run
- **AI cost:** ~$0.10-0.20 per digest (GPT-4)
- **Recommended schedule:** Daily at consistent time (8am)

**Optimization:**
- Cache data that doesn't change often
- Run parallel API requests
- Use GPT-3.5 for simpler insights (cheaper)

## Common Issues & Solutions

**Issue: Metrics from certain sources fail**
- Solution: Add error handling, use cached/previous values, note "unavailable" in digest

**Issue: Too much data, digest is overwhelming**
- Solution: Summarize more aggressively, only show anomalies, add "full report" link

**Issue: False positive anomalies (weekend traffic dips, etc.)**
- Solution: Add day-of-week awareness, adjust thresholds by day

**Issue: Digest arrives too late (after standup)**
- Solution: Adjust schedule trigger timezone and time

**Issue: Critical alerts trigger too often**
- Solution: Implement alert de-duplication, require threshold sustained for N minutes

**Issue: AI insights too generic**
- Solution: Provide more context in prompt (your product, what you care about)

This workflow gives you a comprehensive operational overview every morning, helping you stay ahead of issues and make data-driven decisions.
