---
name: support_agent_admin
description: Nova support-agent-admin MCP — read before preview_document / tools/call.
---

# Support Agent Admin MCP — operator guide

Read this resource before calling tools on **Support Agent Admin** (`https://nova.civai.co/mcp/agents/support-agent-admin`).

## Auth and scope

| Item | Detail |
|------|--------|
| **Auth** | Programmatic Nova API key JWT — `Authorization: Bearer <token>` or `X-API-Key` |
| **Setup** | https://nova.civai.co/api-keys |
| **Owner scope** | Every tool runs as the signed-in Nova account email. You only see **your** Support CRM agents (`user_identifier` on disk). |
| **Not in scope** | Custom manager agents (Sales/Marketing/BD). Visitor public chat URLs. Other users' agents. |

**Tool names** are namespaced: `support_agent_admin__<action>`, `google_sheets__<action>`, etc.

**Mutating tools** (create/update/delete/send/scan/import/schedules) require **Yes/No approval** in MCP hosts that support elicitation.

---

## Mental model

```
Nova owner (API key)
  └── Support Agent Admin MCP
        ├── support_agent_admin__*  → CRM + Leads Discovery (per agent_id)
        ├── google_sheets__*        → export / trackers
        ├── google_docs__*          → call prep, templates
        ├── google_calendar__*      → follow-up meetings
        └── deep_research__*        → prospect enrichment (not pure CRM reads)
```

**Support Agent** = visitor-facing outreach/CRM agent (WhatsApp, email, IG, X, Telegram, in-app).

**agent_id** — copy verbatim from `list_agents` / `get_agent`. Never guess.

---

## Typical flows

### Read pipeline

1. `support_agent_admin__list_agents` or `list_top_agents`
2. `get_agent` / `get_agent_analytics` for one agent
3. `list_sessions` — **requires `filter_by`** (e.g. `needs_human_reply`, `has_email`, date range)
4. `get_chat` — full thread for one `visitor_id`

### Write / outreach

1. Resolve `agent_id` + `visitor_id` (or channel handle from analytics/chat)
2. `compose_draft` — AI draft for one session (credits)
3. `send_message` / `send_followup` — pre-approved text only; **not** agent instructions
4. `bulk_followup` — batch outreach (approval)

### Leads Discovery (LD)

1. `list_connectors` — which channels are connected for this agent
2. `suggest_targets` — IG/X only (AI proposes @handles)
3. `run_scan` — start harvest (approval)
4. `continue_scan` — if scan awaits group/target selection
5. `list_scans` / `get_scan` — history + paginated leads (`imported: true|false` per lead)
6. `import_leads` — CRM import selected `lead_ids` (approval)
7. Schedules: `list_scan_schedules` → `create_scan_schedule` / `update_scan_schedule` / `delete_scan_schedule` / `trigger_scan_schedule`

### Agent settings

- `create_agent` / `update_agent` — partial patch (`instructions`, `context`, `title`, `contact_option`, **`allow_list`** visitor gate, **`auto_reply_allowed_channels`** channel gate, **`followup_*_message`** outreach openings including **`followup_email_message`** for inbound CRM first reply, public fields, **`x_ld_*` X LD public attention** for Lane A/B trend campaigns)
- `get_agent` returns the same outreach/access fields so you can read openings before patching
- `upload_attachments` — KB files (PDF, XLSX, CSV, TXT) on the agent
- `generate_agent_preview` — AI draft title/instructions/context from a business blurb

**Example — set email opening only:**

```json
{
  "agent_id": "<from list_agents>",
  "followup_email_message": "Thanks for emailing us. To help, please share: …"
}
```

**Example — email-only auto-reply + open visitor list:**

```json
{
  "agent_id": "<id>",
  "auto_reply_allowed_channels": ["email"],
  "allow_list": []
}
```

(`allow_list` empty = all visitors allowed; non-empty = only those handles/emails/seller ids.)

---

## Tool reference (support_agent_admin__)

### Agents

