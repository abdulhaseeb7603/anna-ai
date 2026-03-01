# ZeroClaw AI — Product Requirements Document v3.0

> **Personal AI Agent Platform**
> Your 24/7 AI Assistant — Affordable, Autonomous, For Everyone
>
> February 28, 2026 · Version 3.0

---

## 1. Executive Summary

We are building an affordable, cloud-hosted **Personal AI Agent Platform** as a mobile app. Each user gets a fully isolated AI agent running in its own Docker container, powered by **ZeroClaw** (Rust, <5MB RAM), connected to **250+ apps** via Composio, and backed by **MiniMax M2.5** as the primary LLM.

### Three Core Pillars

1. **Autonomous AI Agent:** A 24/7 assistant handling emails, calendars, tasks — in supervised or full autonomy mode.
2. **Deep Research & Content Engine:** Web research via Scrapling, social media posting, lead generation, competitor monitoring.
3. **Personal AI Secretary:** Daily routines, smart reminders, fitness/diet planning, meeting prep, personalized daily briefs.

### Key Differentiators

| | ZeroClaw AI | OpenClaw (self-hosted) | QuickClaw ($200/mo) |
|---|---|---|---|
| **Setup** | Download app, sign up | SSH, CLI, config files | Enterprise onboarding |
| **Price** | $30/mo | Free (+ $5-20/mo VPS) | $200+/mo |
| **Isolation** | Per-user Docker container | Single instance | Shared cloud |
| **Integrations** | 250+ (Composio) | Manual setup per tool | Limited selection |
| **Target User** | Everyone | Developers only | Enterprises |

**Cost Advantage:** ZeroClaw runs on <5MB RAM per user. MiniMax M2.5 costs ~$1-3/user/month. Infrastructure: ~€0.05-0.15/user/month on Hetzner. **Total: ~$3-7/user/month (average) = 77-90% gross margins** at $30/mo. Heavy users (P90) may cost $15-25/mo — managed via model routing, token budgets, and per-user cost tracking. See Section 2 for full market validation.

---

## 2. Idea Validation & Market Research

> Research conducted March 1, 2026. All data sourced from public market reports, analyst firms, and community metrics.

### 2.1 Market Size & Growth

