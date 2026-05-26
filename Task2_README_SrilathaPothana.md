# Task 2 – n8n Workflow README

**Workflow name:** HackerNews Morning Brief  
**File:** `Task2_Workflow_SrilathaPothana.json`

---

## APIs Used & Why

| API | Endpoint | Why |
|-----|----------|-----|
| HackerNews Firebase API | `/v0/topstories.json` + `/v0/item/{id}.json` | Free, no auth required, returns real-time top story IDs and full story details (title, score, author, comments) |
| Algolia HN Search API | `/api/v1/search?tags=story&query=...` | Free, no auth. Used to enrich high-score stories with additional metadata (view count, alternative ranking) |
| Discord Webhook | Incoming webhook URL | Free, instant delivery, supports Markdown formatting — ideal for team digests |

---

## Workflow Flow

```
Schedule (1h)
  → HN: Get top 500 story IDs
  → Code: Slice to first 5 IDs
  → HN: Fetch full details for each story (runs 5×)
  → IF: score > 100?
      YES → Algolia: Enrich with extra metadata → Merge → Build digest
      NO  → Label as NORMAL → Build digest
  → Discord: Post formatted digest
```

---

## Transformation Logic

The **Code node "Transform – Top 5 IDs"** slices the HackerNews top story array (which returns up to 500 IDs) to just the first 5:

```js
const ids = $input.first().json.slice(0, 5);
return ids.map(id => ({ json: { id } }));
```

Each ID is then fetched individually via the HN item endpoint. This runs in parallel across 5 items.

The **"Build – Discord Digest Message"** node aggregates all 5 results (from both IF branches) into a single formatted Discord message using emoji badges to distinguish high-score (🔥) vs normal (📰) stories.

---

## IF Branch (Conditional Logic)

- **Threshold:** `score > 100`
- **TRUE branch (High score):** Story is enriched via Algolia's search API, which returns additional engagement data. This adds context to stories that are already trending.
- **FALSE branch (Normal score):** Story is passed through as-is with a `NORMAL` tier label — still included in the digest, just without enrichment.
- **Rationale:** Algolia calls add latency; limiting enrichment to high-value stories keeps the workflow fast.

---

## Error Handling

Every HTTP Request node has **`onError: continueErrorOutput`** configured. This means:

- If the HN top stories call fails → routed to the **Error Handler node**, which logs the error to console with a timestamp and node name. The workflow does **not** crash.
- If an individual story fetch fails → same error output routing; the remaining stories still process.
- If the Algolia enrichment fails → falls back silently; the main story data is still included.
- If the Discord webhook call fails → logged to the Error Handler. A future improvement would be a fallback email notification.

The **Error Handler** Code node formats a structured log entry:
```js
console.log(`[ERROR ${ts}] Node: ${nodeName} | Message: ${errorMessage}`);
```

In production, this node would be extended to write errors to a Google Sheet or trigger a secondary alert.

---

## Credentials Setup

**Never hardcode secrets.** In n8n:

1. Go to **Settings → Credentials → New**
2. Create a credential of type **"HTTP Custom Auth"** named `Discord Webhook`
3. Set the webhook URL as a credential field
4. Reference it in the Discord node via `$credentials.discordWebhookUrl`

This keeps secrets out of the workflow JSON and out of version control.

---

## How to Import

1. Open your n8n instance
2. Click **Workflows → Import from File**
3. Select `Task2_Workflow_SrilathaPothana.json`
4. Set up your Discord Webhook credential (see above)
5. Activate the workflow — it will fire every hour automatically

---

## Bonus: Uptime Monitor

See `Bonus_UptimeMonitor_SrilathaPothana.json`. This second workflow:
- Pings `demo.realworld.io` every **5 minutes** via Schedule trigger
- Evaluates the HTTP status code
- If non-200: retries once after 30 seconds to avoid false positives, then posts a 🚨 alert to the same Discord webhook
- At **08:00 daily**: sends a summary message via a second Schedule trigger (cron: `0 8 * * *`)
- Silent on success (logs to console only — no notification spam)