| Tool | Purpose |
|------|---------|
| `list_agents` | All your Support agents |
| `list_top_agents` | Busiest by recent activity |
| `get_agent` | Full metadata + chat/public/iframe links |
| `get_agent_analytics` | Totals + top visitors |
| `create_agent` | New CRM agent ✏️ |
| `update_agent` | Partial settings patch ✏️ |
| `delete_agent` | Permanent delete + sessions ✏️ |
| `transfer_agent_owner` | Move to another subscribed user ✏️ |
| `list_agent_collaborators` | Team members on an agent (owner) |
| `invite_agent_collaborator` | Add teammate by email ✏️ |
| `update_agent_collaborator` | Change role/permissions ✏️ |
| `remove_agent_collaborator` | Revoke teammate access ✏️ |
| `generate_agent_preview` | AI playbook preview |
| `upload_agent_icon` | Agent avatar ✏️ |

### Sessions & messaging

| Tool | Purpose |
|------|---------|
| `list_sessions` | Filtered session list (`filter_by` required) |
| `get_chat` | Conversation for one visitor |
| `pending_reply_count` | Count awaiting human reply |
| `compose_draft` | AI reply draft for one visitor ✏️ |
| `compose_bulk_draft` | Bulk draft generation ✏️ |
| `send_message` | Outbound to visitor ✏️ |
| `send_followup` | Follow-up message ✏️ |
| `bulk_followup` | Batch follow-up ✏️ |
| `end_conversation` / `resume_conversation` | Session state ✏️ |
| `bulk_delete_sessions` | Delete sessions ✏️ |
| `list_session_labels` / `update_session_label` / `set_session_labels` / `bulk_set_session_labels` | Label catalog + per-session labels ✏️ |
| `export_contacts` | Export CRM contacts ✏️ |
| `preview_followup_email` | HTML email preview |

### Attachments (knowledge base)

| Tool | Purpose |
|------|---------|
| `list_attachments` | KB files on agent |
| `get_attachment_content` | Schema summary or markdown text |
| `upload_attachments` | Upload base64 files ✏️ |
| `regenerate_attachment` | Re-index attachment ✏️ |
| `delete_attachment` | Remove KB file ✏️ |

**Note:** Attachment KB ≠ live Google Sheets/Docs links. Upload files via `upload_attachments`.

### Channel assignment (agent ↔ account link)

| Tool | Purpose |
|------|---------|
| `list_channel_accounts` | Accounts you can assign (IG, X, TG, SMTP; WA note in response) |
| `assign_whatsapp_account` | Link or clear WA account on agent ✏️ |
| `assign_instagram_account` | Link or clear IG account ✏️ |
| `assign_x_account` | Link or clear **main** X account (inbound + outbound) ✏️ |
| `assign_x_campaign_account` | Link or clear **campaign** X (outbound only; handle must already be main on another agent) ✏️ |
| `assign_telegram_account` | Link or clear Telegram account ✏️ |
| `assign_smtp_account` | Link or clear SMTP account ✏️ |
| `set_pool_mode` | Pool fallback for x / instagram / telegram ✏️ |

**This MCP does not connect OAuth, QR, or SMTP credentials.** It only assigns **already-connected** accounts. See § Channel integrations below.

### Leads Discovery

| Tool | Purpose |
|------|---------|
| `list_connectors` | WA / IG / email / X / TG status for agent |
| `suggest_targets` | AI @targets for IG/X scan |
| `run_scan` | Start scan ✏️ |
| `continue_scan` | Resume with selected groups/targets ✏️ |
| `list_scans` | Scan history |
| `get_scan` / `list_scan_results` | One scan + leads (`offset`/`limit`) |
| `import_leads` | Import `lead_ids` into CRM ✏️ |
| `delete_scans` | Remove scan runs ✏️ |
| `clear_discovery_cache` | Reset seen-handles cache ✏️ |
| `get_discovery_analytics` | 7d/30d sourced-leads stats |
| `create_scan_schedule` | Auto-scan ✏️ |
| `list_scan_schedules` | List schedules |
| `update_scan_schedule` | Patch / pause ✏️ |
| `delete_scan_schedule` | Remove schedule ✏️ |
| `trigger_scan_schedule` | Run now ✏️ |

### Follow-up runs

| Tool | Purpose |
|------|---------|
| `list_followup_runs` / `get_followup_run` | Bulk follow-up job history |
| `resume_followup_run` | Resume stalled run ✏️ |

### Import contacts (manual lists)

| Tool | Purpose |
|------|---------|
| `import_contacts` | Email / WA / IG / X / Telegram lists ✏️ |

### Settings extras

| Tool | Purpose |
|------|---------|
| `get_crm_unreplied_alert_config` / `update_crm_unreplied_alert_config` | Unreplied alert email ✏️ |

