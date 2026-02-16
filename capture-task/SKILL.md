---
name: capture-task
description: "Capture any task or to-do, classify it by topic, and route it to the correct Notion database. Use this skill whenever the user says things like 'log a task', 'log a to-do', 'capture this', 'add a task', 'I need to remember to', 'remind me to', 'add to my list', 'track this', or describes something actionable they want to save for later. Also trigger on 'what tasks do I have', 'show my tasks', or 'check my intake' to review the master intake log."
---

# Universal Task and To-Do Capture

Captures any task or to-do, classifies it by topic using AI, and routes it to the correct Notion topic database. Every capture also creates an audit trail in the master intake database.

## Databases

### Master Intake
- **Master Task Intake** — data source ID: `c20d0a8a-48eb-4556-824a-8db6d41afabd`

### Topic Databases

| Category | Database | Data Source ID |
|----------|----------|----------------|
| Shopping / Errands | Shopping & Errands | `4601cd6c-0145-43bb-842f-5aa0ea21e3f2` |
| Technical / Dev | Technical Tasks | `91b88e19-dcde-43b2-a969-d713b3dfd28d` |
| Class / Study | Study To-Dos | `3c36c983-099b-48e5-9484-5b878bdd95bb` |
| Content / Writing | Content Pipeline | `8dea9c69-3698-4b10-9f04-d8fd629e1937` |
| Business / Sales | Business Tasks | `4953aef4-0aad-4021-a47a-f463f64587bb` |
| Personal | Personal To-Dos | `37eab999-4369-4842-bdf6-6dda3be3142e` |
| Workflow / Process | Workflow Tasks | `dece1175-e5bc-4562-995e-afb6edc4ff9a` |
| Social / Community | Social & Community | `541404e0-43a5-4722-8d4d-d298ab9d8ed9` |
| Needs Sorting | Needs Sorting | `f1a5fa34-3113-4212-b12c-1c1a5fa58955` |

## Logging a Task

Be conversational and low-friction. The user wants to capture a thought before it disappears — don't slow them down with unnecessary questions.

### Step 1: Capture

Receive the user's task. If it's clear, proceed immediately to classification. Do NOT ask follow-up questions unless the input is genuinely ambiguous.

Optionally collect if the user offers:
- **Priority** — High / Medium / Low (default: Medium)
- **Notes** — any additional context

### Step 2: Create Master Record

Immediately create a record in the master intake database with status "Pending":

```
Tool: notion-create-pages
parent: { "data_source_id": "c20d0a8a-48eb-4556-824a-8db6d41afabd" }
pages: [{
  properties: {
    "Task": "<raw input text>",
    "Source": "Claude Skill",
    "Status": "Pending",
    "Priority": "<priority or Medium>"
  }
}]
```

Save the returned page ID and URL for Step 5.

### Step 3: Classify

Evaluate the raw input against these categories. Choose the single best fit.

| Category | What belongs here | Examples |
|----------|------------------|----------|
| **Shopping / Errands** | Physical tasks, purchases, pickups, drop-offs | "Pick up dry cleaning," "Buy birthday gift for Mom," "Return Amazon package," "Get groceries" |
| **Technical / Dev** | Coding, debugging, infrastructure, tooling, dev environment, CI/CD | "Fix the auth bug in login flow," "Set up CI pipeline," "Refactor the API layer," "Update Node version" |
| **Class / Study** | Coursework, certification prep, learning tasks, study sessions, exam prep | "Review Chapter 7 on access controls," "Practice CompTIA labs," "Read the PKI section," "Study for quiz" |
| **Content / Writing** | Blog posts, articles, social media, creative writing, media production, newsletters | "Draft the Substack post on AI workflows," "Write LinkedIn post about MEDDPICC," "Outline podcast episode" |
| **Business / Sales** | Revenue, pipeline, CRM, discovery calls, proposals, customer-facing, operational | "Follow up with Acme Corp on the proposal," "Update CRM with call notes," "Prep for Thursday's discovery call" |
| **Personal** | Health, finance, home, life admin, relationships, non-work tasks | "Schedule dentist appointment," "Call Dad this weekend," "Renew passport," "Pay electric bill" |
| **Workflow / Process** | Building, documenting, improving workflows, systems, skills, automations | "Write SOP for onboarding workflow," "Add a new category to capture skill," "Review classification accuracy" |
| **Social / Community** | Events, meetups, community engagement, networking, social commitments | "RSVP to the AI meetup," "Follow up with Maria from the conference," "Plan game night," "Join the Discord event" |

**Classification rules:**
1. Choose the single best category. Never multi-classify.
2. If the task clearly fits one category, assign it immediately.
3. If the task is ambiguous between exactly two categories, ask ONE clarifying question. Example: "Should I file this under Technical/Dev or Workflow/Process?"
4. If the task doesn't fit any category, classify as "Needs Sorting."
5. Use context from the current conversation to inform classification — if the user was just discussing Security+ study material, a vague "add that to my list" likely means Class / Study.

### Step 4: Route to Topic Database

Create a page in the matched topic database. Each database has a slightly different schema — use the correct properties for the target.

