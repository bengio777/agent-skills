---
name: log-class-question
description: "Log a class question to both Notion and Google Sheets for later follow-up. Use this skill whenever the user says things like 'I have a question for my class', 'log this question', 'save this question for class', 'I want to ask this in class', 'add to my class questions', 'question for my teacher', 'class question', or mentions wanting to remember or track a question related to any course or class they're taking. Also trigger when the user says 'answer a class question' or 'update a class question' to mark a previously logged question as answered. Trigger on 'review my questions' or 'what questions do I have' to show the backlog."
---

# Log Class Question

Captures questions the user wants to ask later in class and logs them to two destinations:

1. **Notion database** — "Class Questions Tracker" (data source ID: `9f733497-7965-414b-8731-6abe0c2fc1b4`, database ID: `de19e6ca-0f46-4fa9-9492-ea1ac60c31fd`)
2. **Google Sheet** — "Class Questions Tracker" (spreadsheet ID: `1yzTkjLx6SULuxAZAATsThdbKyM0fAzQ3_pwACNILqBM`)

## Logging a New Question

Use the AskUserQuestion tool to gather details in a single prompt. Be conversational and quick — the user is studying and doesn't want friction.

### Fields to Collect

| Field | Required | How to collect |
|-------|----------|---------------|
| **Question** | Yes | Ask the user to state their question |
| **Class** | Yes | Options: Hands-on AI, Kite The Planet, Professional, Security+, Spanish, Other. If context makes it obvious (e.g. they were just studying Security+), confirm rather than ask. |
| **Priority** | No | Default "Medium". Options: High, Medium, Low |
| **Source** | No | Default "Conversation". Options: Conversation, Homework / Project Work, Lecture, Reading, Practice Exam, Other |

### Fields to Auto-Fill

- **Date Entered**: Today's date (YYYY-MM-DD)
- **Status**: "Incomplete"
- **Date Answered**: Leave empty
- **Answer**: Leave empty

### Logging Procedure

Once you have the details, log to BOTH destinations concurrently:

**Notion** — Create a page in the database:
```
Tool: notion-create-pages
parent: { "data_source_id": "9f733497-7965-414b-8731-6abe0c2fc1b4" }
pages: [{
  properties: {
    "Question": "<question text>",
    "Class": "<class>",
    "Status": "Incomplete",
    "Priority": "<priority or Medium>",
    "Source": "<source or Conversation>",
    "date:Date Entered:start": "<YYYY-MM-DD>",
    "date:Date Entered:is_datetime": 0
  }
}]
```

**Google Sheet** — Read the sheet to find the last row, then append:
```
Tool: get_google_sheet_content
  spreadsheetId: 1yzTkjLx6SULuxAZAATsThdbKyM0fAzQ3_pwACNILqBM

Then: update_google_sheet to row N+1 with:
  [Question, Class, "Incomplete", Priority, Source, Date Entered, "", ""]
```

If either destination fails, the other is still sufficient — confirm what was logged.

The Google Sheet has conditional formatting applied:
- Red rows = Incomplete
- Green rows = Answered
- Orange rows = Follow Up

### After Logging

Confirm briefly, like: "Logged your question to Google Sheets. You now have X total questions tracked (Y incomplete)."

If Notion worked too, say: "Logged to both Notion and Google Sheets."

Keep it short — they're studying, not doing admin.

## Answering / Updating a Question

If the user wants to answer or update a class question:

1. Read the Google Sheet to get all questions
2. Filter for incomplete ones and show a numbered list
3. Let them pick which one to update
4. Collect the answer text
5. Update Google Sheets:
   - Set the Status column to "Answered" (or "Follow Up" if they say so)
   - Set Date Answered to today (YYYY-MM-DD)
   - Set Answer to the provided text
   - If status is "Answered", move the row to the "Answered" sheet tab in Google Sheets and delete it from Sheet1
6. Update Notion too (notion-update-page) with the same status, date answered, and answer
   - In Notion, setting Status to "Answered" automatically moves the entry to the "Answered" filtered view

## Reviewing Questions

If the user asks to see their questions or check their backlog:

1. Read the Google Sheet to get all questions
2. Present in a clean, scannable format grouped by class
3. Show count summary (e.g., "12 incomplete: 8 Security+, 3 Spanish, 1 Hands-on AI")
4. If the user wants to drill into a specific class, filter and show just those

## Notion Views

The Class Questions Tracker uses filtered views:
- **Active** view: Shows only Incomplete and Follow Up questions (default view)
- **Answered** view: Shows only Answered questions

When a question's status changes to "Answered", it automatically disappears from the Active view and appears in the Answered view. No page moves are needed.