| Metric | Value | Source |
|--------|-------|--------|
| **Global AI Market (2026)** | $375.9B (26.6% CAGR to $2.48T by 2034) | [Fortune Business Insights](https://www.fortunebusinessinsights.com/industry-reports/artificial-intelligence-market-100114) |
| **AI Assistant Market (2025→2030)** | $3.35B → $21.1B (44.5% CAGR) | [MarketsandMarkets](https://www.marketsandmarkets.com/Market-Reports/ai-assistant-market-40111511.html) |
| **Agentic AI Market (2025→2030)** | $8.03B → $45B | [Deloitte TMT Predictions](https://www.oreateai.com/blog/the-agentic-ai-surge-charting-a-course-to-a-45-billion-market-by-2030/b4801f3e172e30ecd30ffcca031057c9) |
| **Consumer Gen AI App Spending (2026)** | >$10B (projected top-5 mobile category by downloads) | [Visual Capitalist / Sensor Tower](https://www.visualcapitalist.com/charted-the-explosive-growth-of-gen-ai-apps/) |
| **Worldwide AI Spending (2026)** | $2.52T (44% YoY increase) | [Gartner](https://www.gartner.com/en/newsroom/press-releases/2026-1-15-gartner-says-worldwide-ai-spending-will-total-2-point-5-trillion-dollars-in-2026) |

### 2.2 Demand Validation: OpenClaw Proves the Thesis

[OpenClaw](https://en.wikipedia.org/wiki/OpenClaw) (formerly Clawdbot) is the open-source AI agent that went viral in late 2025. Its growth validates that there is massive, unmet demand for a personal AI agent that "actually does things."

| OpenClaw Metric | Value | Timeframe |
|-----------------|-------|-----------|
| GitHub Stars | 200,000+ | <3 months (Nov 2025 – Feb 2026) |
| Agents Created | 1.5 million | By Feb 2026 |
| GitHub Forks | 20,000+ | By Feb 2026 |
| X Followers | 277,000 | 10 weeks from first commit |
| Website Traffic | 2 million visitors | Single week peak |

**Why this matters for us:** OpenClaw proved the use case but requires CLI setup, SSH access, and server management. Our product takes the same core value — a personal AI agent that manages emails, calendar, tasks, and research — and delivers it through a mobile app. The gap between "1.5 million agents created by developers" and "0 agents used by non-technical professionals" is our opportunity.

**Key development (Feb 15, 2026):** OpenClaw's creator Peter Steinberger joined OpenAI to lead "next-generation personal agents." This signals that OpenAI will build a competitor — compressing our launch window to **6-12 months** before a well-funded alternative arrives.

Sources: [CNBC](https://www.cnbc.com/2026/02/02/openclaw-open-source-ai-agent-rise-controversy-clawdbot-moltbot-moltbook.html) · [OpenAI Acquisition (Leanware)](https://www.leanware.co/insights/openai-openclaw-acquisition) · [OpenClaw Growth Analysis (LearnDevRel)](https://learndevrel.com/blog/openclaw-ai-agent-phenomenon)

### 2.3 Consumer Willingness to Pay

| Data Point | Value | Source |
|------------|-------|--------|
| Consumers who would use AI as personal assistant | 44% of U.S. adults (70% among Gen Z) | [Index.dev AI Statistics](https://www.index.dev/blog/ai-assistant-statistics) |
| Current AI users globally | ~1.8 billion | [Menlo Ventures](https://menlovc.com/perspective/2025-the-state-of-consumer-ai/) |
| Users who pay for premium AI | ~3% (massive monetization gap) | [Menlo Ventures](https://menlovc.com/perspective/2025-the-state-of-consumer-ai/) |
| Theoretical consumer AI TAM | ~$432B/year (1.8B users × $20/mo) | Derived from Menlo Ventures data |
| AI apps revenue (2024) | $4.5B (136% YoY growth) | [Sensor Tower](https://www.marketingdive.com/news/sensor-tower-state-of-mobile-report-generative-ai/737962/) |
| Non-tech companies using/planning AI agents | 90% | [Master of Code](https://masterofcode.com/blog/ai-agent-statistics) |

**Interpretation:** Only 3% of 1.8 billion AI users currently pay — a $420B+ gap between actual and potential spending. The barrier is not willingness but perceived value. Users will pay when the AI does real, recurring work (not just answers questions). Our email digest + calendar management + daily planning creates daily recurring value that justifies subscription pricing.

### 2.4 Retention Benchmarks (The Hard Truth)

Consumer AI app retention is the biggest risk. Here are the benchmarks we must beat:

| Platform | 6-Month Retention | 12-Month Retention | Source |
|----------|------------------|-------------------|--------|
| ChatGPT Plus | 71% | 68% | [Arcade Research](https://www.arcade.dev/blog/user-retention-in-ai-platforms-metrics) |
| Claude Pro | 62% | — | [Arcade Research](https://www.arcade.dev/blog/user-retention-in-ai-platforms-metrics) |
| Gemini Advanced | 60% | 57% | [Arcade Research](https://www.arcade.dev/blog/user-retention-in-ai-platforms-metrics) |
| Perplexity Pro | 49% | — | [Arcade Research](https://www.arcade.dev/blog/user-retention-in-ai-platforms-metrics) |
| Average mobile app (day 30) | — | — | 38% (subscription apps) |
| Productivity apps (day 30) | — | — | 4.1% |

**Our target:** 50% 6-month retention puts us between Perplexity and Gemini — achievable if the daily brief + email digest creates a morning habit. Below 40% at 6 months signals a product problem.

**What drives retention in AI apps:**
- Products that automate recurring work (billing, scheduling, email) keep users. Products that only satisfy curiosity don't. ([Growth Unhinged](https://www.growthunhinged.com/p/the-ai-churn-wave))
- Personalized onboarding reduces churn by 22%. ([RevenueCat State of Subscription Apps](https://www.revenuecat.com/state-of-subscription-apps-2025/))
- Only 9% of consumers pay for more than one AI subscription — once they pick yours, switching is rare. But they need to pick yours first.

### 2.5 Competitive Landscape

| Competitor | Price | Target | Strength | Our Advantage |
|-----------|-------|--------|----------|---------------|
| **ChatGPT Plus** | $20/mo | Everyone | Brand, distribution, 273M MAU | We do actions (email, calendar, tasks), ChatGPT just chats. No integrations, no proactive daily brief, no autonomous agent mode. |
| **OpenClaw (self-hosted)** | Free + $5-20 VPS | Developers | Open source, customizable, 200K stars | We remove the technical barrier. Mobile app vs. CLI. No SSH, no server management. |
| **Lindy AI** | Free / Pro | Business users | Pre-built AI employees, SOC 2, HIPAA | We're mobile-first for consumers. Lindy is web-only, workflow-oriented. |
| **Relevance AI** | Free / $19-$599/mo | Business ops teams | Multi-agent orchestration, research focus | We're consumer-focused, simpler UX, daily life management vs. business ops. |
| **QuickClaw** | $200+/mo | Enterprise | Enterprise features, compliance | 6x cheaper, consumer-first UX. |
| **Microsoft Copilot** | ~$27-30/mo (bundled) | Microsoft 365 users | Distribution, bundled with Office | We're platform-agnostic, deeper agent capabilities, works with any email/calendar. |
| **Zapier / n8n** | $20-50/mo | Automation builders | 8,000+ integrations | We don't require building workflows. The agent figures it out from natural language. |

**Our positioning:** The only mobile-first, consumer-priced ($30/mo), fully-autonomous AI agent with per-user container isolation. We sit in the gap between "ChatGPT (chats but doesn't act)" and "OpenClaw (acts but requires dev skills)."

### 2.6 Why Startups in This Space Fail

Research from 2025-2026 shows 90% of AI startups fail within year one. The common failure patterns and how we address them:

| Failure Pattern | How We Mitigate | Source |
|----------------|----------------|--------|
| **Novelty over retention** — users try it, then churn | Daily brief + email digest create morning habit loop. Daily tasks create evening completion loop. Two touchpoints/day = recurring value. | [Growth Unhinged](https://www.growthunhinged.com/p/the-ai-churn-wave) |
| **Runaway inference costs** — margins collapse at scale | Per-user cost tracking from day one. Model routing (cheap models for simple tasks). Fair use token budgets. Usage alerts. | [Clarifai](https://www.clarifai.com/blog/reasons-why-ai-native-startups-fail) |
| **Thin LLM wrapper** — platform swallows you | We're not a wrapper. Per-user Docker isolation, Composio integrations, autonomous scheduling, and proactive push notifications are infrastructure-level differentiation. | [Medium](https://medium.com/write-a-catalyst/the-great-ai-collapse-of-2026-why-most-startups-are-failing-and-how-to-build-an-unbreakable-moat-94b81d57df72) |
| **"Big bang" launch** — too many features, none polished | MVP is 5 features in 8 weeks. Email + Calendar + Daily Brief + Tasks + Memory. Deliberately narrow. | [HBR](https://hbr.org/2025/11/most-ai-initiatives-fail-this-5-part-framework-can-help) |
| **No human oversight** — users don't trust autonomous AI | Supervised mode is default. Every agent action requires user approval until they explicitly enable autonomy per category. | [Directual](https://www.directual.com/blog/ai-agents-in-2025-why-95-of-corporate-projects-fail) |

### 2.7 Validation Verdict

| Question | Answer | Confidence |
|----------|--------|------------|
| Is there market demand? | **Yes** — OpenClaw's 1.5M agents + 44% consumer interest + $21B TAM by 2030 | High |
| Is the timing right? | **Yes** — 2026 is the inflection year for consumer AI agents. Gen AI is projected top-5 mobile category. | High |
| Can we differentiate? | **Yes** — Mobile-first + per-user isolation + consumer pricing. No competitor occupies this exact position. | Medium-High |
| Can we retain users? | **Probable** — Daily recurring features (email, calendar, brief) create habit loops. But unproven until real usage data. | Medium |
| Are unit economics viable? | **Fragile** — Average user margins are strong (77-90%), but heavy users and Composio costs need active management. | Medium |
| Can we ship fast enough? | **Critical** — OpenAI hiring OpenClaw's creator compresses the window to 6-12 months. MVP in 8 weeks is aggressive but necessary. | Medium |

**Bottom line:** The idea is validated by market data and demand signals. The risk is not "will people want this?" — they demonstrably do. The risk is execution speed (ship before OpenAI) and unit economics (manage costs before they eat margins). Build it, but launch fast and track per-user costs from day one.

---

## 3. Problem Statement

Current AI assistants are either too expensive ($100-200+/mo), too technical (require server setup), or too limited (chat-only with no actions). Only **0.03% of people** use AI in advanced, agentic ways.

**Problems We Solve:**

- **Cost barrier:** Comparable functionality at $30/mo instead of $200/mo.
- **Technical barrier:** Mobile app removes CLI, SSH, config files entirely.
- **Capability gap:** Our agent can research, scrape data, post content, manage schedules, send emails, automate workflows — not just chat.
- **Integration fragmentation:** One agent connects all your apps through Composio's 250+ integrations.
- **Privacy concern:** Per-user Docker containers mean complete data isolation. One compromised container cannot access another user's data.

---

## 4. Target Market

**Primary:** Non-technical professionals (25-45) who want AI to handle their daily admin but don't know how to set up tools like OpenClaw. Freelancers, small business owners, content creators, busy parents. Research shows 44% of U.S. consumers would use an AI personal assistant; 70% among Gen Z.

**Secondary:** Power users and developers who want an affordable managed alternative to self-hosting OpenClaw.

**TAM:** ~500M knowledge workers globally who use email + calendar daily. The AI assistant market is projected at $21.1B by 2030 ([MarketsandMarkets](https://www.marketsandmarkets.com/Market-Reports/ai-assistant-market-40111511.html)).
**SAM:** ~50M who already pay for productivity tools ($10-50/mo range). Only 3% of 1.8B AI users currently pay for premium AI — massive conversion headroom ([Menlo Ventures](https://menlovc.com/perspective/2025-the-state-of-consumer-ai/)).
**SOM (Year 1):** 10,000 paying subscribers = $300K-500K ARR. Benchmark: winning formula for 2026 is "1,000-10,000 initial users within 6 months" with "CAC $1-2, LTV $100+."

---

## 5. MVP Priority Features (Ranked)

Based on the most popular community-validated OpenClaw use cases ([awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases)) and real-world adoption patterns:

| Rank | Feature | Audience | Phase | Why This Rank |
|------|---------|----------|-------|---------------|
| **1** | **Email & Inbox Management** | Everyone with email | 1 | Most universal daily pain point. Newsletter declutter, inbox triage, draft replies. |
| **2** | **Calendar & Scheduling** | Everyone with a schedule | 1 | Daily life management. Conflict detection, auto-scheduling, meeting prep notes. |
| **3** | **Daily Brief & Routine Planning** | Mass audience | 2 | Most popular OpenClaw setup. Morning briefing: weather, calendar, tasks, news. Zero-config. |
| **4** | **Personal Task Assistant** | All users | 2 | Goal-driven daily task generation. Breaks big goals into daily steps. Streak tracking. |
| **5** | **Multi-Channel (Text + Voice)** | Broad audience | 3 | Access anywhere: app, Telegram, WhatsApp, voice. Unified context across channels. |
| **6** | **Second Brain (Memory Recall)** | Frequent AI users | 1 | Built into ZeroClaw. "What did I say about X?" Grows over time. |
| **7** | **News / Digest Summaries** | Many users | 2 | Configurable: Reddit, YouTube, RSS, Hacker News. Scrapling + LLM synthesis. |
| **8** | **Meal / Health Planning** | General population | 4 | Premium skill. Meal plans, food logging, grocery lists, pattern detection. |

**MVP Launch (Week 8):** Features 1-4 + 6 = Email, calendar, daily brief, task assistant, and memory. This alone is a compelling $30/mo product.

---

## 6. Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Mobile App** | Expo SDK 54 + React Native + NativeWind v4.2.0+ (Tailwind CSS 3.4.17) | Cross-platform iOS/Android/Web. New Architecture enabled. |
| **Boilerplate** | [expo-supabase-ai-template](https://github.com/abdulhaseeb7603/expo-supabase-ai-template) | Auth, routing, AI chat UI pre-built |
| **Backend** | Supabase (PostgreSQL + Auth + Edge Functions + Realtime + Storage) | User management, database, serverless functions. Edge Functions used as **thin auth proxy only** (2s CPU limit). |
| **Agent Runtime** | [ZeroClaw](https://github.com/zeroclaw-labs/zeroclaw) (Rust, <5MB RAM, <10ms startup) | Per-user AI agent in Docker container. **Note:** Pre-release (Feb 2026). Must benchmark independently before committing. See fallback plan in Section 13. |
| **Deployment** | Per-user Docker containers on Hetzner VPS | Isolated agent per user, Gateway manages lifecycle |
| **Web Scraping** | [Scrapling](https://github.com/D4Vinci/Scrapling) (Python, adaptive, anti-bot) | Deep research on user-initiated URLs only. Official APIs preferred for HN/Reddit/YouTube/RSS. |
| **LLM** | MiniMax M2.5 (primary) + Gemini 2.5 Flash (fallback for reasoning tasks) | MiniMax: $0.20-0.30/M input, $1.00-1.20/M output. Strong at tool-use, weaker at general reasoning (SimpleQA 44%). Model routing selects best model per task type. |
| **Integrations** | Composio (250+ OAuth apps, SOC 2) + Direct OAuth for top 3 apps | Gmail, Slack, Calendar via abstraction layer. Composio Starter: $99/mo for 1K users, 100K API calls. Apply for startup program ($25K credits). |
| **Payments** | RevenueCat (react-native-purchases v9+) | iOS/Android/Web subscriptions |
| **Push** | Expo Notifications + Supabase Edge Functions | Proactive nudges, reminders |
| **Voice** | Deepgram (STT) + ElevenLabs (TTS) | Voice commands and responses. Realistic latency: <3s (not <2s). |
| **Reverse Proxy** | Caddy | TLS termination, HTTPS |
| **Container Security** | Tecnativa docker-socket-proxy | Restricts Docker API surface. Phase 2: evaluate gVisor for kernel-level sandboxing. |

---

## 7. System Architecture

See **architecture.md** for the detailed system architecture document. Key points:

- **Per-user Docker container isolation** — each user gets their own container with separate filesystem, memory, network namespace
- **Gateway pattern** (Docker-out-of-Docker) — a persistent Gateway container manages user container lifecycle via Docker API
- **Async request flow** — Edge Functions (2s CPU limit) act as thin auth proxies; Gateway handles all orchestration and pushes responses via Supabase Realtime
- **Bind-mounted user volumes** — all user data (config, memory, skills, workspace) lives on host disk, persists across container restarts
- **On-demand lifecycle** — containers start when needed (~2-3s cold, ~1s warm), stop after 10min idle, use 0 CPU/RAM when stopped
- **Docker Socket Proxy** — restricts Gateway to only container create/start/stop operations
- **Internal Docker network** — no user container is exposed to the internet
- **Scaling path** — Phase 1: Docker Compose (single VPS), Phase 2: k3s (multi-node with HPA autoscaling)

---

## 8. Database Schema (Supabase PostgreSQL)

| Table | Key Columns | Purpose |
|-------|------------|---------|
| `users` | id, email, full_name, avatar_url, timezone, wake_time, onboarding_complete, created_at | Core user profiles |
| `subscriptions` | id, user_id, plan_tier, rc_customer_id, status, current_period_end | RevenueCat subscription sync |
| `agents` | id, user_id, container_name, node_endpoint, status, autonomy_level, config_json | Per-user agent state + container mapping |
| `integrations` | id, user_id, composio_entity_id, app_name, status, connected_at | Connected third-party apps |
| `schedules` | id, user_id, title, cron_expression, timezone, action_type, payload_json, active | Cron jobs (synced to user volume) |
| `tasks` | id, user_id, agent_id, type, status, input, output, created_at, completed_at | Agent task queue and history |
| `conversations` | id, user_id, agent_id, messages_json, created_at | Chat history (also in ZeroClaw memory.db) |
| `goals` | id, user_id, category, title, description, target_date, progress, streak_count | User goals from onboarding |
| `skills` | id, name, description, category, manifest_toml, is_premium, installed_count | Skill marketplace catalog |
| `user_skills` | user_id, skill_id, installed_at, config_json | Skills installed per user |

All tables use **Row Level Security (RLS)** — users can only read/write their own rows.

---

## 9. Pricing Strategy

| | Starter ($30/mo) | Pro ($50/mo) | Power ($100/mo) | Enterprise (Custom) |
|---|---|---|---|---|
| **Agent Mode** | Supervised only | Supervised + selective Full | Full autonomy | Full + custom policies |
| **Integrations** | 10 apps | 25 apps | Unlimited | Unlimited + custom |
| **Scrapling** | 5/day | 25/day | 100/day | Unlimited |
| **Social Posting** | 5/week | 20/week | Unlimited | Unlimited + team |
| **Skills** | Free only | All | All + priority | Custom dev |
| **Channels** | App only | App + Telegram | All channels | All + Slack + custom |
| **Voice** | No | Basic | Advanced + custom voice | Custom |
| **Container Resources** | 50MB / 0.5 CPU | 100MB / 1.0 CPU | 200MB / 2.0 CPU | Custom |
| **LLM** | MiniMax M2.5 | MiniMax M2.5 | MiniMax + Claude/GPT-4o | Custom routing |

**Unit Economics (Revised — March 2026):**

| Cost Component | Per User/Month (Average) | Per User/Month (Heavy — P90) | Notes |
|---------------|------------------------|-----------------------------|-------|
| Hetzner compute (shared) | ~€0.05-0.15 | ~€0.20 | Prices increased up to 37% in April 2026. Verify current rates at hetzner.com/cloud. |
| MiniMax M2.5 tokens | ~$1.00-3.00 | ~$8.00-15.00 | Heavy users (50+ tasks/day) consume significantly more. Model routing to cheaper models for simple tasks reduces this. |
| Composio platform + API calls | ~$1.00-3.00 | ~$5.00+ | Starter plan: $99/mo for 1K users, 100K API calls. Growth: $199/mo for 5K users, 500K calls. Enterprise: custom. Each inbox digest = hundreds of API calls. **Must verify enterprise pricing before launch.** |
| Supabase (shared) | ~$0.05-0.10 | ~$0.15 | Pro plan at $25/mo shared across users |
| Push notifications | ~$0.01 | ~$0.03 | Minimal cost |
| Voice (Pro+ only) | $0.00 (Starter) | ~$2.00-5.00 | Deepgram STT: $0.0043/min. ElevenLabs TTS: $0.050/1K chars. |
| **Total (Average)** | **~$3-7/user/month** | **~$15-25/user/month** | |

At $30/mo Starter (average user) = **~77-90% gross margin**. At $30/mo Starter (heavy user) = **~17-50% gross margin**.

**Margin Protection Measures:**
- **Per-user cost tracking** from day one — monitor actual costs per user in real-time
- **Fair use policy** with soft monthly token budgets per tier (e.g., Starter: 500K tokens/mo, Pro: 2M tokens/mo)
- **Model routing** — use cheaper models (Gemini 2.5 Flash) for simple tasks, MiniMax M2.5 for complex agentic tasks
- **Composio abstraction layer** — top 3 integrations (Gmail, Google Calendar, Slack) can fall back to direct OAuth if Composio costs exceed budget
- **Usage alerts** — notify team when any user's monthly cost exceeds 50% of their subscription price

---

## 10. Security Model

| Layer | Mechanism | Protects Against |
|-------|----------|-----------------|
| Supabase Auth | JWT tokens, email/social login | Unauthorized access |
| Supabase RLS | Row-level security policies | Cross-user data access |
| Edge Function | Subscription + rate limit checks | Abuse, free-tier bypass |
| Gateway Auth | Bearer token (Edge Fn → Gateway) | Direct Gateway access |
| Docker Socket Proxy | Restricted API surface | Gateway privilege escalation |
| Container Isolation | Separate process/FS/network per user | Cross-user contamination |
| Container Hardening | Non-root, read-only FS, dropped caps | Container breakout |
| Encrypted Secrets | ZeroClaw secrets.enc per user | Credential theft |
| Composio SOC 2 | Managed OAuth, AI never sees tokens | Token exposure to LLM |
| Supervised Mode | User approval for all agent actions (default) | Unwanted autonomous actions |

---

## 11. Mobile App Screens

| # | Screen | Description |
|---|--------|------------|
| 1 | **Onboarding** | 4-step wizard: profile → goals → connect Gmail/Calendar → first daily plan |
| 2 | **Home / Dashboard** | Today's schedule + agent status + quick actions + streak counter |
| 3 | **Chat** | Primary AI conversation with action cards and approval dialogs |
| 4 | **Daily Planner** | Full-day timeline with time blocks, tasks, drag-to-reorder |
| 5 | **Integrations Hub** | Browse/connect 250+ apps via Composio OAuth flow |
| 6 | **Skills Marketplace** | Browse/install agent skills with ratings and categories |
| 7 | **Research Panel** | Deep research reports, scraped data, source citations |
| 8 | **Social Media Manager** | Content calendar, scheduled posts, multi-platform |
| 9 | **Settings** | Account, subscription (RevenueCat paywall), agent config, notifications |
| 10 | **Voice Mode** | Full-screen voice interaction with waveform visualizer |

---

## 12. User Stories

### Epic 0: MVP Priority Features

| ID | Story | Points |
|----|-------|--------|
| US-001 | **Inbox Digest & Email Triage:** Agent reads inbox daily, categorizes (urgent/FYI/promo/spam), summarizes newsletters, drafts replies. Composio Gmail connected. Daily digest at user's time. Supervised mode for outbound. | 8 |
| US-002 | **Calendar Sync & Scheduling:** Sync Google Calendar/Outlook via Composio. Detect conflicts, suggest slots, auto-schedule recurring routines. 15min prep notification before meetings. | 8 |
| US-003 | **Morning Daily Brief:** Personalized morning briefing at wake time — weather, calendar, top 3 tasks, headlines, goal progress. Push notification + in-app timeline. Under 150 words. | 5 |
| US-004 | **Goal-Driven Daily Tasks:** Generate 4-5 daily tasks from user's goals. Push reminders. Streak counter. Adaptive difficulty. Integrates with Notion/Trello/Jira via Composio. | 8 |
| US-005 | **Multi-Channel Sync:** Chat on app, continue on Telegram, use voice — all shared context. Channel access gated by tier. | 13 |
| US-006 | **Second Brain / Memory:** "What did I say about X?" SQLite hybrid search (vector + keyword). Auto-saves context. Explicit bookmarking. | 5 |
| US-007 | **News & Content Digest:** Configure sources (RSS, Reddit, YouTube, HN). Scrapling fetches. LLM summarizes and ranks by relevance. Evening or morning delivery. | 8 |
| US-008 | **Meal & Health Planning:** Dietary preferences, weekly meal plans, grocery lists, conversational food logging, pattern detection. Premium skill. | 13 |

### Epic 1: Onboarding & Auth

| ID | Story | Points |
|----|-------|--------|
| US-101 | **User Registration:** Email/social login via Supabase Auth. Profile creation. | 5 |
| US-102 | **Goal Setting:** Select 3+ goal categories during onboarding. AI generates first daily plan. | 8 |
| US-103 | **First Integration:** Connect Gmail or Calendar during onboarding via Composio OAuth. | 8 |

### Epic 2: Core Agent Interaction

| ID | Story | Points |
|----|-------|--------|
| US-201 | **Chat with Agent:** Real-time messaging. <3s response. History persisted. | 5 |
| US-202 | **Supervised Approval:** Action cards with approve/reject. Agent executes only after approval. | 8 |
| US-203 | **Autonomous Execution:** Per-category autonomy toggle. Action history. Revocable. | 13 |
| US-204 | **Voice Interaction:** STT/TTS, hands-free mode. Wake button activation. | 13 |

### Epic 3: Daily Planning

| ID | Story | Points |
|----|-------|--------|
| US-301 | **Daily Routine Generation:** AI schedule by 7am. Goal-aligned. Push notification. | 8 |
| US-302 | **Smart Reminders:** 15min before events. Context-aware. Snooze/dismiss. | 5 |
| US-303 | **Fitness/Diet Adaptation:** Personalized workouts/meals in daily plan. Progress tracking. | 13 |
| US-304 | **Meeting Preparation:** Auto-research attendees via Scrapling. Briefing 15min before. | 13 |

### Epic 4: Research & Scraping

| ID | Story | Points |
|----|-------|--------|
| US-401 | **Deep Research:** Multi-source crawl via Scrapling. LLM synthesis with citations. | 13 |
| US-402 | **Lead Scraping:** Structured data extraction. Table view. CSV export. | 13 |
| US-403 | **Competitor Monitoring:** Scheduled Scrapling checks. Change alerts. History. | 8 |

### Epic 5: Social Media & Content

| ID | Story | Points |
|----|-------|--------|
| US-501 | **Social Post Creation:** AI drafts, preview, multi-platform posting via Composio. | 8 |
| US-502 | **Content Calendar:** AI-suggested times. Drag-to-reschedule. | 8 |
| US-503 | **Video Upload:** Supabase Storage → YouTube/TikTok via API. | 13 |

### Epic 6: Integrations & Skills

| ID | Story | Points |
|----|-------|--------|
| US-601 | **Browse & Connect Apps:** Integration grid. OAuth flow. Status indicators. | 8 |
| US-602 | **Install Agent Skills:** Marketplace. One-tap install. Tool list update. | 8 |
| US-603 | **Custom Skill Creation:** TOML editor. Validation. Share to marketplace. | 21 |

### Epic 7: Payments

| ID | Story | Points |
|----|-------|--------|
| US-701 | **View & Select Plan:** Paywall with feature comparison. RevenueCat purchase. | 5 |
| US-702 | **Upgrade/Downgrade:** Prorated billing. Immediate feature changes. | 5 |
| US-703 | **Free Trial:** 7-day Pro trial. Full features. Paywall at expiry. | 5 |

**Total Story Points: 237**

---

## 13. Development Roadmap

| Phase | Timeline | Deliverables | Features Shipped |
|-------|---------|-------------|-----------------|
| **Phase 0: Foundation** | Weeks 1-2 | Boilerplate, Supabase schema, auth, CI/CD, Gateway service (async pattern), Docker setup on Hetzner, ZeroClaw benchmarking, AI consent screen, Composio startup program application | Infrastructure ready |
| **Phase 1: Core Agent** | Weeks 3-5 | ZeroClaw per-user containers, chat UI, Composio email + calendar (with direct OAuth fallback layer), memory system, model routing (MiniMax + Gemini 2.5 Flash) | **#1 Email, #2 Calendar, #6 Memory** |
| **Phase 2: Secretary** | Weeks 6-8 | Daily brief, task generation, push notifications, news digests, scheduler service | **#3 Daily Brief, #4 Tasks, #7 News** |
| **🚀 MVP Launch** | **Week 8** | **Ship to beta users** | **$30/mo product ready** |
| **Phase 3: Integrations** | Weeks 9-11 | Composio hub UI, Telegram/WhatsApp channels, voice mode | **#5 Multi-Channel** |
| **Phase 4: Scrapling** | Weeks 12-14 | Scrapling sidecar, research reports, lead scraping | Deep research, competitor monitoring |
| **Phase 5: Content** | Weeks 15-16 | Social posting, content calendar, video upload | Social media management |
| **Phase 6: Payments** | Weeks 17-18 | RevenueCat SDK, paywall, Apple/Google/Stripe | Subscription management |
| **Phase 7: Health** | Weeks 19-20 | Meal/health skill, skills marketplace, fitness planning | **#8 Meal Planning** |
| **Phase 8: Polish** | Weeks 21-24 | Beta testing, analytics, App Store submission | Public launch |

---

## 14. Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|-----------|-----------|
| **ZeroClaw immaturity (NEW)** | **Critical** | **High** | ZeroClaw launched Feb 2026 — no production track record. **Pre-launch:** run independent benchmarks under realistic load (multi-tool, memory.db active, HTTP serving). **Fallback plan:** if ZeroClaw proves unreliable, deploy OpenClaw in a slimmed Alpine container (~200MB RAM) or fork ZeroClaw and stabilize critical paths. Build a thin runtime abstraction layer so Gateway does not depend on ZeroClaw-specific APIs. |
| **Apple App Store AI rejection (NEW)** | **Critical** | **High** | Guideline 5.1.2(i) requires explicit AI provider disclosure and user consent before sharing personal data with third-party AI. **Required:** (1) Add AI Data Consent screen to onboarding naming MiniMax as the AI provider and listing data types shared. (2) Add Settings > AI & Privacy screen. (3) Include detailed AI disclosure in App Store Review Notes. (4) Ensure all consent is opt-in, not pre-checked. (5) Build with iOS 26 SDK before April 28, 2026 deadline. |
| **Supabase Edge Function CPU limit (NEW)** | **Critical** | **High** | Edge Functions have a 2-second CPU limit. The original request flow used Edge Functions as synchronous orchestrators — this will timeout. **Fixed:** Edge Functions now act as thin auth proxies (validate JWT + fire-and-forget POST to Gateway). Gateway handles orchestration and pushes responses via Supabase Realtime or webhook. Edge Function never waits for container response. See architecture.md Section 8. |
| **Composio pricing at scale (UPDATED)** | **High** | **High** | Composio Starter: $99/mo for 1K users, 100K API calls. Growth: $199/mo for 5K users. Enterprise: custom. Each feature (inbox digest, calendar sync) generates hundreds of API calls/user/day. **Mitigations:** (1) Apply for Composio startup program ($25K credits). (2) Build abstraction layer — implement direct OAuth for Gmail, Google Calendar, Slack as fallback. (3) Cache Composio responses where possible. (4) Monitor API call volume per user. |
| MiniMax M2.5 quality insufficient | High | Medium | M2.5 excels at tool-use (SWE-Bench 80.2%) but weak at general reasoning (SimpleQA 44%). **Pre-launch:** A/B test MiniMax vs Gemini 2.5 Flash for email categorization, scheduling, and summarization. Implement model routing — use cheaper/better models per task type. Claude/GPT-4o available on Power tier. |
| Scrapling blocked / legal risk | Medium | Medium | Scrapling anti-bot bypass drew negative press (WIRED, Feb 2026). **Tiered approach:** use official APIs for HN/Reddit/YouTube/RSS. Reserve Scrapling StealthyFetcher for user-initiated research URLs only. Add ToS placing responsibility on users. Consider managed proxy service (ScraperAPI, BrightData) as paid-tier fallback. |
| Docker container scaling limits | High | Low | ZeroClaw's <5MB footprint. Phase 2: k3s (skip Docker Swarm — stagnant ecosystem, no autoscaling). |
| Docker socket security | High | Low | Socket proxy restricts API. Gateway is only service with access. Phase 2: evaluate gVisor for kernel-level container sandboxing. |
| Single VPS failure | High | Low | Hetzner Storage Box backups every 6h. 15-min recovery procedure. |
| User data breach | Critical | Low | Per-container isolation. Encrypted secrets. Non-root. Read-only FS. |
| **GDPR cross-border data transfers (NEW)** | **High** | **Medium** | User data sent to MiniMax (China), Composio (USA), Deepgram (USA), ElevenLabs (USA/Poland). Requires Data Processing Agreements (DPAs) with each processor. Cross-border transfers need Standard Contractual Clauses (SCCs). MiniMax (China) is highest risk. **Action:** engage GDPR lawyer before launch, prepare DPAs, add data flow transparency to privacy policy and in-app Settings. |
| **Heavy user margin erosion (NEW)** | **High** | **Medium** | P90 heavy users (50+ tasks/day) could cost $15-25/mo — exceeding $30 subscription. **Mitigations:** per-user cost tracking from day one, fair use policy with soft token budgets, model routing to cheaper models for simple tasks, usage alerts when cost exceeds 50% of subscription price. |

---

## 15. Success Metrics

| Metric | Month 3 | Month 6 | Month 12 |
|--------|---------|---------|----------|
| Monthly Active Users | 500 | 2,000 | 10,000 |
| Paying Subscribers | 100 | 500 | 2,500 |
| MRR | $3,000 | $17,500 | $100,000 |
| D7 Retention | 40% | 50% | 60% |
| Agent Tasks/User/Day | 10 | 20 | 30 |
| Avg Response Time | <3s | <2s | <1.5s |
| Container Cold Start | <3s | <2s | <2s |
| App Store Rating | 4.2+ | 4.4+ | 4.6+ |

---

## 16. References

| Resource | Link |
|----------|------|
| Per-User Docker Pattern (Adola) | [dev.to article](https://dev.to/reeddev42/running-one-docker-container-per-user-on-a-35month-server-51hh) |
| Per-User Container Isolation Pattern | [dev.to article](https://dev.to/reeddev42/per-user-docker-container-isolation-a-pattern-for-multi-tenant-ai-agents-8eb) |
| ZeroClaw Agent Runtime | [github.com/zeroclaw-labs/zeroclaw](https://github.com/zeroclaw-labs/zeroclaw) |
| Scrapling Web Scraper | [github.com/D4Vinci/Scrapling](https://github.com/D4Vinci/Scrapling) |
| Awesome OpenClaw Use Cases | [github.com/hesamsheikh/awesome-openclaw-usecases](https://github.com/hesamsheikh/awesome-openclaw-usecases) |
| OpenClaw Use Cases (35 real ways) | [sidsaladi.substack.com](https://sidsaladi.substack.com/p/openclaw-use-cases-35-real-ways-people) |
| Docker Security (OWASP) | [cheatsheetseries.owasp.org](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html) |
| Docker Socket Proxy | [github.com/Tecnativa/docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy) |
| k3s Lightweight Kubernetes | [k3s.rocks](https://k3s.rocks/) |
| Expo Supabase AI Template | [github.com/abdulhaseeb7603/expo-supabase-ai-template](https://github.com/abdulhaseeb7603/expo-supabase-ai-template) |
| **Market Research & Validation** | |
| AI Assistant Market Size ($21.1B by 2030) | [MarketsandMarkets](https://www.marketsandmarkets.com/Market-Reports/ai-assistant-market-40111511.html) |
| Agentic AI Market ($45B by 2030) | [Deloitte / Oreate AI](https://www.oreateai.com/blog/the-agentic-ai-surge-charting-a-course-to-a-45-billion-market-by-2030/b4801f3e172e30ecd30ffcca031057c9) |
| Worldwide AI Spending $2.52T (2026) | [Gartner](https://www.gartner.com/en/newsroom/press-releases/2026-1-15-gartner-says-worldwide-ai-spending-will-total-2-point-5-trillion-dollars-in-2026) |
| Consumer Gen AI App Spending >$10B (2026) | [Visual Capitalist / Sensor Tower](https://www.visualcapitalist.com/charted-the-explosive-growth-of-gen-ai-apps/) |
| State of Consumer AI 2025 (3% pay) | [Menlo Ventures](https://menlovc.com/perspective/2025-the-state-of-consumer-ai/) |
| State of Consumer AI 2025 (a16z) | [Andreessen Horowitz](https://a16z.com/state-of-consumer-ai-2025-product-hits-misses-and-whats-next/) |
| AI Platform Retention Benchmarks | [Arcade Research](https://www.arcade.dev/blog/user-retention-in-ai-platforms-metrics) |
| The AI Churn Wave | [Growth Unhinged](https://www.growthunhinged.com/p/the-ai-churn-wave) |
| State of Subscription Apps 2025 | [RevenueCat](https://www.revenuecat.com/state-of-subscription-apps-2025/) |
| OpenClaw Rise & Growth | [CNBC](https://www.cnbc.com/2026/02/02/openclaw-open-source-ai-agent-rise-controversy-clawdbot-moltbot-moltbook.html) |
| OpenClaw Wikipedia | [Wikipedia](https://en.wikipedia.org/wiki/OpenClaw) |
| OpenAI Acquires OpenClaw Creator | [Leanware](https://www.leanware.co/insights/openai-openclaw-acquisition) |
| Why AI Startups Fail (2026) | [Medium](https://medium.com/write-a-catalyst/the-great-ai-collapse-of-2026-why-most-startups-are-failing-and-how-to-build-an-unbreakable-moat-94b81d57df72) |
| AI Startup Failures (2025) | [TechStartups](https://techstartups.com/2025/12/09/top-ai-startups-that-shut-down-in-2025-what-founders-can-learn/) |
| AI Agent Pricing 2026 | [NoCodeFinder](https://www.nocodefinder.com/blog-posts/ai-agent-pricing) |
| How to Price AI Products | [Aakash Gupta](https://www.news.aakashg.com/p/how-to-price-ai-products) |

---

*— End of PRD —*
