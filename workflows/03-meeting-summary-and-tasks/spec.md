# Workflow: Meeting Summary and Tasks

## Summary

Automatically processes meeting recordings or transcripts to generate clean summaries, extract action items, and create tasks in your project management system. Works with Zoom, Google Meet, or manually uploaded audio/transcripts.

**Time saved:** 20-30 minutes per meeting → 2 minutes automated
**Best for:** Teams with frequent meetings, consultants, project managers

## Use Cases

1. **Client meetings**
   - Record Zoom call with client
   - Auto-generate summary with decisions made
   - Extract action items and assign to team

2. **Internal standups/planning**
   - Daily standup recordings
   - Extract blockers and next steps
   - Update project board automatically

3. **Discovery/sales calls**
   - Capture prospect requirements
   - Identify follow-up needed
   - Log notes to CRM with AI-extracted insights

## Inputs & Dependencies

**Required:**
- **Meeting source:** One of:
  - Zoom (with recording enabled)
  - Google Meet (with transcript enabled)
  - Manual upload (audio file or transcript)
- **AI API access:** OpenAI (Whisper for transcription + GPT for summarization) or Anthropic
- **Task destination:** Asana, ClickUp, Todoist, or Google Sheets
- **n8n credentials:**
  - Zoom/Google Meet OAuth2
  - OpenAI or Anthropic API key
  - Project management tool credentials

**Meeting data format:**
- Audio file (MP3, M4A, WAV) OR
- Transcript (TXT, VTT, SRT)
- Meeting metadata (date, attendees, topic)

## High-Level Flow

```
1. Trigger (new Zoom recording / manual upload / webhook)
   ↓
2. Fetch meeting data:
   2a. Download recording OR
   2b. Fetch transcript OR
   2c. Receive uploaded file
   ↓
3. If audio file: Transcribe with Whisper AI
   ↓
4. Clean and format transcript
   ↓
5. Call AI to extract:
   5a. Meeting summary
   5b. Key decisions
   5c. Action items (who, what, when)
   5d. Follow-up topics
   ↓
6. For each action item:
   6a. Create task in project management tool
   6b. Assign to person mentioned
   6c. Set due date if mentioned
   ↓
7. Compile full meeting note
   ↓
8. Save to:
   8a. Google Doc (shareable link)
   8b. Notion/Confluence page
   8c. CRM note (if sales call)
   ↓
9. Send summary notification to attendees
   ↓
10. Log completion
```

## Node-by-Node Implementation

### 1. Trigger: New Zoom Recording (Zoom Trigger node)

```
Event: Recording Completed
Options:
  - Download recording: Yes
  - Include transcript: Yes (if available)
```

**Alternative: Webhook Trigger (for manual uploads):**

```
HTTP Method: POST
Path: /webhook/meeting-upload
Authentication: Header Auth
Expected payload:
{
  "meetingTitle": "string",
  "meetingDate": "ISO date",
  "attendees": ["email1", "email2"],
  "audioUrl": "url" OR "transcript": "text"
}
```

### 2. Extract Meeting Metadata (Function node)

```javascript
const input = $input.item.json;

// Zoom webhook format
if (input.event === 'recording.completed') {
  return {
    meetingId: input.payload.object.id,
    topic: input.payload.object.topic,
    startTime: input.payload.object.start_time,
    duration: input.payload.object.duration,
    recordingUrl: input.payload.object.recording_files[0]?.download_url,
    transcript: input.payload.object.transcript,
    attendees: input.payload.object.participant_email?.split(',') || [],
    source: 'zoom'
  };
}

// Manual upload format
return {
  meetingId: Date.now().toString(),
  topic: input.meetingTitle,
  startTime: input.meetingDate,
  attendees: input.attendees || [],
  audioUrl: input.audioUrl,
  transcript: input.transcript,
  source: 'manual'
};
```

### 3. Check if Transcription Needed (IF node)

```
Condition: {{ $json.transcript === undefined || $json.transcript === '' }}
True: Download and transcribe audio
False: Skip to transcript processing
```

### 4. Download Recording (HTTP Request node)