✏️ = mutating; requires approval in MCP clients that support it.

---

## Channel integrations — UI steps (required before assign_* works)

Support Agent Admin can **list** and **assign** channel accounts. **Connecting** accounts happens in the Nova web app (Integrations + agent settings). Without these steps, `list_connectors` shows disconnected and scans/outbound fail.

### Email / SMTP

| Step | Nova UI | Then MCP |
|------|---------|----------|
| 1 | **Integrations** or agent **Settings → Email** — add SMTP account (host, port, credentials, from address) or use Mailgun API mode | `list_channel_accounts` shows `smtp_accounts` |
| 2 | Agent **Settings → Channels** — pick SMTP account | `assign_smtp_account` with `smtp_account_id` |
| 3 | Set agent `contact_option: email` via `update_agent` if needed | `import_contacts` / email outreach |

### WhatsApp Web

| Step | Nova UI | Then MCP |
|------|---------|----------|
| 1 | **Integrations → WhatsApp Web** — scan QR, wait for connected | Account appears in integrations registry |
| 2 | Agent **Settings → Channels** — assign WhatsApp account | `assign_whatsapp_account` |
| 3 | `list_connectors` → `whatsapp: connected` | `run_scan` with `channel: whatsapp` |

### Instagram

| Step | Nova UI | Then MCP |
|------|---------|----------|
| 1 | **Integrations → Instagram** — OAuth connect business account | IG account in registry |
| 2 | Assign to agent in UI or MCP | `assign_instagram_account` |
| 3 | `list_connectors` → instagram connected | LD scan / IG outreach |

### X (Twitter)

| Step | Nova UI | Then MCP |
|------|---------|----------|
| 1 | **Integrations → X** — OAuth connect | X account for owner |
| 2 | Pick **one** agent as **main** for that handle (inbound DMs) | `assign_x_account` |
| 3 | Extra trend/campaign agents on the **same** @handle (LD sends only) | `assign_x_campaign_account` (not `assign_x_account`) |
| 4 | Optional: **pool mode** for shared outreach accounts | `set_pool_mode` channel `x` |

**Main vs campaign:** `assign_x_account` = exclusive inbound owner per X session. `assign_x_campaign_account` = same login for outreach/Leads Discovery; replies stay on the main agent. Never set both on one agent.

**Campaign assign (already-main handles):** Call `list_channel_accounts` and use an id from **`x_campaign_accounts`** (each row’s `x_account_id` is the Nova session UUID). Pass that value as **`x_campaign_account_id`** on `assign_x_campaign_account` — not `x_account_id`, and not the Twitter numeric id from Audience Resolver `default_targets.account_id`. The handle must already be **main** on another agent (`primary_owner_agent_title` in the list). Multiple wave agents may share the same `x_campaign_account_id`.

### Telegram

| Step | Nova UI | Then MCP |
|------|---------|----------|
| 1 | **Integrations → Telegram** — phone login / session | Telegram account connected |
| 2 | Assign to agent | `assign_telegram_account` |
| 3 | LD: group picker may require `continue_scan` with `selected_resources` | `run_scan` channel `telegram` |

### Mockup — connector vs assign

```
Before UI connect:
  list_connectors → { whatsapp: disconnected, instagram: disconnected, ... }
  assign_whatsapp_account → 400 / account not found

After Integrations QR + assign:
  list_connectors → { whatsapp: connected }
  list_channel_accounts → { whatsapp_account_id: "...", ... }
  assign_whatsapp_account { agent_id, whatsapp_account_id } → OK
  run_scan { channel: whatsapp, params: { group_ids: [...] } } → leads
```

**Future:** Full channel CRUD (create SMTP, OAuth flows, QR) belongs in a separate **Integrations MCP** — not Support Agent Admin.

---

## Bundled capabilities (same MCP endpoint)

| Prefix | Use for |
|--------|---------|
| `google_sheets__*` | Export sessions/leads to sheets |
| `google_docs__*` | Call prep docs, templates |
| `google_calendar__*` | Schedule follow-up calls |
| `deep_research__*` | Company/lead research — not for `list_sessions` |

Route by job: CRM/LD → `support_agent_admin__*`; export → Sheets; narrative → Docs.

---

## Leads: imported vs not

`get_scan` returns per-lead:

```json
{
  "handle": "example_user",
  "imported": false,
  "imported_visitor_id": null,
  "lead_id": "ld_..."
}
```