**Shopping & Errands:**
```
parent: { "data_source_id": "4601cd6c-0145-43bb-842f-5aa0ea21e3f2" }
properties: {
  "Task": "<task>",
  "Priority": "<priority or Medium>",
  "Status": "To Do",
  "Location": "<if mentioned>",
  "Cost Estimate": <if mentioned, number>,
  "Notes": "<any extra context>"
}
```

**Technical Tasks:**
```
parent: { "data_source_id": "91b88e19-dcde-43b2-a969-d713b3dfd28d" }
properties: {
  "Task": "<task>",
  "Priority": "<priority or Medium>",
  "Status": "To Do",
  "Project": "<if identifiable from context>",
  "Repo": "<if identifiable from context>",
  "Notes": "<any extra context>"
}
```

**Study To-Dos:**
```
parent: { "data_source_id": "3c36c983-099b-48e5-9484-5b878bdd95bb" }
properties: {
  "Task": "<task>",
  "Priority": "<priority or Medium>",
  "Status": "To Do",
  "Class": "<one of: Hands-on AI, Kite The Planet, Professional, Security+, Spanish, Other>",
  "Topic": "<specific topic if mentioned>",
  "Notes": "<any extra context>"
}
```

**Content Pipeline:**
```
parent: { "data_source_id": "8dea9c69-3698-4b10-9f04-d8fd629e1937" }
properties: {
  "Task": "<task>",
  "Priority": "<priority or Medium>",
  "Status": "To Do",
  "Platform": "<one of: Substack, LinkedIn, YouTube, Other — if identifiable>",
  "Notes": "<any extra context>"
}
```

**Business Tasks:**
```
parent: { "data_source_id": "4953aef4-0aad-4021-a47a-f463f64587bb" }
properties: {
  "Task": "<task>",
  "Priority": "<priority or Medium>",
  "Status": "To Do",
  "Company": "<if mentioned>",
  "Contact": "<if mentioned>",
  "Notes": "<any extra context>"
}
```

**Personal To-Dos:**
```
parent: { "data_source_id": "37eab999-4369-4842-bdf6-6dda3be3142e" }
properties: {
  "Task": "<task>",
  "Priority": "<priority or Medium>",
  "Status": "To Do",
  "Area": "<one of: Health, Finance, Home, Life Admin, Other — best fit>",
  "Notes": "<any extra context>"
}
```

**Workflow Tasks:**
```
parent: { "data_source_id": "dece1175-e5bc-4562-995e-afb6edc4ff9a" }
properties: {
  "Task": "<task>",
  "Priority": "<priority or Medium>",
  "Status": "To Do",
  "Workflow": "<workflow name if identifiable>",
  "Type": "<one of: Build, Document, Improve — best fit>",
  "Notes": "<any extra context>"
}
```

**Social & Community:**
```
parent: { "data_source_id": "541404e0-43a5-4722-8d4d-d298ab9d8ed9" }
properties: {
  "Task": "<task>",
  "Priority": "<priority or Medium>",
  "Status": "To Do",
  "Event / Group": "<if mentioned>",
  "Location": "<if mentioned>",
  "Notes": "<any extra context>"
}
```

**Needs Sorting:**
```
parent: { "data_source_id": "f1a5fa34-3113-4212-b12c-1c1a5fa58955" }
properties: {
  "Task": "<task>",
  "Status": "Unsorted",
  "Suggested Category": "<best guess or Unknown>",
  "Reason": "<why classification was uncertain>",
  "Notes": "<any extra context>"
}
```

Save the returned page URL for Step 5.

### Step 5: Link Master Record

Update the master intake record created in Step 2:

```
Tool: notion-update-page
page_id: <master_record_page_id from Step 2>
command: "update_properties"
properties:
  "Status": "Routed"
  "Category": "<classified category>"
  "Topic Link": "<topic_page_url from Step 4>"
  "Confidence": "<High, Medium, or Low>"
```

For Needs Sorting items, set Status to "Needs Sorting" instead of "Routed".

### Step 6: Confirm

Confirm briefly — the user is working, not doing admin.

**Routed successfully:**
> Logged to [Category]: "[task summary]"

**User was asked to clarify:**
> Logged to [Category] (you confirmed): "[task summary]"

**Needs Sorting:**
> Captured "[task summary]" but couldn't classify confidently — added to Needs Sorting.

**Partial failure (topic write failed):**
> Captured "[task summary]" to the master log, but couldn't route to [Category] — will need manual sorting.

Keep it to one or two sentences. Always include the category name.

## Reviewing Tasks

If the user asks to see their tasks, check intake, or review what's been captured:

1. Query the master intake database for recent entries
2. Present in a clean, scannable format grouped by category
3. Show count summary (e.g., "8 tasks captured today: 3 Technical/Dev, 2 Personal, 2 Shopping/Errands, 1 Needs Sorting")
4. Highlight any "Needs Sorting" items and offer to classify them

## Adding a New Category

To add a new category to this system:

1. Create a new topic database under the Universal Task Capture parent page (page ID: `309fb3a7-cad4-8124-83dd-ecf36323c0ec`) with domain-specific properties
2. Add the category to the Master Task Intake "Category" select options
3. Add the category + data source ID to the Topic Databases table in this skill file
4. Add classification examples to the category table in Step 3
5. Add the routing template to Step 4