```
Method: GET
URL: {{ $json.recordingUrl || $json.audioUrl }}
Response Format: File
Options:
  - Download: Yes
  - Destination: /tmp/meeting-{{ $json.meetingId }}.mp4
```

### 5. Extract Audio from Video (Execute Command node)

If recording is video (MP4), extract audio:

```bash
ffmpeg -i {{ $json.downloadPath }} -vn -acodec libmp3lame -q:a 2 {{ $json.downloadPath.replace('.mp4', '.mp3') }}
```

**Note:** This requires ffmpeg installed on your n8n instance. Skip if audio-only or use Zoom's audio file.

### 6. Transcribe with Whisper (OpenAI node)

```
Operation: Transcribe
Resource: Audio
File: {{ $json.audioFilePath }}
Model: whisper-1
Language: en (or auto-detect)
Options:
  - Response format: text
  - Temperature: 0
```

**Output:** Full transcript as text

### 7. Clean Transcript (Function node)

```javascript
let transcript = $input.item.json.transcript || $input.item.json.outputText;

// Remove excessive line breaks
transcript = transcript.replace(/\n{3,}/g, '\n\n');

// Remove timestamp artifacts if present
transcript = transcript.replace(/\[\d{2}:\d{2}:\d{2}\]/g, '');

// Remove filler words (optional)
transcript = transcript.replace(/\b(um|uh|like|you know)\b/gi, '');

// Split into paragraphs (every ~500 chars)
const paragraphs = [];
let current = '';
const words = transcript.split(' ');

words.forEach(word => {
  current += word + ' ';
  if (current.length > 500 && word.endsWith('.')) {
    paragraphs.push(current.trim());
    current = '';
  }
});
if (current) paragraphs.push(current.trim());

return {
  cleanTranscript: paragraphs.join('\n\n'),
  wordCount: words.length,
  originalTranscript: transcript
};
```

### 8. Build AI Extraction Prompt (Function node)

```javascript
const transcript = $input.item.json.cleanTranscript;
const metadata = $('Extract Meeting Metadata').item.json;

const prompt = `Analyze this meeting transcript and extract structured information.

**Meeting Details:**
Topic: ${metadata.topic}
Date: ${metadata.startTime}
Attendees: ${metadata.attendees.join(', ')}

**Transcript:**
${transcript}

Please extract and return JSON with:

{
  "summary": "2-3 sentence overview of the meeting",
  "keyPoints": ["bullet point 1", "bullet point 2", ...],
  "decisions": ["decision 1", "decision 2", ...],
  "actionItems": [
    {
      "task": "description of what needs to be done",
      "assignee": "name or email if mentioned, otherwise 'Unassigned'",
      "dueDate": "mentioned date or 'Not specified'",
      "priority": "high/medium/low based on context"
    }
  ],
  "followUpTopics": ["topic 1", "topic 2", ...],
  "nextMeeting": "details if a follow-up meeting was scheduled, otherwise null"
}

Focus on concrete action items. If someone says "I'll do X" or "We need to Y", that's an action item.`;

return {
  prompt,
  transcript,
  metadata
};
```

### 9. Call AI for Extraction (AI Call Wrapper subflow)

```json
{
  "prompt": "={{ $json.prompt }}",
  "mode": "json",
  "systemInstructions": "You are an expert meeting analyst. Extract action items accurately and comprehensively. Be specific about assignments and deadlines. If information is unclear, note it.",
  "temperature": 0.2,
  "model": "gpt-4-turbo"
}
```

### 10. Check AI Success (IF node)

```
Condition: {{ $json.success === true }}
True: Process extracted data
False: Handle error (manual review)
```

### 11. Parse AI Output (Function node)

```javascript
const extracted = $input.item.json.outputJson;
const metadata = $('Build AI Extraction Prompt').item.json.metadata;
const transcript = $('Build AI Extraction Prompt').item.json.transcript;