- **`imported: true`** — already in CRM (`imported_visitor_id` set). Scheduled runs with `auto_import: true` import on completion.
- **`imported: false`** — still only in scan storage. Call `import_leads` with `lead_ids` and optional `outreach: none|immediate|bulk_followup`.

`get_discovery_analytics` summarizes found vs imported over 7d/30d.

### Region/country filter (X only)

`run_scan` / `create_scan_schedule` / `update_scan_schedule` accept an optional
`params.region_filter_countries` on **channel `x`**: a list of country names, ISO 3166-1
alpha-2 codes, **and/or continent names** (e.g. `["Nigeria"]`, `["NG"]`, `["Africa"]`, or a mix
like `["NG", "Europe"]`), OR'd together. Validated and normalized server-side
(`utils/leads_discovery/region_match.normalize_region_filter_countries`); an unrecognised
country/continent is rejected (400), not silently dropped.

A continent target accepts **any** country within it, even one not individually listed (e.g.
`["Africa"]` accepts a Kenya bio without Kenya being named), as well as a bio that only names
the continent itself ("Africa", "West Africa 🌍"). When both a country and its continent are
targeted (`["NG", "AFRICA"]`), a matching bio resolves to the specific country, not the
continent. See `region_match.resolve_region_filter_match` for the exact precedence.

**Signal priority:** X's "About this account" page `account_based_in` field (signup/App Store
country — e.g. `x.com/<user>/about`) is the primary signal, checked first; the user-typed bio
`location` ("Lagos, Nigeria", "Lasgidi 🇳🇬") is only a **fallback** when `account_based_in` is
unavailable (about-page fetch failed, or X has none for that account). `account_based_in` is
far more often populated than bio location (most leads never set the bio location field at
all) and reflects X's own account-level signal, not something the user had to type. When
`account_based_in` IS present but doesn't match the filter, that's a confident reject — bio
location is not re-checked. Matched against the same `region_match.py` alias list either way.
`lead["region_filter_signal"]` on a scanned lead says which one decided it:
`account_based_in` / `bio_location` / `none`.

**Caveat:** neither signal is a verified, government-issued location. `account_based_in` is
X's own derived "account based in" field and ships its own caveat flags
(`created_country_accurate`, `location_accurate` — not currently surfaced, see
`services/x-agent/index.js` `/get-about-account`); bio location is pure free text. A lead with
no usable signal at all, or one Nova's matcher doesn't recognise, is **rejected**, not passed
through — this filter narrows a scan, it does not guarantee every match is correct. When set,
every lead gets up to two live profile fetches (`getUser` + `getAboutAccount`, one extra X API
call each) even if the agent's LLM classifier (`ld_classifier_enabled`) is off.

A bio that only names a continent/region **not** among the filter's targets, implied only by
the countries listed (e.g. filter `["NG"]`, bio "Africa") — is also rejected by default, but
with its own reject reason (`region_filter_continent_only`, not lumped in with a truly unknown
location) so operators can spot-check that bucket separately. Add the continent itself to the
target list to accept that case instead of just flagging it.

---

## Guardrails

1. **Never send** `instructions` or `context` to visitors — use `compose_draft` or fixed generic fallback only.
2. **Copy IDs** from tool results — do not invent `agent_id` / `visitor_id`.
3. **`list_sessions`** without `filter_by` fails validation.
4. **Ownership** — other users' `agent_id` returns 404, not their data.
5. **Credits** — `compose_draft`, `compose_bulk_draft`, and MCP `tools/call` debit Nova credits.
6. **Channel cold outreach** — Telegram group reply and X DM rules apply; see `docs/crm/ADDING_A_CRM_CHANNEL.md` in the API repo.

---

## Related docs (API repo)

| Path | Topic |
|------|--------|
| `docs/SUPPORT_AGENT_ATTACHMENTS.md` | Attachment KB |
| `docs/crm/ADDING_A_CRM_CHANNEL.md` | Channel matrix, LD harvest |
| `config/mcp-listings/README.md` | Publish Registry + Smithery |
| `constraints.md` | MCP auth, bundled vs virtual agents |

---

## Resource URI

This file is exposed as an MCP resource:

- **URI:** `civai-nova://support-agent-admin/SKILLS.md`

Call `resources/list` then `resources/read` on the Support Agent Admin MCP endpoint (API key required for read).
