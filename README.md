# AI Content Generation and Publishing Agent

An n8n workflow that turns a research update into a LinkedIn post, a newsletter summary and a website announcement using Google Gemini, then publishes **only after explicit human approval**.

Built with **n8n**, **Google Gemini**, **Google Sheets** and **Telegram**. Publishing is demonstrated against a **mock API** (test mode). All data is fictional.

## Workflows

| File | Purpose |
|---|---|
| `Part2A_Generate_Content.json` | Form → Gemini → 3 drafts saved with status `PENDING_APPROVAL` → Telegram notification to the reviewer |
| `Part2B_Review_and_Publish.json` | Review form (approve, reject or edit, plus channel selection) → rules → publish → log → notification |

## How it works

```
Form (project update) -> Gemini (LinkedIn, Newsletter, Website)
  -> Sheet "Content" (PENDING_APPROVAL) -> Telegram notification (content ID)

Review form (ID, decision, channels, optional edits)
  -> Read row -> Decide
       reject  -> status REJECTED
       edits   -> save edits, status back to PENDING_APPROVAL, NOT published
       approve -> status APPROVED -> publish each selected channel (mock API)
                -> "PublishLog" (status, URL, timestamp)
                -> status PUBLISHED / PUBLISH_FAILED
                -> Telegram notification (success or failure)
```

**Generated content:** LinkedIn post (80-150 words with hashtags), newsletter summary (60-100 words), website announcement (headline plus article with intro, key findings and next steps). The AI is told to use only the supplied facts.

**Statuses:** `PENDING_APPROVAL`, `APPROVED`, `REJECTED`, `PUBLISHED`, `PUBLISH_FAILED`.

## Safety rules

1. Nothing is published unless the reviewer submits *Approve & Publish* for content in `PENDING_APPROVAL` (or `PUBLISH_FAILED` for a retry).
2. **Any edit resets the status to `PENDING_APPROVAL`** and nothing is published. The reviewer must approve the updated version again.
3. Published content cannot be approved again.
4. At least one channel must be selected.

Every channel attempt is recorded in `PublishLog` with status, URL, timestamp and message.

## Setup

1. In n8n, import both JSON files (**Workflow menu → Import from file**).
2. Create credentials: **Telegram API**, **Google Sheets OAuth2**, **Google Gemini API**.
3. Create a Google Sheet with two tabs and these headers in row 1:
   - `Content`: `content_id, title, source_text, linkedin, newsletter, website, status, channels, edited, created_at, approved_at, published_at`
   - `PublishLog`: `content_id, channel, status, published_url, timestamp, message`
4. In every Google Sheets node choose your document and tab. In every Telegram node replace `YOUR_CHAT_ID` with your chat ID.
5. Run Part 2A from its form, then Part 2B using the content ID received on Telegram.

## Error handling

- Gemini calls retry automatically on temporary failures (up to 4 tries).
- Each publishing channel is logged separately, so one failure does not hide the others. A failed channel is recorded as `FAILED` and the content can be approved again.

## Limitations and production notes

- Publishing uses a mock API (`jsonplaceholder.typicode.com`), so published URLs are placeholders. In production, replace the mock HTTP nodes with the WordPress REST API for the website, the LinkedIn API for the company page, and the email platform's API for the newsletter.
- The reviewer enters the content ID manually. A production version could send a pre-filled link.
- The review form is not access-controlled in this prototype. In production, restrict access so only authorized team members can approve.

*All data is fictional. No credentials are included.*