return {
  meetingId: metadata.meetingId,
  topic: metadata.topic,
  date: metadata.startTime,
  attendees: metadata.attendees,
  summary: extracted.summary,
  keyPoints: extracted.keyPoints,
  decisions: extracted.decisions,
  actionItems: extracted.actionItems,
  followUpTopics: extracted.followUpTopics,
  nextMeeting: extracted.nextMeeting,
  fullTranscript: transcript
};
```

### 12. Split Action Items (Split In Batches node)

```
Batch Size: 1
Options: Keep input data
Input: {{ $json.actionItems }}
```

### 13. Map Action Item to Task (Function node)

```javascript
const actionItem = $input.item.json;
const meeting = $('Parse AI Output').item.json;

// Try to match assignee to email
const assigneeEmail = meeting.attendees.find(email =>
  email.toLowerCase().includes(actionItem.assignee.toLowerCase())
) || null;

// Parse due date
let dueDate = null;
if (actionItem.dueDate !== 'Not specified') {
  // Try to parse relative dates
  if (actionItem.dueDate.includes('tomorrow')) {
    dueDate = new Date();
    dueDate.setDate(dueDate.getDate() + 1);
  } else if (actionItem.dueDate.includes('next week')) {
    dueDate = new Date();
    dueDate.setDate(dueDate.getDate() + 7);
  } else {
    dueDate = new Date(actionItem.dueDate);
  }
}

return {
  taskName: actionItem.task,
  description: `From meeting: ${meeting.topic}\nDate: ${meeting.date}\n\nFull context: ${meeting.summary}`,
  assignee: assigneeEmail,
  assigneeName: actionItem.assignee,
  dueDate: dueDate?.toISOString(),
  priority: actionItem.priority,
  tags: ['meeting-action', meeting.topic.toLowerCase().replace(/\s+/g, '-')],
  meetingId: meeting.meetingId
};
```

### 14. Create Task in Project Tool (ClickUp/Asana/Todoist node)

**Example for ClickUp:**

```
Operation: Create Task
List ID: {{ $env.CLICKUP_TASK_LIST_ID }}
Name: {{ $json.taskName }}
Description: {{ $json.description }}
Assignees: {{ $json.assignee }}
Due Date: {{ $json.dueDate }}
Priority: {{ $json.priority === 'high' ? 1 : $json.priority === 'medium' ? 2 : 3 }}
Tags: {{ $json.tags.join(',') }}
```

**Example for Asana:**

```
Operation: Create Task
Workspace: {{ $env.ASANA_WORKSPACE_ID }}
Project: {{ $env.ASANA_PROJECT_ID }}
Name: {{ $json.taskName }}
Notes: {{ $json.description }}
Assignee: {{ $json.assignee }}
Due On: {{ $json.dueDate }}
```

### 15. Aggregate Task Results (Aggregate node)

```
Aggregate All Items
```

### 16. Build Meeting Notes Document (Function node)

```javascript
const meeting = $('Parse AI Output').item.json;
const tasks = $input.all();

const createdTasks = tasks.filter(t => t.json.success !== false);

const notesMarkdown = `# ${meeting.topic}

**Date:** ${new Date(meeting.date).toLocaleString()}
**Attendees:** ${meeting.attendees.join(', ')}

---

## Summary

${meeting.summary}

## Key Points

${meeting.keyPoints.map(p => `- ${p}`).join('\n')}

## Decisions Made

${meeting.decisions.length > 0
  ? meeting.decisions.map(d => `- ${d}`).join('\n')
  : '_No explicit decisions recorded_'}

## Action Items

${meeting.actionItems.map((item, i) => `
${i + 1}. **${item.task}**
   - Assigned to: ${item.assignee}
   - Due: ${item.dueDate}
   - Priority: ${item.priority}
   ${createdTasks[i] ? `- Task created: ${createdTasks[i].json.taskUrl || 'Success'}` : ''}
`).join('\n')}

## Follow-up Topics

${meeting.followUpTopics.length > 0
  ? meeting.followUpTopics.map(t => `- ${t}`).join('\n')
  : '_None identified_'}

${meeting.nextMeeting ? `\n## Next Meeting\n\n${meeting.nextMeeting}` : ''}

---

## Full Transcript

