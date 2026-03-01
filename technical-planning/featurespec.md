# ZeroClaw AI — Feature Specification Document v3.0

> Detailed Feature Specs, Acceptance Criteria & API Contracts
> February 28, 2026

---

## Table of Contents

1. [Feature Overview & Phase Map](#1-feature-overview--phase-map)
2. [F-001: Email & Inbox Management](#2-f-001-email--inbox-management)
3. [F-002: Calendar & Scheduling](#3-f-002-calendar--scheduling)
4. [F-003: Daily Brief & Routine Planning](#4-f-003-daily-brief--routine-planning)
5. [F-004: Personal Task Assistant](#5-f-004-personal-task-assistant)
6. [F-005: Multi-Channel Assistant](#6-f-005-multi-channel-assistant)
7. [F-006: Second Brain (Memory Recall)](#7-f-006-second-brain-memory-recall)
8. [F-007: News / Digest Summaries](#8-f-007-news--digest-summaries)
9. [F-008: Meal / Health Planning](#9-f-008-meal--health-planning)
10. [F-100: Onboarding & Auth](#10-f-100-onboarding--auth)
11. [F-200: Core Agent Chat](#11-f-200-core-agent-chat)
12. [F-300: Container Lifecycle (Internal)](#12-f-300-container-lifecycle-internal)
13. [F-400: Scrapling Deep Research](#13-f-400-scrapling-deep-research)
14. [F-500: Social Media & Content](#14-f-500-social-media--content)
15. [F-600: Integrations Hub & Skills Marketplace](#15-f-600-integrations-hub--skills-marketplace)
16. [F-700: Payments & Subscriptions](#16-f-700-payments--subscriptions)
17. [Non-Functional Requirements](#17-non-functional-requirements)

---

## 1. Feature Overview & Phase Map

### 1.1 Phase Delivery Schedule

```
PHASE 0 (Weeks 1-2): Foundation
  F-100  Onboarding & Auth
  F-300  Container Lifecycle (internal)

PHASE 1 (Weeks 3-5): Core Agent
  F-001  Email & Inbox Management
  F-002  Calendar & Scheduling
  F-006  Second Brain (Memory)
  F-200  Core Agent Chat

PHASE 2 (Weeks 6-8): Secretary ──── MVP LAUNCH ────
  F-003  Daily Brief & Routine Planning
  F-004  Personal Task Assistant
  F-007  News / Digest Summaries

PHASE 3 (Weeks 9-11): Channels
  F-005  Multi-Channel (Telegram, WhatsApp, Voice)
  F-600  Integrations Hub + Skills Marketplace

PHASE 4 (Weeks 12-16): Research + Content
  F-400  Scrapling Deep Research
  F-500  Social Media & Content

PHASE 5 (Weeks 17-20): Payments + Health
  F-700  Payments & Subscriptions
  F-008  Meal / Health Planning

PHASE 6 (Weeks 21-24): Polish + Launch
  Beta testing, App Store submission, public launch
```

### 1.2 Feature Dependency Graph

```
F-100 (Auth) ──────────────────────────────────────────┐
  │                                                      │
F-300 (Container Lifecycle) ───────────────────────────┤
  │                                                      │
F-200 (Chat) ──┬── F-001 (Email) ── F-003 (Brief)     │
               │                                        │
               ├── F-002 (Calendar) ── F-003 (Brief)   │
               │                                        │
               ├── F-006 (Memory) ─── all features      │
               │                                        │
               ├── F-004 (Tasks)                        │
               │                                        │
               ├── F-007 (News)                         │
               │                                        │
               ├── F-005 (Multi-Channel)                │
               │                                        │
               ├── F-400 (Research)                     │
               │                                        │
               └── F-500 (Social) ── F-400 (Research)   │
                                                        │
F-600 (Integrations Hub) ─────────────── F-001, F-002  │
                                                        │
F-700 (Payments) ──────────────────── gates all tiers ──┘
```

---

## 2. F-001: Email & Inbox Management

**Priority:** #1 · **Phase:** 1 · **Story Points:** 8 · **Tier:** Starter+

### 2.1 Description

The agent connects to the user's email (Gmail/Outlook) via Composio, reads the inbox on a configurable schedule, categorizes every message, summarizes newsletters into one clean digest, and drafts replies to urgent items for user approval.

### 2.2 User Flows

**Flow A — Initial Setup:**
```
User taps "Connect Email" in Integrations Hub
  → Composio OAuth flow (Gmail or Outlook)
  → On success: Edge Function stores composio_entity_id
  → Agent container receives connected event
  → Agent runs first inbox scan immediately
  → Shows categorized inbox summary in chat
```

**Flow B — Daily Digest (Automated):**
```
Scheduler triggers at user's configured time (default 8:00 AM)
  → Gateway starts user container (if stopped)
  → Agent calls Composio Gmail tool: fetch unread emails since last scan
  → LLM categorizes each email: URGENT | ACTION | FYI | PROMO | NEWSLETTER | SPAM
  → Newsletters: LLM summarizes content, strips formatting, extracts key links
  → Composes digest: urgent items first, then action, then FYI
  → Saves digest to workspace/drafts/inbox-digest-{date}.md
  → Push notification: "Your inbox digest is ready — 3 urgent, 7 FYI"
  → In-app: digest displayed as a structured card in chat timeline
```

**Flow C — Draft Reply (Supervised):**
```
Agent identifies urgent email needing reply
  → LLM generates draft reply based on email context + user tone/style
  → Sends approval card to user in chat:
    ┌──────────────────────────────────────────┐
    │ 📧 Draft Reply to: Sarah (Re: Q1 Report) │
    │                                           │
    │ "Hi Sarah, thanks for sending this over.  │
    │  I've reviewed the numbers and they look  │
    │  aligned with our projections. Let's       │
    │  discuss the variance in Section 3 during  │
    │  our Thursday standup."                    │
    │                                           │
    │  [✓ Send]  [✏️ Edit]  [✕ Discard]         │
    └──────────────────────────────────────────┘
  → User taps Send → Agent executes Composio Gmail send
  → User taps Edit → Opens draft in editable text field
```

### 2.3 Acceptance Criteria

| # | Criterion | Validation |
|---|-----------|-----------|
| AC-1 | Gmail or Outlook connects via Composio OAuth in <30 seconds | Manual QA |
| AC-2 | Inbox scan processes up to 200 unread emails per run | Load test |
| AC-3 | Each email categorized into one of 6 categories (URGENT/ACTION/FYI/PROMO/NEWSLETTER/SPAM) | Sample accuracy ≥90% on 50-email test set |
| AC-4 | Newsletters summarized to ≤150 words each with source links preserved | Manual review |
| AC-5 | Daily digest delivered within ±5 minutes of configured time | Scheduler test |
| AC-6 | Draft replies match user's tone (learned from previous emails over time) | User feedback loop |
| AC-7 | No email is sent without explicit user approval in supervised mode | Security audit |
| AC-8 | Push notification delivered for digest with accurate count summary | Device testing |
| AC-9 | Email history stored in memory.db for future reference ("What did Sarah email about last week?") | Memory recall test |

### 2.4 API Contracts

**Edge Function → Gateway (trigger digest):**
```json
POST /api/agent/action
Authorization: Bearer {gateway_secret}
{
  "user_id": "abc123",
  "action": "inbox_digest",
  "params": {
    "provider": "gmail",
    "max_emails": 200,
    "since": "2026-02-27T00:00:00Z"
  }
}
```

**Gateway → User Container:**
```json
POST http://zc-abc123:8080/action
{
  "action": "inbox_digest",
  "params": { ... }
}
```

**Container Response (digest ready):**
```json
{
  "status": "complete",
  "result": {
    "total_emails": 47,
    "categories": {
      "urgent": 3,
      "action": 8,
      "fyi": 12,
      "promo": 15,
      "newsletter": 6,
      "spam": 3
    },
    "digest_markdown": "## Inbox Digest — Feb 28...",
    "draft_replies": [
      {
        "email_id": "msg-123",
        "from": "sarah@company.com",
        "subject": "Re: Q1 Report",
        "draft_body": "Hi Sarah, thanks for...",
        "requires_approval": true
      }
    ],
    "newsletter_summaries": [
      {
        "source": "Morning Brew",
        "summary": "Key highlights: ...",
        "link": "https://..."
      }
    ]
  }
}
```

### 2.5 Composio Tools Used

| Tool | Action | Rate Limit |
|------|--------|-----------|
| `gmail_fetch_emails` | Read unread emails with filters | 200/scan |
| `gmail_send_email` | Send approved draft reply | 50/day (Starter), 200/day (Pro) |
| `gmail_create_draft` | Save draft without sending | Unlimited |
| `outlook_fetch_emails` | Same as Gmail for Outlook | 200/scan |
| `outlook_send_email` | Same as Gmail for Outlook | Tier-based |

### 2.6 Edge Cases

| Scenario | Handling |
|----------|---------|
| OAuth token expired | Composio auto-refreshes. If refresh fails, push notification: "Please reconnect your email" |
| >200 unread emails | Process most recent 200, note remainder: "47 older emails skipped" |
| Email in non-English language | Detect language, summarize in user's preferred language |
| Attachment-heavy emails | Note "has attachment: budget.xlsx (2.3MB)" without processing content |
| User has multiple email accounts | Support up to 3 connected accounts. Digest merges all. |
| Spam/phishing detection | Flag suspicious links. Never auto-click links in emails. |

---

## 3. F-002: Calendar & Scheduling

**Priority:** #2 · **Phase:** 1 · **Story Points:** 8 · **Tier:** Starter+

### 3.1 Description

Agent syncs with Google Calendar or Outlook via Composio. Creates, moves, cancels events. Detects conflicts and suggests optimal slots. Auto-schedules recurring routines (gym, deep work, lunch). Pushes prep notes 15 minutes before meetings.

### 3.2 User Flows

**Flow A — Natural Language Scheduling:**
```
User: "Schedule a dentist appointment next Tuesday at 2pm"
  → Agent checks calendar for conflicts
  → If conflict: "You have 'Team Standup' at 2pm. Want me to reschedule to 3pm?"
  → If clear: Creates event via Composio → Confirmation card in chat
  → Saves to memory: "User has dentist appointment Tue 2pm"
```

**Flow B — Smart Conflict Resolution:**
```
User: "I need 2 hours for deep work tomorrow"
  → Agent scans tomorrow's calendar for free blocks
  → Identifies 3 possible slots: 8-10am, 11am-1pm, 3-5pm
  → Suggests best slot based on user patterns (learned from memory):
    "Based on your past focus sessions, I'd suggest 8-10am. You 
     tend to do your best deep work in the morning. Want me to block it?"
  → User confirms → Event created with "Do Not Disturb" flag
```

**Flow C — Meeting Prep (Automated):**
```
15 minutes before any calendar event:
  → Agent checks event attendees
  → If attendees found: quick Scrapling lookup on LinkedIn/company
  → Generates prep note:
    ┌──────────────────────────────────────────┐
    │ 📅 Meeting in 15min: Q1 Review           │
    │                                           │
    │ 👤 Attendees: Sarah (PM), Ahmed (Eng)     │
    │ 📝 Last interaction: Discussed Q1 report  │
    │    variance on Feb 20                     │
    │ 🎯 Suggested agenda:                      │
    │    - Address Section 3 variance           │
    │    - Review updated projections            │
    │    - Confirm Q2 timeline                  │
    │                                           │
    │ [Open Calendar]  [Dismiss]                │
    └──────────────────────────────────────────┘
  → Push notification delivered
```

### 3.3 Acceptance Criteria

| # | Criterion | Validation |
|---|-----------|-----------|
| AC-1 | Google Calendar or Outlook connects via Composio in <30s | Manual QA |
| AC-2 | Natural language creates events with correct date, time, title, duration | 50-query test set, ≥95% accuracy |
| AC-3 | Conflict detection catches overlapping events within 1-minute precision | Unit tests |
| AC-4 | Recurring routines (gym, deep work) can be set once and auto-schedule weekly | Scheduler integration test |
| AC-5 | Meeting prep notification arrives 15min before event (±2min) | Timing test |
| AC-6 | Prep notes include attendee context from memory.db if available | Memory recall test |
| AC-7 | Calendar modifications require user approval in supervised mode | Security audit |
| AC-8 | Timezone handling correct for user's configured timezone | Timezone edge case tests |

### 3.4 Composio Tools Used

| Tool | Action |
|------|--------|
| `google_calendar_list_events` | Fetch events for date range |
| `google_calendar_create_event` | Create new event |
| `google_calendar_update_event` | Move/reschedule event |
| `google_calendar_delete_event` | Cancel event |
| `outlook_calendar_*` | Same operations for Outlook |

---

## 4. F-003: Daily Brief & Routine Planning

**Priority:** #3 · **Phase:** 2 · **Story Points:** 5 · **Tier:** Starter+

### 4.1 Description

The single most popular OpenClaw setup. Every morning at the user's wake time, the agent delivers a personalized briefing: weather, today's calendar, top 3 priority tasks, news headlines from preferred sources, goal progress, and one motivational nudge. Delivered as push notification + in-app timeline card. Zero configuration required beyond setting wake time during onboarding.

### 4.2 Brief Template

```
┌──────────────────────────────────────────────────┐
│ ☀️ Good morning, Alia! — Friday, Feb 28           │
│                                                    │
│ 🌡️ Weather: 24°C, sunny. High 28°C.               │
│                                                    │
│ 📅 Today's Schedule:                               │
│   09:00  Team standup (Zoom)                       │
│   11:00  Deep work block                           │
│   14:00  Dentist appointment                       │
│   16:00  Client call — prep notes ready             │
│                                                    │
│ ✅ Top 3 Tasks:                                     │
│   1. Review Q1 report variance (Goal: Career)      │
│   2. Draft blog post outline (Goal: Content)        │
│   3. 30-min workout (Goal: Fitness) 🔥 5-day streak │
│                                                    │
│ 📰 Headlines:                                       │
│   • Pakistan startup raises $12M Series A           │
│   • New AI model beats GPT-4 on reasoning          │
│   • Bitcoin crosses $95K                            │
│                                                    │
│ 💪 "Small consistent steps > occasional big leaps"  │
│                                                    │
│ [View Full Day]  [Adjust Plan]                      │
└──────────────────────────────────────────────────┘
```

### 4.3 Acceptance Criteria

| # | Criterion |
|---|-----------|
| AC-1 | Brief delivered at user's wake time ±3 minutes via push notification |
| AC-2 | Weather accurate for user's configured city (via free weather API) |
| AC-3 | Calendar events pulled from connected calendar (F-002) |
| AC-4 | Tasks generated from goals set during onboarding (F-004) |
| AC-5 | News headlines from user's configured sources or defaults (F-007) |
| AC-6 | Brief is ≤200 words total (concise, scannable) |
| AC-7 | Streak counter accurate and increments on task completion |
| AC-8 | Brief works even with no calendar/email connected (graceful degradation) |
| AC-9 | "Adjust Plan" opens chat with agent for interactive rescheduling |

### 4.4 Schedule File

The Scheduler service reads this file from the user's bind-mounted volume:

```json
// /data/users/{uid}/schedules/morning-brief.json
{
  "id": "morning-brief",
  "type": "cron",
  "expression": "0 7 * * *",
  "timezone": "Asia/Karachi",
  "action": "daily_brief",
  "payload": {
    "include_weather": true,
    "weather_city": "Karachi",
    "include_calendar": true,
    "include_tasks": true,
    "include_news": true,
    "news_sources": ["hackernews", "bbc_tech"],
    "include_motivation": true,
    "max_headlines": 3,
    "max_tasks": 3
  }
}
```

### 4.5 Dependencies

| Dependency | Required? | Fallback |
|-----------|-----------|---------|
| F-002 (Calendar) | Optional | "No calendar connected — connect one for schedule in your brief" |
| F-004 (Tasks) | Optional | Generate 3 generic productivity tasks |
| F-007 (News) | Optional | Use default sources (BBC, HN) |
| Weather API | Required | OpenWeatherMap free tier (1000 calls/day) |
| F-006 (Memory) | Built-in | Streak and preference tracking |

---

## 5. F-004: Personal Task Assistant

**Priority:** #4 · **Phase:** 2 · **Story Points:** 8 · **Tier:** Starter+

### 5.1 Description

Agent generates 4-5 daily tasks aligned to the user's goals (set during onboarding). Tracks completion via check-ins and push reminders. Adapts difficulty based on streaks and completion rate. Optionally integrates with Notion, Trello, or Jira via Composio to sync with existing task systems.

### 5.2 Task Generation Logic

```
EVERY MORNING (triggered by daily brief or standalone):

1. Load user goals from /agent/config.toml
   goals = ["career:get promoted", "fitness:run 5k", "learning:finish AI course"]

2. Load completion history from memory.db
   streak = 5 days, avg_completion = 72%, missed_category = "fitness"

3. LLM generates 4-5 tasks:
   - 2 tasks from highest-priority goal
   - 1 task from lowest-completion goal (catch-up)
   - 1 quick-win task (<15 min, high satisfaction)
   - 1 stretch task (optional, harder than usual if streak > 3)

4. Each task has:
   {
     "title": "Review Q1 report Section 3",
     "goal": "career:get promoted",
     "estimated_minutes": 45,
     "difficulty": "medium",
     "time_slot": "11:00 (during deep work block)",
     "is_stretch": false
   }

5. Tasks displayed in Daily Planner UI
   - Swipe right = complete
   - Swipe left = skip (agent asks why, adapts)
   - Tap = expand details + timer
```

### 5.3 Acceptance Criteria

| # | Criterion |
|---|-----------|
| AC-1 | 4-5 tasks generated daily, aligned to user's configured goals |
| AC-2 | Tasks adapt: if user completes <50% for 3 days, reduce difficulty |
| AC-3 | Tasks adapt: if user completes >90% for 3 days, add stretch task |
| AC-4 | Streak counter persists across days and resets on zero-completion day |
| AC-5 | Push reminder sent for each task at its suggested time slot |
| AC-6 | Check-in prompt at 8pm if tasks remain incomplete |
| AC-7 | Composio sync: completed tasks update Notion/Trello/Jira if connected |
| AC-8 | Task history searchable via memory ("What tasks did I do last week?") |

### 5.4 Database

```sql
-- Supabase: tasks table
CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  title TEXT NOT NULL,
  goal_category TEXT,
  estimated_minutes INT,
  difficulty TEXT CHECK (difficulty IN ('easy', 'medium', 'hard', 'stretch')),
  suggested_time TIME,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'completed', 'skipped', 'deferred')),
  skip_reason TEXT,
  created_at TIMESTAMPTZ DEFAULT now(),
  completed_at TIMESTAMPTZ
);

-- RLS: users only see their own tasks
CREATE POLICY tasks_user_only ON tasks
  FOR ALL USING (auth.uid() = user_id);
```

---

## 6. F-005: Multi-Channel Assistant

**Priority:** #5 · **Phase:** 3 · **Story Points:** 13 · **Tier:** Pro+

### 6.1 Description

Users access their agent through multiple channels: in-app chat (primary), Telegram, WhatsApp, SMS, and voice. Context is unified — start a conversation in the app, continue on Telegram, finish by voice. Channel access is gated by subscription tier.

### 6.2 Channel Access by Tier

| Channel | Starter | Pro | Power |
|---------|---------|-----|-------|
| In-app chat | ✅ | ✅ | ✅ |
| Telegram | ❌ | ✅ | ✅ |
| WhatsApp | ❌ | ❌ | ✅ |
| SMS | ❌ | ❌ | ✅ |
| Voice (in-app) | ❌ | ✅ (basic) | ✅ (custom voice) |

### 6.3 Architecture

```
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│  Mobile   │ │ Telegram │ │ WhatsApp │ │  Voice   │
│  App      │ │  Bot     │ │ Business │ │  (STT/   │
│           │ │          │ │  API     │ │   TTS)   │
└─────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘
      │            │            │            │
      ▼            ▼            ▼            ▼
┌──────────────────────────────────────────────────┐
│  Supabase Edge Function: Channel Router           │
│                                                    │
│  1. Identify user (phone number / telegram ID /    │
│     JWT token)                                     │
│  2. Check subscription tier for channel access     │
│  3. Normalize message format                       │
│  4. Forward to Gateway with user_id + channel      │
└───────────────────────┬──────────────────────────┘
                        │
                        ▼
              ┌─────────────────┐
              │  Gateway →       │
              │  User Container  │
              │  (same container │
              │   regardless of  │
              │   channel)       │
              └─────────────────┘
```

All channels route to the **same user container**. The container doesn't care which channel the message came from — it processes identically. The response is routed back through the originating channel.

### 6.4 Voice Pipeline

```
User taps microphone → Audio stream starts
  → Deepgram STT (real-time transcription, ~150ms)
  → Text sent to Gateway → Container → LLM response (~350ms-1s)
  → ElevenLabs TTS Flash v2.5 (text-to-speech, ~75-300ms)
  → Audio streamed back to user (streaming TTS — start playback before full response)
  → Transcript shown in chat alongside audio

Latency target: <3 seconds end-to-end (stretch goal: <2s)

Note: Most production voice AI agents achieve 800ms-2s under ideal conditions.
STT (~150ms) + LLM (~500ms-1s) + TTS (~75-300ms) + network (~100-200ms) = 825ms-1.7s typical.
Under load, ElevenLabs queue delays can push TTS to 600ms, exceeding 2s total.
Consider Deepgram Voice Agent API ($4.50/hr bundled) for 200-250ms total latency.
```

### 6.5 Acceptance Criteria

| # | Criterion |
|---|-----------|
| AC-1 | Telegram bot responds within 3 seconds of message receipt |
| AC-2 | Context carries across channels (start in app, continue on Telegram) |
| AC-3 | Tier enforcement: Pro user gets Telegram, Starter user gets "Upgrade to unlock" |
| AC-4 | Voice transcription accuracy ≥95% for English |
| AC-5 | Voice response latency <3 seconds end-to-end (STT + LLM + TTS). Stretch goal: <2 seconds. Use streaming TTS to reduce perceived latency. |
| AC-6 | Telegram bot handles /start, /help, /status commands |
| AC-7 | WhatsApp integration uses official Business API (compliance) |

---

## 7. F-006: Second Brain (Memory Recall)

**Priority:** #6 · **Phase:** 1 (built-in) · **Story Points:** 5 · **Tier:** All

### 7.1 Description

ZeroClaw's SQLite hybrid memory system (vector + keyword search) automatically saves important context from every conversation. The user can ask "What did I say about X last week?" and get accurate recall. Functions as a personal knowledge base that grows over time.

### 7.2 Memory Architecture

```
memory.db (SQLite, per-user volume)
│
├── conversations table
│   - message_id, role, content, timestamp, channel
│   - Full conversation history (searchable)
│
├── facts table
│   - fact_id, category, content, source_message_id, confidence
│   - Extracted facts: "User's dentist is Dr. Ahmed"
│   - Auto-extracted by LLM after each conversation
│
├── embeddings table (vector search)
│   - embedding_id, content_hash, vector (768-dim)
│   - Enables semantic search: "that thing about budgets"
│
└── preferences table
    - key, value, learned_at, confidence
    - "prefers_morning_meetings: false"
    - "tone: casual but professional"
    - "timezone: Asia/Karachi"
```

### 7.3 How Memory Works

```
EVERY CONVERSATION TURN:

1. User message saved to conversations table

2. LLM extracts notable facts (if any):
   User says: "My sister's wedding is March 15"
   → Fact: { category: "personal", content: "sister wedding March 15" }

3. Embedding generated for the message chunk
   → Stored in embeddings table for semantic search

4. On user query "What did I say about...":
   → Hybrid search: keyword match + vector similarity
   → Top 5 results ranked by relevance + recency
   → LLM synthesizes answer from matched context
```

### 7.4 Acceptance Criteria

| # | Criterion |
|---|-----------|
| AC-1 | Memory recall responds in <3 seconds for queries up to 6 months old |
| AC-2 | Semantic search finds relevant results even with different wording |
| AC-3 | Facts auto-extracted: names, dates, preferences, relationships |
| AC-4 | Memory persists across container restarts (stored on bind-mounted volume) |
| AC-5 | User can explicitly bookmark: "Remember this: my budget is $5000" |
| AC-6 | User can request deletion: "Forget everything about X" |
| AC-7 | Memory size per user stays under 500MB for 1 year of daily use |
| AC-8 | Cross-channel memory: fact learned on Telegram recalled in app chat |

---

## 8. F-007: News / Digest Summaries

**Priority:** #7 · **Phase:** 2 · **Story Points:** 8 · **Tier:** Starter+

### 8.1 Description

Configurable news sources: RSS feeds, Reddit subreddits, YouTube channels, Hacker News, X/Twitter lists. Scrapling fetches and extracts content. LLM synthesizes into a personalized daily or weekly digest scored by relevance to user interests.

### 8.2 Supported Sources

| Source | Method | Rate Limit |
|--------|--------|-----------|
| Hacker News | HN API (free) | Unlimited |
| Reddit | Reddit API (free tier) | 100 requests/min |
| RSS/Atom feeds | Scrapling fetch | 50 feeds/user |
| YouTube | YouTube Data API | 10,000 units/day |
| X/Twitter | Composio integration | Tier-based |
| Custom URLs | Scrapling StealthyFetcher | 5/day (Starter), 25/day (Pro) |

### 8.3 Digest Format

```json
{
  "digest_type": "daily",
  "generated_at": "2026-02-28T19:00:00Z",
  "sections": [
    {
      "source": "Hacker News",
      "items": [
        {
          "title": "New AI reasoning model...",
          "summary": "Researchers at DeepMind...",
          "relevance_score": 0.92,
          "url": "https://...",
          "points": 847
        }
      ]
    },
    {
      "source": "r/startups",
      "items": [...]
    }
  ],
  "total_items_scanned": 247,
  "items_included": 12,
  "reading_time_minutes": 4
}
```

### 8.4 Acceptance Criteria

| # | Criterion |
|---|-----------|
| AC-1 | User configures sources during onboarding or in settings |
| AC-2 | Digest delivered at configured time (default: with Daily Brief) |
| AC-3 | Items ranked by relevance to user's stated interests + past reading patterns |
| AC-4 | Each summary ≤50 words with original source link |
| AC-5 | Total digest ≤15 items (no information overload) |
| AC-6 | Scrapling handles paywalled sites gracefully (skip, don't crash) |
| AC-7 | Digest works standalone or embedded in Daily Brief (F-003) |

---

## 9. F-008: Meal / Health Planning

**Priority:** #8 · **Phase:** 4 (Skill) · **Story Points:** 13 · **Tier:** Pro+

### 9.1 Description

Premium skill. User sets dietary goals, restrictions, fitness targets during setup. Agent generates weekly meal plans with grocery lists. Tracks food intake via conversational logging ("I had pasta for lunch"). Spots patterns and adjusts recommendations. Optional wearable integration via Composio.

### 9.2 Conversational Food Logging

```
User: "I had two eggs and toast for breakfast"
  → Agent extracts: { meal: "breakfast", items: ["eggs x2", "toast"], calories_est: 320 }
  → Saves to memory with nutrition estimates
  → Responds: "Got it! ~320 cal. You're at 320/2000 for today. 
     Your lunch plan has a grilled chicken salad (450 cal) — still on track?"

User: "What did I eat this week?"
  → Memory recall: aggregates 7 days of food logs
  → LLM analysis: "You averaged 1,850 cal/day. Protein was low on 
     Tuesday and Wednesday. Consider adding a protein shake on gym days."
```

### 9.3 Acceptance Criteria

| # | Criterion |
|---|-----------|
| AC-1 | Weekly meal plan generated every Sunday with 7 days of meals |
| AC-2 | Grocery list generated from meal plan, grouped by store section |
| AC-3 | Dietary restrictions respected (vegetarian, halal, keto, allergies) |
| AC-4 | Conversational logging extracts food items + calorie estimates |
| AC-5 | Weekly nutrition summary shows calories, macros, patterns |
| AC-6 | Suggestions adapt based on logged vs. planned meals |
| AC-7 | Grocery list shareable (copy to clipboard / send via email) |

---

## 10. F-100: Onboarding & Auth

**Phase:** 0 · **Story Points:** 21 total

### 10.1 Auth Flow

```
App Launch → Splash Screen → Sign Up / Log In
  ├── Email + password (Supabase Auth)
  ├── Google OAuth (Supabase Auth)
  ├── Apple Sign In (Supabase Auth)
  └── Phone + OTP (future)
```

### 10.2 Onboarding Wizard (5 Steps)

> **IMPORTANT — Apple Guideline 5.1.2(i):** Apps must explicitly disclose and obtain user permission before sharing personal data with third-party AI. Step 1 below is legally required for App Store approval. It must name the AI provider, list data types shared, and require opt-in consent.

**Step 1: AI Data Consent (REQUIRED — Apple App Store Compliance)**
```
┌──────────────────────────────────────────────────┐
│  How Your AI Assistant Works                      │
│                                                    │
│  Your assistant uses AI to help manage your        │
│  emails, calendar, tasks, and more. To do this,   │
│  some of your data is processed by:               │
│                                                    │
│  🤖 AI Provider: MiniMax (minimax.io)              │
│     Data shared: Messages you send, email          │
│     summaries, calendar events, task content       │
│                                                    │
│  🔗 Integration Provider: Composio (composio.dev)  │
│     Data shared: OAuth tokens for connected apps   │
│     (Gmail, Calendar, etc.)                        │
│                                                    │
│  🔒 Your data is:                                  │
│     • Never sold to third parties                  │
│     • Processed only to provide your AI service    │
│     • Stored in your isolated container            │
│     • Deletable at any time from Settings          │
│                                                    │
│  [View Full Privacy Policy]                        │
│                                                    │
│  ☐ I understand and consent to AI data processing  │
│                                                    │
│  [Continue]  (disabled until checkbox is checked)   │
└──────────────────────────────────────────────────┘

- Consent checkbox must be unchecked by default (opt-in, not opt-out)
- "View Full Privacy Policy" links to detailed data flow documentation
- Consent timestamp stored in users table
- Without consent, user cannot proceed — no skip option
```

**Step 2: Profile Setup**
```
- Name, timezone (auto-detected), wake time (default 7:00 AM)
- Avatar (optional, default generated)
- Language preference
```

**Step 3: Goal Setting**
```
- Select 3+ categories: Career, Fitness, Learning, Finance, Health, Relationships, Creativity, Other
- For each: one-sentence goal + target date
- Example: Career → "Get promoted to senior engineer" → June 2026
```

**Step 4: First Integration**
```
- "Connect your email to get started" → Composio Gmail/Outlook OAuth
- Optional: Connect calendar
- Skip option: "I'll do this later"
```

**Step 5: First Daily Plan**
```
- Agent generates first day's plan based on goals + connected calendar
- User previews tomorrow's schedule
- "Your AI assistant is ready! Here's your plan for tomorrow."
```

### 10.3 Acceptance Criteria

| # | Criterion |
|---|-----------|
| AC-1 | Sign up to first daily plan in <3 minutes |
| AC-2 | Timezone auto-detected from device, user can override |
| AC-3 | At least 3 goals required before proceeding |
| AC-4 | Skip integration allowed, with gentle reminder in daily brief |
| AC-5 | First agent container provisioned during onboarding (pre-warm) |
| AC-6 | Onboarding state persisted — resumable if app closes mid-flow |
| AC-7 | **AI consent screen names MiniMax as AI provider and lists data types shared (Apple Guideline 5.1.2(i))** |
| AC-8 | **Consent checkbox is unchecked by default — must be opt-in** |
| AC-9 | **Consent timestamp recorded in users table for audit trail** |
| AC-10 | **User cannot proceed past Step 1 without granting AI consent — no skip option** |

---

## 11. F-200: Core Agent Chat

**Phase:** 1 · **Story Points:** 26 total

### 11.1 Chat Interface

```
┌──────────────────────────────────────────┐
│  🤖 Alia's Assistant         [🔊] [⚙️]  │
│─────────────────────────────────────────│
│                                          │
│  💬 Agent: Good morning! Your inbox has   │
│  3 urgent emails and 12 newsletters.     │
│  Here's your digest...                   │
│                                          │
│  📧 [Inbox Digest Card]                  │
│  ├── Urgent: 3 items                     │
│  ├── Action: 8 items                     │
│  └── [View Full Digest]                  │
│                                          │
│  👤 You: Schedule a meeting with Sarah    │
│  tomorrow at 10am                        │
│                                          │
│  💬 Agent: I'll create that for you.     │
│                                          │
│  📅 [Calendar Action Card]               │
│  │ Meeting with Sarah                    │
│  │ Tomorrow, 10:00 AM - 10:30 AM        │
│  │ [✓ Create]  [✏️ Edit]  [✕ Cancel]    │
│  └───────────────────────────────────────│
│                                          │
│  ┌──────────────────────────────────┐    │
│  │ Type a message...        [🎤][📎]│    │
│  └──────────────────────────────────┘    │
└──────────────────────────────────────────┘
```

### 11.2 Message Types

| Type | Description | Visual |
|------|------------|--------|
| `text` | Plain text message | Chat bubble |
| `action_card` | Requires user approval (create event, send email) | Card with approve/reject buttons |
| `info_card` | Read-only structured data (digest, report) | Expandable card |
| `status` | Agent doing something ("Checking your calendar...") | Typing indicator + progress text |
| `error` | Something failed ("Couldn't connect to Gmail") | Red alert with retry button |
| `system` | System message ("Container starting...") | Gray italic text |

### 11.3 Real-Time Communication

> **IMPORTANT:** Supabase Edge Functions have a 2-second CPU time limit and cannot act as synchronous orchestrators. The Edge Function is a **thin auth proxy** that validates the user and fires-and-forgets to the Gateway. The Gateway pushes responses back asynchronously via Supabase Realtime.

```
Mobile App ←─ WebSocket (Supabase Realtime) ─→ Supabase
                                                    │
Supabase Edge Function ──fire-and-forget──→ Gateway ←─ HTTP ─→ Container
                                               │
                                               └──→ Supabase Realtime (push response)

Flow:
1. User sends message → HTTPS POST to Supabase Edge Function
2. Edge Function validates JWT + subscription tier (must complete in <2s CPU)
3. Edge Function fire-and-forget POSTs to Gateway → returns 202 Accepted to app
4. App subscribes to Supabase Realtime channel for this conversation
5. Gateway ensures container is running, routes request to user container
6. Container streams response chunks back to Gateway
7. Gateway pushes chunks directly to Supabase Realtime channel
8. Gateway logs completed task to Supabase tasks table
9. Mobile app renders response incrementally as chunks arrive via WebSocket
```

### 11.4 Acceptance Criteria

| # | Criterion |
|---|-----------|
| AC-1 | First response token in <3 seconds (including cold start) |
| AC-2 | Streaming response renders incrementally (not all-at-once) |
| AC-3 | Action cards block until user approves or dismisses |
| AC-4 | Chat history persists across sessions (Supabase + memory.db) |
| AC-5 | Typing indicator shown while agent processes |
| AC-6 | Offline: queue messages, sync when back online |
| AC-7 | Attach images (max 5MB) — agent can describe/analyze |
| AC-8 | Deep link: action cards can open specific app screens |

---

## 12. F-300: Container Lifecycle (Internal)

**Phase:** 0 · **Internal service — not user-facing**

### 12.1 Description

The Gateway service manages the complete lifecycle of per-user Docker containers. This is the core infrastructure that enables the per-user isolation pattern.

### 12.2 Container States

```
                    ┌───────────────┐
         create     │   CREATED     │
     ┌─────────────▶│  (not yet     │
     │              │   started)    │
     │              └───────┬───────┘
     │                      │ start
     │                      ▼
     │              ┌───────────────┐
     │              │   RUNNING     │◄─── start (resume)
     │              │  ~50MB RAM    │
     │              │  0.5 CPU      │
     │              └───────┬───────┘
     │                      │ stop (10min idle)
     │                      ▼
     │              ┌───────────────┐
     │              │   STOPPED     │
     │              │  0 RAM, 0 CPU │
     │              │  disk only    │
     │              └───────┬───────┘
     │                      │ rm (24hr idle)
     │                      ▼
     │              ┌───────────────┐
     │              │   REMOVED     │
     │              │  volume stays │
     │              │  on host disk │
     │              └───────────────┘
     │                      │
     └──────────────────────┘ next request: create again
```

### 12.3 Gateway Health Check

```
EVERY 60 SECONDS:
  - Count running containers
  - Count total containers (running + stopped)
  - Check host RAM usage
  - Check host disk usage
  - Log metrics to stdout (JSON structured logs)

  If running_containers > MAX_CONTAINERS * 0.8:
    → Alert: "Approaching container limit"

  If host_ram_pct > 85:
    → Force-stop oldest idle containers

  If host_disk_pct > 80:
    → Remove containers stopped for >12 hours
    → Alert: "Disk pressure — consider cleanup"
```

### 12.4 Gateway API

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `POST /api/chat` | POST | Route message to user's container |
| `POST /api/agent/action` | POST | Trigger specific agent action (digest, brief) |
| `GET /api/container/{user_id}/status` | GET | Check container state |
| `POST /api/container/{user_id}/restart` | POST | Force restart container |
| `GET /api/health` | GET | Gateway health + metrics |
| `GET /api/stats` | GET | Running/stopped/total container counts |

---

## 13. F-400: Scrapling Deep Research

**Phase:** 4 · **Story Points:** 26 total · **Tier:** Pro+

### 13.1 Description

Powered by the Scrapling Python sidecar. Agent performs multi-source web research with synthesis reports, fact-checking, and citation tracking. Also supports lead scraping and competitor monitoring.

### 13.2 Research Flow

```
User: "Research the top 5 AI agent frameworks in 2026 and compare them"

Agent:
  1. Plans research strategy (3-5 search queries)
  2. Calls Scrapling sidecar for each query:
     - StealthyFetcher for anti-bot sites
     - Adaptive element tracking for structured data
  3. Extracts relevant content from top 3-5 results per query
  4. LLM synthesizes into structured report:
     - Executive summary
     - Comparison table
     - Pros/cons per framework
     - Sources with links
  5. Saves report to workspace/research/
  6. Displays in Research Panel UI

Total time: 30-120 seconds depending on complexity
```

### 13.3 Scrapling Sidecar API

```json
POST http://scrapling:8001/fetch
{
  "url": "https://example.com/article",
  "mode": "stealthy",        // stealthy | dynamic | basic
  "extract": {
    "title": "h1",
    "body": "article",
    "date": "time[datetime]"
  },
  "timeout": 30
}

Response:
{
  "status": "success",
  "data": {
    "title": "Top AI Agent Frameworks...",
    "body": "In 2026, the landscape...",
    "date": "2026-02-15"
  },
  "metadata": {
    "url": "https://...",
    "fetched_at": "2026-02-28T12:00:00Z",
    "content_length": 4521
  }
}
```

### 13.4 Rate Limits by Tier

| Tier | Research/Day | Pages/Research | Max Concurrent |
|------|-------------|---------------|---------------|
| Starter | 5 | 10 | 1 |
| Pro | 25 | 25 | 3 |
| Power | 100 | 50 | 5 |

---

## 14. F-500: Social Media & Content

**Phase:** 4 · **Story Points:** 29 total · **Tier:** Pro+

### 14.1 Features

| Feature | Description |
|---------|------------|
| **Post Drafting** | AI generates platform-specific content (X: 280 char, LinkedIn: professional, Instagram: visual) |
| **Multi-Platform Posting** | Single draft → publish to X, LinkedIn, Instagram via Composio |
| **Content Calendar** | Visual weekly/monthly view of scheduled posts |
| **Hashtag Suggestions** | LLM suggests relevant hashtags based on content and trending |
| **Analytics** | Basic engagement metrics from connected platforms |
| **Video Upload** | Upload pre-made videos to YouTube/TikTok |

### 14.2 Post Creation Flow

```
User: "Write a LinkedIn post about our new product launch"

Agent:
  1. Asks clarifying questions (product name, key features, tone)
  2. Generates 2-3 draft variations
  3. Presents as action cards:
     ┌─────────────────────────────────────┐
     │ 📝 LinkedIn Post Draft              │
     │                                      │
     │ Option A (Professional):             │
     │ "Excited to announce..."             │
     │                                      │
     │ Option B (Story-driven):             │
     │ "Two years ago, we had a problem..." │
     │                                      │
     │ [Post A] [Post B] [Edit] [Schedule]  │
     └─────────────────────────────────────┘
  4. User selects → Composio posts to LinkedIn
  5. Saved to content calendar
```

---

## 15. F-600: Integrations Hub & Skills Marketplace

**Phase:** 3 · **Story Points:** 37 total

### 15.1 Integrations Hub

```
┌─────────────────────────────────────────────┐
│  🔗 Connected Apps (4/10)                    │
│                                               │
│  ✅ Gmail          ✅ Google Calendar          │
│  ✅ Notion         ✅ Telegram                 │
│                                               │
│  ── Available (250+) ──────────────────────  │
│                                               │
│  [Slack] [Trello] [Jira] [X/Twitter]         │
│  [LinkedIn] [GitHub] [Stripe] [Zapier]       │
│  [YouTube] [Discord] [WhatsApp] [Figma]      │
│  ...                                          │
│                                               │
│  Search: [                          🔍]       │
└─────────────────────────────────────────────┘
```

Each integration connects via Composio OAuth. The agent gains new tools automatically when an app is connected.

### 15.2 Skills Marketplace

```
┌─────────────────────────────────────────────┐
│  🧩 Skills Marketplace                       │
│                                               │
│  Featured:                                    │
│  ┌──────────────┐ ┌──────────────┐           │
│  │ 📧 Inbox     │ │ 📰 News      │           │
│  │ Digest Pro   │ │ Curator      │           │
│  │ ⭐ 4.8 (234) │ │ ⭐ 4.6 (189) │           │
│  │ [Installed✓] │ │ [Install]    │           │
│  └──────────────┘ └──────────────┘           │
│                                               │
│  Categories:                                  │
│  [Productivity] [Health] [Finance]            │
│  [Social] [Research] [Creative]               │
│                                               │
│  Premium Skills: 🔒 (Pro tier required)       │
│  [Meal Planner] [SEO Research] [Lead Gen]    │
└─────────────────────────────────────────────┘
```

Skills are TOML manifests + SKILL.md instruction files that install into the user's `/agent/skills/` directory. The agent loads them at startup.

---

## 16. F-700: Payments & Subscriptions

**Phase:** 5 · **Story Points:** 15 total

### 16.1 RevenueCat Integration

```
Mobile App (react-native-purchases)
  │
  ├── iOS: App Store subscriptions
  ├── Android: Google Play subscriptions
  └── Web: Stripe via RevenueCat Web Billing

  RevenueCat webhooks → Supabase Edge Function
    → Updates subscriptions table
    → Supabase Realtime notifies app of tier change
    → Gateway applies new container resource limits
```

### 16.2 Paywall Screen

```
┌─────────────────────────────────────────────┐
│  Choose Your Plan                            │
│                                               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐     │
│  │ Starter │  │  Pro ✨  │  │ Power   │     │
│  │ $30/mo  │  │ $50/mo  │  │ $100/mo │     │
│  │         │  │         │  │         │     │
│  │ 10 apps │  │ 25 apps │  │ Unlim.  │     │
│  │ 5 rsrch │  │ 25 rsrch│  │ 100/day │     │
│  │ App only│  │+Telegram│  │All chan. │     │
│  │ No voice│  │ Voice   │  │ Custom  │     │
│  │         │  │         │  │ voice   │     │
│  │[Select] │  │[Select] │  │[Select] │     │
│  └─────────┘  └─────────┘  └─────────┘     │
│                                               │
│  🎁 7-day free trial on Pro                   │
│  Cancel anytime. Billed monthly.              │
└─────────────────────────────────────────────┘
```

### 16.3 Acceptance Criteria

| # | Criterion |
|---|-----------|
| AC-1 | Purchase flow completes in <30 seconds on iOS and Android |
| AC-2 | Subscription status synced to Supabase within 5 seconds of purchase |
| AC-3 | Feature gating immediate: upgrade unlocks Telegram within seconds |
| AC-4 | Downgrade: features locked at period end, not immediately |
| AC-5 | Free trial: 7 days Pro, no charge unless user keeps plan |
| AC-6 | Restore purchases works across devices |
| AC-7 | RevenueCat dashboard shows MRR, churn, trial conversions |

---

## 17. Non-Functional Requirements

### 17.1 Performance

| Metric | Target |
|--------|--------|
| Chat response (first token) | <3 seconds (including cold start) |
| Chat response (warm container) | <1.5 seconds |
| Container cold start | <3 seconds |
| Container warm restart | <1 second |
| Push notification delivery | <5 seconds from trigger |
| App launch to interactive | <2 seconds |
| API response (p99) | <5 seconds |
| Voice end-to-end latency | <3 seconds (STT + LLM + TTS). Stretch goal: <2 seconds. |
| Edge Function execution | <2 seconds CPU (hard limit — auth + fire-and-forget only) |

### 17.2 Reliability

| Metric | Target |
|--------|--------|
| Gateway uptime | 99.5% (allows ~3.6 hours/month downtime) |
| Supabase uptime | 99.9% (managed by Supabase) |
| Data durability | No user data loss. Backups every 6 hours. |
| Recovery time | <15 minutes from full server failure |
| Mean time to detect | <5 minutes for gateway failure |

### 17.3 Scalability

| Phase | Users | Concurrent | Infrastructure |
|-------|-------|-----------|---------------|
| Launch | 0–300 | ~45 | 1× Hetzner CX32 (8GB, €16/mo) |
| Growth | 300–1,000 | ~150 | 1× Hetzner CX42 (16GB, €30/mo) |
| Scale | 1K–5K | ~750 | 2× CX42 + LB (€66/mo) |
| Expand | 5K–10K | ~1,500 | 4× CX42 + LB (€126/mo) |
| Big | 10K+ | 1,500+ | k3s cluster (€200+/mo) |

### 17.4 Security

| Requirement | Implementation |
|------------|---------------|
| Authentication | Supabase Auth (JWT, bcrypt, email/social) |
| Authorization | Supabase RLS + Edge Function tier checks |
| Data isolation | Per-user Docker containers with separate filesystems |
| Encryption at rest | ZeroClaw encrypted secrets, Supabase encrypted storage |
| Encryption in transit | HTTPS everywhere (Caddy TLS termination) |
| Container hardening | Non-root, read-only FS, dropped capabilities, PID limit |
| Socket security | Docker socket proxy (restricted API surface) |
| Credential management | Composio SOC 2 (AI never sees raw OAuth tokens) |
| Rate limiting | Per-user, per-endpoint limits at Edge Function level |
| Audit logging | All agent actions logged to tasks table with timestamps |

### 17.5 Compliance

| Area | Approach |
|------|---------|
| **Apple App Store (Guideline 5.1.2(i))** | **Mandatory AI consent screen in onboarding (Step 1).** Must name MiniMax as AI provider, list data types shared, require opt-in checkbox. App Store Review Notes must describe AI integration, data flow, and consent mechanism. Build with iOS 26 SDK before April 28, 2026. Supervised mode as default. Content moderation. |
| **GDPR** | User data deletion on request. Export via settings. **Data Processing Agreements (DPAs) required with:** MiniMax (China), Composio (USA), Deepgram (USA), ElevenLabs (USA/Poland). **Standard Contractual Clauses (SCCs)** for cross-border transfers to non-EU countries. Data flow diagram documenting which data goes to which processor. Privacy policy must list all third-party processors per GDPR Articles 13-14. |
| **GDPR — Data Residency** | Hetzner EU datacenter (Falkenstein/Nuremberg) for user containers and volumes. Note: LLM inference (MiniMax) and integrations (Composio) process data outside the EU. Users must be informed of this in the AI consent screen. |
| Email (CAN-SPAM) | Only send emails user explicitly approves |
| **Google Play** | Similar AI disclosure requirements. Privacy policy must be linked in store listing and accessible in-app. |

### 17.6 Monitoring & Alerting

| What | How | Alert |
|------|-----|-------|
| Container count | Gateway metrics endpoint | >80% of max |
| Host RAM | Node exporter | >85% |
| Host disk | Node exporter | >80% |
| Cold start latency | Gateway structured logs | >5 seconds |
| Failed requests | Gateway error rate | >5% in 5-minute window |
| Supabase health | Supabase dashboard | Automatic notifications |
| SSL certificate | Caddy auto-renewal | 7 days before expiry |

---

*— End of Feature Specification Document —*