${meeting.fullTranscript}

---

_Generated automatically on ${new Date().toLocaleString()}_
`;

return {
  markdown: notesMarkdown,
  documentTitle: `Meeting Notes: ${meeting.topic} - ${new Date(meeting.date).toLocaleDateString()}`,
  meeting
};
```

### 17. Save to Google Docs (Google Docs node)

```
Operation: Create Document
Document Name: {{ $json.documentTitle }}
Folder ID: {{ $env.MEETING_NOTES_FOLDER_ID }}
Content: {{ $json.markdown }}
```

**Alternative: Save to Notion:**

```
Operation: Create Page
Parent: {{ $env.NOTION_MEETING_DATABASE_ID }}
Title: {{ $json.documentTitle }}
Content: {{ $json.markdown }}
```

### 18. Optional: Save to CRM (if sales/client meeting)

```javascript
// Check if meeting involves client/prospect
const isClientMeeting = meeting.topic.toLowerCase().includes('client') ||
                        meeting.topic.toLowerCase().includes('discovery') ||
                        meeting.topic.toLowerCase().includes('demo');

if (!isClientMeeting) {
  return null;  // Skip CRM
}

return {
  saveTocrm: true,
  crmNote: meeting.summary,
  contactEmail: meeting.attendees.find(e => !e.includes('@yourdomain.com'))
};
```

**HubSpot note:**

```
Operation: Create
Resource: Engagement (Note)
Contact Email: {{ $json.contactEmail }}
Note Body: {{ $('Build Meeting Notes Document').item.json.markdown }}
```

### 19. Send Summary to Attendees (Email node)

```
To: {{ $('Parse AI Output').item.json.attendees.join(',') }}
Subject: Meeting Summary: {{ $('Parse AI Output').item.json.topic }}
Email Type: HTML

Body:
Hi team,

Here's the summary from our meeting on {{ $('Parse AI Output').item.json.date }}:

**Summary:** {{ $('Parse AI Output').item.json.summary }}

**Action Items:** {{ $('Parse AI Output').item.json.actionItems.length }} tasks created

Full notes: {{ $('Save to Google Docs').item.json.documentUrl }}

Best,
Automated Meeting Assistant
```

### 20. Send Notification (Notification Dispatch subflow)

```json
{
  "severity": "info",
  "message": "Meeting processed and tasks created",
  "context": {
    "Meeting": "={{ $('Parse AI Output').item.json.topic }}",
    "Tasks created": "={{ $('Aggregate Task Results').item.json.length }}",
    "Notes": "={{ $('Save to Google Docs').item.json.documentUrl }}"
  },
  "workflowName": "meeting-summary-and-tasks",
  "channels": ["slack"]
}
```

### 21. Log Completion (Audit Log subflow)

```json
{
  "workflowName": "meeting-summary-and-tasks",
  "eventType": "success",
  "payloadSummary": "Processed meeting: {{ $('Parse AI Output').item.json.topic }}",
  "details": {
    "tasksCreated": "={{ $('Aggregate Task Results').item.json.length }}",
    "documentUrl": "={{ $('Save to Google Docs').item.json.documentUrl }}"
  }
}
```

## Subflows Used

- **ai-call-wrapper:** For transcript analysis and extraction (step 9)
- **notification-dispatch:** For completion notification (step 20)
- **audit-log:** For workflow logging (step 21)

Optional:
- **safe-http-call:** For downloading recordings (step 4)

## Customization Tips

### Add speaker identification

If using Zoom with speaker-tagged transcripts:

```javascript
// Parse transcript with speaker labels
const transcript = input.transcript_vtt;
const lines = transcript.split('\n');
const speakers = {};

lines.forEach(line => {
  const match = line.match(/^(\w+): (.+)$/);
  if (match) {
    const [, speaker, text] = match;
    if (!speakers[speaker]) speakers[speaker] = [];
    speakers[speaker].push(text);
  }
});

// Include in AI prompt
prompt += `\n\nSpeakers:\n${Object.entries(speakers).map(([name, lines]) =>
  `${name}: ${lines.length} statements`
).join('\n')}`;
```

### Filter meetings by type

Only process certain meetings:

```javascript
// After trigger
const topic = $json.topic.toLowerCase();

// Skip standup meetings
if (topic.includes('standup') || topic.includes('daily sync')) {
  return null;
}

// Process only client meetings
if (!topic.includes('client') && !topic.includes('discovery')) {
  return null;
}

return $json;
```

### Auto-assign tasks based on keywords

```javascript
// In task mapping
const task = actionItem.task.toLowerCase();

// Auto-assign based on keywords
if (task.includes('design') || task.includes('mockup')) {
  assigneeEmail = 'designer@company.com';
} else if (task.includes('code') || task.includes('implement')) {
  assigneeEmail = 'developer@company.com';
} else if (task.includes('write') || task.includes('document')) {
  assigneeEmail = 'writer@company.com';
}
```

### Create calendar event for next meeting

```javascript
if (extracted.nextMeeting) {
  // Parse next meeting details
  const nextMeetingPrompt = `Extract date/time from: "${extracted.nextMeeting}"`;

  // Call AI to parse
  // Then create Google Calendar event
}
```

### Integrate with time tracking

If using Toggl/Harvest:

```javascript
// After transcription
const duration = metadata.duration;

// Create time entry
{
  "description": meeting.topic,
  "duration": duration,
  "projectId": inferredProjectId,
  "tags": ["meeting"]
}
```

### Add sentiment analysis

```javascript
// In AI prompt
prompt += `\n\nAlso analyze:
- Overall meeting sentiment (positive/neutral/negative)
- Any concerns or risks mentioned
- Energy level (engaged/routine/low)`;
```

## Environment Variables Needed

```bash
# Meeting sources
ZOOM_WEBHOOK_TOKEN=your-zoom-verification-token
ZOOM_API_KEY=your-zoom-key
ZOOM_API_SECRET=your-zoom-secret

# AI
OPENAI_API_KEY=your-openai-key  # For Whisper + GPT
AI_MODEL=gpt-4-turbo
AI_TEMPERATURE=0.2

# Task management
CLICKUP_API_KEY=your-clickup-key
CLICKUP_TASK_LIST_ID=list-id
ASANA_ACCESS_TOKEN=your-asana-token
ASANA_PROJECT_ID=project-id

# Document storage
MEETING_NOTES_FOLDER_ID=google-drive-folder-id
NOTION_MEETING_DATABASE_ID=notion-database-id

# CRM (optional)
HUBSPOT_API_KEY=your-hubspot-key

# Notifications
SLACK_WEBHOOK_URL=your-slack-webhook

# Audit
AUDIT_LOG_SHEET_ID=your-audit-sheet-id
```

## Performance Notes

- **Transcription time:** ~30-50% of audio length (30 min meeting = 10-15 min to transcribe)
- **AI extraction:** ~30-60 seconds per meeting
- **Total time:** 5-20 minutes depending on meeting length
- **Cost:** ~$0.20-0.50 per meeting (Whisper + GPT-4)

**Optimization tips:**
- Use GPT-3.5-turbo for simple meetings (saves 90% cost)
- Skip transcription if transcript already available
- Process overnight for batch recordings

## Common Issues & Solutions

**Issue: Transcription quality poor (lots of errors)**
- Solution: Ensure good audio quality, use external mic for Zoom
- Solution: Try Whisper's "large" model for better accuracy

**Issue: AI misses action items**
- Solution: Add examples to prompt: "Example action item: 'John will send the proposal by Friday'"

**Issue: Can't match assignee names to emails**
- Solution: Maintain a name-to-email mapping in environment variables or Google Sheet

**Issue: Tasks created in wrong project**
- Solution: Add logic to map meeting topic to project ID

**Issue: Duplicate tasks if workflow runs twice**
- Solution: Check if tasks already exist by searching for meetingId in task description

**Issue: Very long meetings (>2 hours) timeout**
- Solution: Split transcript into chunks and process separately, then combine results

This workflow turns hours of meeting follow-up into minutes, ensuring nothing falls through the cracks.
