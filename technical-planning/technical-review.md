# ZeroClaw AI — Technical Planning Review & Analysis

> Research-Based Review of PRD v3.0, Architecture v3.0, and Feature Spec v3.0
> March 1, 2026

---

## Overview

This document is a research-backed review of all three technical planning documents (PRD, Architecture, Feature Spec). Every recommendation is grounded in online research from official documentation, industry benchmarks, community reports, and expert analyses as of early 2026.

---

## CRITICAL ISSUES (Must Change)

### 1. Supabase Edge Functions: 2-Second CPU Limit Will Break the Agent Pipeline

**Affected Documents:** architecture.md (Section 8 — Request Flow)

**The Problem:**
The architecture routes every user message through Supabase Edge Functions as the orchestrator. Supabase Edge Functions have a hard **2-second CPU time limit** per request. The request flow (Section 8 of architecture.md) has the Edge Function doing: JWT validation + subscription check + rate limit check + POST to Gateway + wait for response + log to DB + push via Realtime. If any step involves non-trivial CPU work, the function will hit 504 timeouts.

Community developers report the limit is "too short to use AI functionalities," and Supabase themselves describe Edge Functions as "best treated as glue logic rather than full backend replacements." The wall-clock idle timeout is 150 seconds, but CPU-bound operations are capped at 2 seconds. Edge Functions also have 200-400ms cold starts that require warming for latency-sensitive endpoints.

**Recommendation:**
- Restructure the Edge Function to do **only** auth validation and fire-and-forget forwarding
- Move all orchestration logic into the Gateway itself
- Use Supabase Realtime only for pushing responses back to the client
- The Edge Function should never wait for the container response — use an async pattern where the Gateway pushes results directly to Supabase Realtime or a webhook
- Consider the Edge Function as a thin auth proxy, not a processing layer

**Sources:**
- [Supabase Edge Function Limits](https://supabase.com/docs/guides/functions/limits)
- [Supabase Edge Function CPU Limits Troubleshooting](https://supabase.com/docs/guides/troubleshooting/edge-function-cpu-limits)
- [Supabase Edge Functions Architecture](https://supabase.com/docs/guides/functions/architecture)

---

### 2. ZeroClaw Is Extremely New and Unproven — High Risk as Core Runtime

**Affected Documents:** prd.md (Section 5 — Tech Stack, Section 13 — Risks), architecture.md (all sections)

**The Problem:**
ZeroClaw appeared on GitHub in **mid-February 2026** — roughly 2 weeks before the planning documents were dated. It has ~11,500 stars but no independent benchmarks validating its performance claims. The <5MB RAM and <10ms startup claims have no published methodology. A Medium comparison of Rust agent runtimes noted the repository still has "planned Docker/WASM work" incomplete.

The entire product depends on ZeroClaw as the agent runtime. If it has bugs, architectural limitations, or the project stalls, there is no fallback. The project is MIT licensed and built by contributors from Harvard/MIT/Sundai.Club, but maturity is extremely low for a production dependency.

Every subsystem in ZeroClaw (providers, channels, tools, memory, tunnels, runtime, security, identity, observability) is defined as a Rust trait, which means the architecture is pluggable — but also means the interface contracts could change rapidly at this stage.

**Recommendation:**
- Add a **concrete fallback plan** to the PRD risks table: if ZeroClaw proves unreliable, what is the alternative? (OpenClaw in a slimmed container, a custom lightweight agent runtime, or a fork of ZeroClaw)
- Before committing, **run independent benchmarks** — validate the <5MB RAM claim under realistic workloads (multiple tools loaded, SQLite memory DB active, HTTP server running, Composio integrations active)
- Track the repository closely — at this maturity stage, breaking changes are likely
- Consider building a **thin abstraction layer** over the agent runtime so the runtime can be swapped later without rewriting the Gateway or container infrastructure
- Evaluate alternatives: MicroClaw (chat-native workflows) and Moltis (full gateway/platform capability) are competing Rust agent runtimes with different strengths

**Sources:**
- [ZeroClaw: Lightweight Rust Agent Runtime (DEV Community)](https://dev.to/brooks_wilson_36fbefbbae4/zeroclaw-a-lightweight-secure-rust-agent-runtime-redefining-openclaw-infrastructure-2cl0)
- [Rust Agent Runtime Showdown: MicroClaw vs ZeroClaw vs Moltis (Medium)](https://medium.com/@everettjf/rust-agent-runtime-showdown-microclaw-vs-zeroclaw-vs-moltis-df1ecb85c676)
- [ZeroClaw GitHub](https://github.com/zeroclaw-labs/zeroclaw)

---

### 3. Composio Pricing Destroys the Unit Economics

**Affected Documents:** prd.md (Section 8 — Pricing Strategy, Unit Economics table)

**The Problem:**
The PRD estimates Composio costs at ~$0.50-1.00/user/month. Composio's actual pricing tiers are:

| Tier | Price | User Accounts | API Calls/Month |
|------|-------|--------------|-----------------|
| Hobby (Free) | $0 | 100 | 5,000 |
| Starter | $99/mo | 1,000 | 100,000 |
| Growth | $199/mo | 5,000 | 500,000 |
| Enterprise | Custom | Custom | Custom |

At the target of 10,000 users, the Enterprise tier would be required (custom pricing). Even at Growth ($199/mo for 5K users), that is $0.04/user/month in platform fees alone — but API call volume is the real concern. A single inbox digest scanning 200 emails generates hundreds of API calls per user per day. 10K users doing daily email + calendar operations could easily generate millions of API calls per month, far exceeding included quotas.

Rate limits also apply: Free tier gets 100/min tool calls, Paid gets 5,000/min, Enterprise gets custom limits.

**Recommendation:**
- Contact Composio for enterprise pricing and volume API call rates before finalizing unit economics
- Apply for their **startup program** immediately ($25K in free credits, $2K after free period, most startups run 6+ months on that)
- Update the unit economics table with **realistic Composio costs** based on actual API call projections per feature
- Build an **abstraction layer** so the top 3-5 integrations (Gmail, Google Calendar, Slack) could be replaced with direct OAuth integration if Composio becomes too expensive
- Factor in that Composio charges per API call at volume, not just per user account

**Sources:**
- [Composio Pricing](https://composio.dev/pricing)
- [Composio Startup Program](https://composio.dev/startups)
- [Composio Reviews (G2)](https://www.g2.com/products/composio/reviews)
- [Composio Overview (SalesForge)](https://www.salesforge.ai/directory/sales-tools/composio)

---

### 4. Apple App Store AI Data Sharing Rules — Rejection Risk Is HIGH

**Affected Documents:** prd.md (Section 13 — Risks), featurespec.md (F-100 — Onboarding)

**The Problem:**
Apple's **Guideline 5.1.2(i)** (updated November 2025) now requires that apps sharing personal data with third-party AI must:
- Explicitly disclose the AI provider and data types being shared
- Obtain explicit user permission before any data transmission
- Show a consent modal specifying the provider and purpose

The app sends user emails, calendar data, and personal messages to MiniMax M2.5 (a third-party Chinese AI model). This is exactly what Apple is scrutinizing. AI apps face greater scrutiny in 2026 — some apps have been delayed simply because the team did not mention the AI component in their App Store Review Notes.

Additionally, starting **April 28, 2026**, all apps uploaded to App Store Connect must be built with the iOS 26 SDK or later.

**Recommendation:**
- Add a **dedicated AI data consent screen** to onboarding (Step 0, before anything else) that explicitly names MiniMax as the AI provider
- Add a **Settings > AI & Privacy** screen showing exactly what data is shared, with whom, and for what purpose
- Document this in both the PRD and Feature Spec
- Update the F-100 Onboarding Wizard to include an AI consent step
- Consider offering an **on-device processing option** for sensitive data or using Apple's own AI APIs where possible
- The PRD's risk table mentions "App Store rejection" but the mitigation ("Ship supervised mode only. Clear consent. Content moderation.") is too vague — make it concrete with specific UI flows and consent mechanisms
- Ensure all App Store Review Notes explicitly describe the AI integration, the data flow, and the consent mechanism

**Sources:**
- [Apple App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- [Apple AI Data Sharing Rules (TechCrunch)](https://techcrunch.com/2025/11/13/apples-new-app-review-guidelines-clamp-down-on-apps-sharing-personal-data-with-third-party-ai/)
- [Navigating AI Rejections in App Store Submissions](https://appitventures.com/blog/navigating-ai-rejections-app-store-play-store-submissions)
- [Top Reasons iOS Apps Get Rejected in 2026](https://www.eitbiz.com/blog/top-reasons-ios-apps-get-rejected-by-the-app-store-and-fixes/)

---

## HIGH PRIORITY ISSUES (Should Change)

### 5. Expo SDK 54 + NativeWind Has Active Compatibility Issues

**Affected Documents:** prd.md (Section 5 — Tech Stack)

**The Problem:**
The tech stack specifies Expo SDK 54 + NativeWind without version pinning. Research shows this combination has known compatibility problems:

- NativeWind v4 requires Reanimated v3, but Expo SDK 54 ships with **Reanimated v4** — these are incompatible
- Multiple GitHub issues report broken styling, lost `ref` props, and build failures
- NativeWind v5 is available but still in preview/unstable state
- Expo SDK 54 is the **last SDK to support the Old Architecture** — SDK 55 (in beta) will only support the New Architecture
- As of January 2026, approximately 83% of SDK 54 projects use the New Architecture

Additional risk: Babel config conflicts between `react-native-reanimated/plugin` and `react-native-worklets/plugin` cause duplicate plugin detection. The worklets plugin is already included inside the reanimated plugin.

**Recommendation:**
- Pin to **NativeWind v4.2.0+** with **Tailwind CSS 3.4.17** (not v4.x which is for NativeWind v5)
- Test NativeWind v5 compatibility early, as that is the forward path
- Enable the **New Architecture now** — do not wait until SDK 55 forces it
- Add explicit version pinning to the tech stack table in the PRD
- Plan for SDK 55 migration within 3-6 months of launch
- Upgrade SDK and architecture **separately**, not simultaneously, to isolate issues
- Force `lightningcss` to a specific version in package.json to avoid deserialization errors

**Sources:**
- [Expo SDK 54 Changelog](https://expo.dev/changelog/sdk-54)
- [NativeWind + SDK 54 ref prop issue (GitHub #39657)](https://github.com/expo/expo/issues/39657)
- [NativeWind + Expo Version Compatibility Discussion](https://github.com/nativewind/nativewind/discussions/1604)
- [NativeWind Styling Issues with SDK 54 (Medium)](https://medium.com/@matthitachi/nativewind-styling-not-working-with-expo-sdk-54-54488c07c20d)
- [NativeWind v5 Installation Guide](https://www.nativewind.dev/v5/getting-started/installation)

---

### 6. MiniMax M2.5 — General Reasoning Weakness + Reward-Hacking History

**Affected Documents:** prd.md (Section 5 — Tech Stack, Section 13 — Risks)

**The Problem:**
MiniMax M2.5 is strong in coding and agentic tool use (80.2% SWE-Bench Verified, #4 on OpenHands Index, 51.3% Multi-SWE-Bench). However, the product is a **personal assistant** that needs to:
- Understand natural language scheduling ("next Tuesday at 2pm")
- Categorize emails by urgency with 90%+ accuracy
- Generate personalized meal plans considering dietary restrictions
- Summarize news articles and produce coherent daily briefs

M2.5's weaknesses are exactly in these areas: **AIME 2025 at 45%, SimpleQA at 44%, Terminal-Bench 2 at 52% vs Opus 65.4%**. It was "optimized for coding and agentic tasks, not broad knowledge work." The community has also flagged MiniMax's **history of benchmark reward-hacking** with previous versions (M2 and M2.1).

The model is a 230B Mixture-of-Experts with 10B active parameters and a 200K context window. The cost advantage ($0.20-0.30/M input, $1.00-1.20/M output) is real and significant — approximately 13x cheaper than Opus.

**Recommendation:**
- **A/B test MiniMax M2.5 vs. Gemini 2.5 Flash** for non-coding tasks (email categorization, scheduling, summarization) before launch
- Consider using **different models for different tasks**: MiniMax for tool-use/agentic tasks, a stronger reasoning model (Gemini 2.5 Flash, Claude Haiku) for natural language understanding tasks
- Add a **model routing layer** in the Gateway or container config that selects the model based on task type
- Budget for higher model costs if MiniMax's quality proves insufficient for core features
- The risk table mitigation "A/B test DeepSeek V3.2, Gemini 2.5 Flash" is correct but should be elevated to a pre-launch requirement, not a reactive measure

**Sources:**
- [MiniMax M2.5 Official Announcement](https://www.minimax.io/news/minimax-m25)
- [MiniMax M2.5 Benchmark Analysis (Artificial Analysis)](https://artificialanalysis.ai/models/minimax-m2-5)
- [MiniMax M2.5 Review (Thomas Wiegold)](https://thomas-wiegold.com/blog/minimax-m25-review/)
- [MiniMax M2.5 OpenHands Analysis](https://openhands.dev/blog/minimax-m2-5-open-weights-models-catch-up-to-claude)
- [Open Source LLM Leaderboard 2026](https://vertu.com/lifestyle/open-source-llm-leaderboard-2026-rankings-benchmarks-the-best-models-right-now/)

---

### 7. Docker Swarm for Phase 2 Scaling — Skip It, Go Straight to k3s

**Affected Documents:** architecture.md (Section 7 — Scaling Strategy)

**The Problem:**
The scaling strategy uses Docker Swarm for Phase 2 (1K-10K users). Research shows:

- Docker Swarm has a **declining ecosystem** with "limited new features, smaller ecosystem, and declining market share"
- **No autoscaling** — Swarm cannot watch CPU/memory and auto-scale replicas like Kubernetes HPA
- No equivalent to Kubernetes operators for database management, backups, or failover
- Basic secrets management with no external integration, no rotation strategies
- Swarm development has slowed and represents a "stable rather than growing platform choice"

Swarm is confirmed supported through 2030 by Mirantis, so it is not dead. However, it is a **dead-end technology** that would require migration to Kubernetes anyway at Phase 3, creating two migration steps instead of one.

k3s is a single ~100MB binary that implements the full Kubernetes API, compatible with existing Docker images and compose patterns. It can start with a single-node "cluster" on a $5/month server and scale up by adding nodes with a few commands.

**Recommendation:**
- Skip Docker Swarm entirely
- Go from **Docker Compose (Phase 1)** directly to **k3s (Phase 2)**
- k3s supports HPA autoscaling, proper secrets management, and has a growing ecosystem
- This eliminates one migration step and avoids investing in a stagnating platform
- Update the scaling strategy in architecture.md:
  - Phase 1: Docker Compose on single VPS (0-1,000 users)
  - Phase 2: k3s cluster on multiple Hetzner VPS (1,000-10,000+ users)
- Consider [Syself Autopilot](https://syself.com/hetzner) for managed k3s on Hetzner if operational overhead is a concern

**Sources:**
- [Docker Swarm vs Kubernetes 2026 (The Decipherist)](https://thedecipherist.com/articles/docker_swarm_vs_kubernetes/)
- [Docker Swarm vs Kubernetes 2026 (Hacker News Discussion)](https://news.ycombinator.com/item?id=47039073)
- [Mirantis Long-Term Swarm Support Through 2030](https://www.mirantis.com/blog/mirantis-guarantees-long-term-support-for-swarm/)
- [K3s Rocks — Production Guide](https://k3s.rocks/)
- [Docker Swarm Guide 2026 (GitHub)](https://github.com/TheDecipherist/docker-swarm-guide)

---

### 8. Scrapling: Legal and Reliability Risk

**Affected Documents:** prd.md (Section 13 — Risks), featurespec.md (F-400 — Scrapling Deep Research)

**The Problem:**
Scrapling has received significant negative attention in February 2026:

- A **WIRED report** highlighted OpenClaw + Scrapling being used to bypass Cloudflare Turnstile (which protects roughly 20% of all websites), making security teams nervous
- Scrapling "doesn't guarantee continuous success" for anti-bot bypass — what works today may fail tomorrow as anti-bot systems evolve
- The legal landscape around scraping is tightening, especially in the EU where Hetzner servers are located
- OpenClaw's ecosystem has faced serious issues: "a one-click remote code execution exploit, over 800 malicious skills on ClawHub, and more than 42,000 publicly exposed instances"
- The gap between "works on my laptop" and "works in production for months" is significant — success rates must be monitored and approaches updated regularly

Scrapling's StealthyFetcher can bypass Cloudflare's Turnstile in simple scenarios, but it is "limited to simple scenarios" for reliable production use.

**Recommendation:**
- **Tier the scraping approach:**
  - For APIs with official access (Hacker News, Reddit, YouTube): use their official APIs directly, not Scrapling
  - For RSS/Atom feeds: use a standard RSS parser library
  - Reserve Scrapling's StealthyFetcher **only** for user-initiated research on specific URLs
- Add a **Terms of Service** that places responsibility on users for URLs they choose to scrape
- Consider offering a **managed proxy rotation service** (BrightData, ScraperAPI, ScrapingBee) as a fallback for higher reliability on paid tiers
- Update the risk table with explicit legal/reputational risk for scraping
- Monitor scraping success rates and have automatic fallback to alternative fetching methods
- Add rate limiting and domain blocklists to prevent scraping of sensitive/protected sites

**Sources:**
- [OpenClaw + Scrapling Bypassing Cloudflare (Techstrong.ai)](https://techstrong.ai/features/openclaw-users-are-using-scrapling-to-bypass-cloudflare-and-other-anti-bot-systems/)
- [Scrapling GitHub](https://github.com/D4Vinci/Scrapling)
- [Web Scraping with Scrapling Tutorial (ZenRows)](https://www.zenrows.com/blog/scrapling-web-scraper)
- [Scrapling Documentation](https://scrapling.readthedocs.io/en/latest/index.html)

---

### 9. Voice Latency Target of <2s Is Unrealistic

**Affected Documents:** featurespec.md (F-005, AC-5; Section 17.1 — Performance NFRs)

**The Problem:**
The feature spec targets <2 seconds end-to-end for voice (STT + LLM + TTS). Research on actual production latencies shows:

| Component | Latency Range |
|-----------|--------------|
| Deepgram STT (Nova-3) | 150-184ms |
| LLM processing (MiniMax M2.5) | 350ms-1s+ |
| ElevenLabs TTS (Flash v2.5) | 75-300ms |
| Network overhead (2 hops) | 100-200ms |
| **Total realistic range** | **675ms-1.7s+** |

Under load, ElevenLabs queue delays can stretch TTS to 600ms, and LLM processing time is highly variable based on response length and complexity. Most voice AI agents in production take 800ms to 2 seconds to respond. The <2s target would be achievable only under ideal conditions with short responses.

Additionally, Deepgram's pricing is $0.0043/min for STT and $0.030/1K chars for TTS. ElevenLabs Flash v2.5 is $0.050/1K chars. These costs should be factored into the unit economics for voice-enabled tiers.

**Recommendation:**
- Change the voice latency target to **<3 seconds** for the initial release, with a stretch goal of <2 seconds
- Consider **Deepgram's Voice Agent API** ($4.50/hour bundled) which provides STT+TTS+orchestration with 200-250ms total latency — significantly faster than the separate pipeline approach
- Use **streaming TTS** (start playing audio before the full response is generated) to reduce perceived latency
- Update both the Feature Spec (F-005, AC-5) and the NFR Performance table (Section 17.1)
- Add voice API costs to the unit economics table for Pro and Power tiers
- For latency reference: under 300ms feels natural, 300-600ms is acceptable, over 1 second feels robotic

**Sources:**
- [Deepgram vs ElevenLabs Comparison](https://deepgram.com/learn/deepgram-vs-elevenlabs)
- [Voice AI Infrastructure Guide (Introl)](https://introl.com/blog/voice-ai-infrastructure-real-time-speech-agents-asr-tts-guide-2025)
- [Deepgram Pricing](https://deepgram.com/pricing)
- [Best AI Voice Models 2026 Comparison](https://www.teamday.ai/blog/best-ai-voice-models-2026)
- [How to Choose STT/TTS for Voice Agents (Softcery)](https://softcery.com/lab/how-to-choose-stt-tts-for-ai-voice-agents-in-2025-a-comprehensive-guide)

---

## MEDIUM PRIORITY ISSUES (Consider Changing)

### 10. SQLite Hybrid Search — Appropriate but Add Caveats

**Affected Documents:** architecture.md (Section 3 — Per-User Data Volume), featurespec.md (F-006)

**The Problem:**
Using SQLite with sqlite-vec for per-user memory is a good fit for the per-user isolation pattern. sqlite-vec is "particularly appealing in situations where each customer or user has their very own database instance." However, there are limitations the documents should acknowledge:

- sqlite-vec performs **exact nearest neighbor search** (brute-force full table scan), not approximate nearest neighbor — this degrades significantly at scale
- Hybrid search requires a **separate FTS5 extension** alongside sqlite-vec, adding complexity
- There is no approximate nearest neighbor index — every query traverses all embeddings
- The scale ceiling per database is limited: "A single sqlite-vec database can't serve millions of shared vectors efficiently"

For per-user use cases with <100K embeddings per user, this is fine. The acceptance criterion AC-7 (memory size under 500MB for 1 year of daily use) is reasonable.

**Recommendation:**
- Add a note in architecture.md acknowledging that if memory.db grows beyond ~100K embeddings per user, performance will degrade
- Document a migration path to pgvector (available via Supabase PostgreSQL) for users who accumulate very large memory databases
- Enforce the 500MB per-user limit at the container level, not just as a target
- Consider periodic pruning of old, low-relevance embeddings
- Note that pgvector with PostgreSQL supports hybrid search natively (via tsvector) and scales to 5-10M vectors comfortably

**Sources:**
- [SQLite Vector Search and RAG (Turso)](https://turso.tech/blog/sqlite-retrieval-augmented-generation-and-vector-search)
- [pgvector 2026 Guide (Instaclustr)](https://www.instaclustr.com/education/vector-database/pgvector-key-features-tutorial-and-pros-and-cons-2026-guide/)
- [Best Vector Databases 2026 (Firecrawl)](https://www.firecrawl.dev/blog/best-vector-databases)

---

### 11. Hetzner Price Increases — Update Cost Estimates

**Affected Documents:** prd.md (Section 8 — Unit Economics), architecture.md (Section 7.3 — Cost Table)

**The Problem:**
Hetzner increased prices **up to 37%** on some plans in April 2026 due to industry-wide DRAM and NAND flash cost increases. The cost tables in the planning documents reference pre-increase pricing (e.g., CX42 at ~€30/mo). Even after the increase, Hetzner remains competitive (less than DigitalOcean or Vultr for similar specs), but the numbers in the documents are now inaccurate.

Hetzner also introduced new plan categories: Cost-Optimized, Regular Performance, and Dedicated General Purpose, with different AMD EPYC-Genoa processors for the Regular tier.

**Recommendation:**
- Re-check [Hetzner's current pricing page](https://www.hetzner.com/cloud) and update all cost tables
- Update both the PRD unit economics table and the architecture scaling cost table
- Consider the new plan categories when selecting server types — Dedicated General Purpose may be more appropriate for production workloads running many containers
- Note that European datacenters include 20TB bandwidth vs 1TB for US/Singapore

**Sources:**
- [Hetzner Cloud Pricing](https://www.hetzner.com/cloud)
- [Hetzner Cloud Review 2026 (Bitdoze)](https://www.bitdoze.com/hetzner-cloud-review/)
- [Hetzner VPS Review 2026 (EXPERTE)](https://www.experte.com/server/hetzner)

---

### 12. Pricing Strategy: $30/mo Margin Risk from Heavy Users

**Affected Documents:** prd.md (Section 8 — Pricing Strategy)

**The Problem:**
The $30/mo price point is competitive (same as ChatGPT Business) and well-positioned in the consumer/SMB segment. However:

- **Replit's cautionary tale**: gross margins swung from 36% to -14% when AI agent consumption exceeded what their pricing covered
- The 83% margin estimate assumes MiniMax stays at current prices and usage stays at average levels
- A power user doing 50+ agent tasks/day could cost $10-15/month in LLM tokens alone
- Nearly half of companies in 2026 use hybrid pricing (2-3 models simultaneously) — pure flat-rate is becoming less common
- AI costs are deflating but consumption patterns are unpredictable

The pricing comparison table is strong, but the unit economics analysis only models average users.

**Recommendation:**
- Add **usage-based guardrails** even on fixed-price plans (e.g., fair use policy, monthly token budget per tier)
- Implement **real-time cost tracking** per user from day one — this is essential for margin management
- Stress-test unit economics with a **heavy user model** (90th percentile), not just average usage
- Consider a **soft usage cap** with overage charges rather than hard limits that degrade experience
- Monitor per-user costs weekly during beta and adjust tier limits before public launch
- Document the cost floor per user (minimum costs even for inactive users: container storage, Composio entity, Supabase row overhead)

**Sources:**
- [How to Price AI Products (Aakash Gupta)](https://www.news.aakashg.com/p/how-to-price-ai-products)
- [AI Agent Pricing 2026 Guide (NoCodeFinder)](https://www.nocodefinder.com/blog-posts/ai-agent-pricing)
- [Selling Intelligence: Pricing AI Agents Playbook (Chargebee)](https://www.chargebee.com/blog/pricing-ai-agents-playbook/)
- [The 2026 Guide to SaaS, AI, and Agentic Pricing Models](https://www.getmonetizely.com/blogs/the-2026-guide-to-saas-ai-and-agentic-pricing-models)

---

### 13. Container Security: Consider gVisor for Stronger Isolation

**Affected Documents:** architecture.md (Section 5 — Security Architecture)

**The Problem:**
The container hardening is solid (non-root, read-only FS, dropped capabilities, PID limits, `no-new-privileges`). However, standard Docker containers share the host kernel. This means a kernel exploit in any user container could theoretically compromise the host and all other containers.

Research recommends that for stricter isolation, **sandbox containers like gVisor** or **virtualized containers like Kata Containers / AWS Firecracker** add a kernel-level isolation boundary. This is especially relevant when running untrusted user workloads (agent tasks with LLM-generated tool calls that could be unpredictable).

The Docker security documentation states: "If container isolation fails, the root user inside the container maps to the root user on the host. Always drop privileges." The current setup does drop privileges, but a defense-in-depth approach would add an additional kernel isolation layer.

**Recommendation:**
- Add gVisor as a **Phase 2 enhancement** for container security
- Mention gVisor/Kata Containers in the security section as a planned upgrade path
- For Phase 1, the current hardening is acceptable — the combination of non-root, read-only FS, dropped capabilities, PID limits, and network isolation provides strong baseline security
- Consider running **Docker Bench for Security** (automated CIS benchmark auditing) as part of the CI/CD pipeline
- Disable inter-container communication (ICC) at the Docker daemon level with `--icc=false` — the architecture mentions this as possible but does not enforce it

**Sources:**
- [Container Security Best Practices 2026 (OX Security)](https://www.ox.security/blog/container-security-best-practices/)
- [Best Practices for Container Isolation (Snyk)](https://snyk.io/blog/best-practices-for-container-isolation/)
- [Docker Security Practices (LinuxServer.io)](https://www.linuxserver.io/blog/docker-security-practices)
- [Docker Security (Official Docs)](https://docs.docker.com/engine/security/)
- [Docker Socket Myths (Adam Faris)](https://amf3.github.io/articles/virtualization/docker_socket/)

---

### 14. Missing: GDPR Data Processing Agreements + Data Residency Clarity

**Affected Documents:** prd.md (Section 9 — Security Model), featurespec.md (Section 17.5 — Compliance)

**The Problem:**
The compliance section mentions GDPR data deletion and Hetzner EU datacenter. However, user data is sent to multiple non-EU third-party processors:

| Service | Company HQ | Data Sent |
|---------|-----------|-----------|
| MiniMax M2.5 | China | User emails, calendar data, personal messages, goals |
| Composio | USA | OAuth tokens, API calls to user services |
| Deepgram | USA | Voice recordings (speech-to-text) |
| ElevenLabs | USA/Poland | Text content (text-to-speech) |

Under GDPR, Data Processing Agreements (DPAs) are required with each processor handling EU personal data. Cross-border data transfers to countries without EU adequacy decisions (USA, China) require Standard Contractual Clauses (SCCs) or equivalent legal mechanisms.

Sending personal data to a Chinese AI model (MiniMax) is particularly sensitive from a GDPR perspective and may also trigger concerns under EU cybersecurity regulations.

**Recommendation:**
- Add a **dedicated compliance section** addressing:
  - DPAs with all third-party AI/data processors
  - Cross-border data transfer mechanisms (SCCs for US/China transfers)
  - Data flow diagram showing exactly which data goes where
- Add user-facing transparency about where data is processed (required by GDPR Articles 13-14)
- Consider offering a **data residency option** for EU-only processing (use EU-hosted LLM alternatives)
- This overlaps with Apple App Store requirements (Issue #4) — a single comprehensive consent and transparency flow would address both
- Consult with a GDPR-specialized lawyer before launch, especially regarding the MiniMax (China) data flow

**Sources:**
- [Apple AI Data Sharing Rules (TechCrunch)](https://techcrunch.com/2025/11/13/apples-new-app-review-guidelines-clamp-down-on-apps-sharing-personal-data-with-third-party-ai/)
- [Container Security Best Practices — Governance (OX Security)](https://www.ox.security/blog/container-security-best-practices/)

---

## WHAT'S GOOD (No Changes Needed)

The following architectural and technology decisions are well-researched and align with industry best practices:

### Per-User Docker Container Isolation Pattern
The core isolation pattern is sound and matches recommendations from Docker security documentation, Snyk container isolation best practices, and the Adola production reference. The one-container-per-user approach provides genuine data isolation that shared-process multi-tenancy cannot match.

### Bind Mounts Over Docker Volumes
Correct choice for the scheduler architecture. The ability to read user schedule files while containers are stopped is a real operational advantage that Docker named volumes would make difficult.

### Docker Socket Proxy (Tecnativa)
Industry standard solution, properly configured. The environment variable restrictions (CONTAINERS=1, NETWORKS=0, IMAGES=0, VOLUMES=0) match the principle of least privilege. The separate `gateway-proxy` network is correct.

### RevenueCat for Payments
Solid choice. The React Native SDK (v9.0.0) now supports React Native Web, enabling cross-platform subscription management. RevenueCat handles App Store/Google Play/Stripe billing complexity and provides A/B testing for pricing. The web billing support is new and directly relevant for the planned web app.

### Redis for Container State Mapping
Appropriate for ephemeral lookup data (user_id → container name + last_active timestamp). The `allkeys-lru` eviction policy is correct for a cache-like workload.

### Caddy for Reverse Proxy
Simpler than Nginx, automatic TLS via Let's Encrypt, correct choice for a small team. No unnecessary complexity.

### Container Hardening Settings
Non-root user (1000:1000), read-only root filesystem, dropped all capabilities (add back only NET_RAW), PID limit of 50, 10MB noexec tmpfs, `no-new-privileges` — all match the OWASP Docker Security Cheat Sheet recommendations.

### On-Demand Container Lifecycle
The idle cleanup pattern (10-minute stop, 24-hour remove) is well-designed. The three-tier state model (Running → Stopped → Removed) with preserved volumes is operationally sound and cost-efficient.

### Feature Prioritization
Email > Calendar > Daily Brief > Tasks matches real-world AI assistant adoption patterns validated by OpenClaw community use cases. The MVP scope (Features 1-4 + 6) is tight and deliverable.

### Supervised Mode as Default
Critical for both user safety and App Store approval. The explicit approve/reject action cards for agent actions are well-designed and address the biggest trust concern with autonomous AI agents.

### Composio for Integration Management (Architecture)
While the pricing needs investigation (Issue #3), the architectural choice of using a managed integration platform with SOC 2 compliance is correct. Building and maintaining 250+ OAuth integrations in-house would be impractical for a startup.

### Deepgram + ElevenLabs for Voice (Architecture)
The provider choices are correct — Deepgram leads in STT latency and pricing, ElevenLabs leads in voice naturalness. The combination is standard in the industry. Only the latency target needs adjustment (Issue #9).

---

## SUMMARY TABLE

| # | Issue | Severity | Document(s) | Section to Update |
|---|-------|----------|-------------|-------------------|
| 1 | Supabase Edge Function 2s CPU limit | **CRITICAL** | architecture.md | Section 8 (Request Flow) |
| 2 | ZeroClaw too new/unproven | **CRITICAL** | prd.md, architecture.md | Tech Stack, Risks |
| 3 | Composio pricing breaks unit economics | **CRITICAL** | prd.md | Section 8 (Pricing) |
| 4 | Apple App Store AI consent rules | **CRITICAL** | prd.md, featurespec.md | Risks, F-100 (Onboarding) |
| 5 | Expo SDK 54 + NativeWind compat issues | **HIGH** | prd.md | Section 5 (Tech Stack) |
| 6 | MiniMax M2.5 weak at reasoning tasks | **HIGH** | prd.md | Tech Stack, Risks |
| 7 | Docker Swarm is a dead-end | **HIGH** | architecture.md | Section 7 (Scaling) |
| 8 | Scrapling legal/reliability risk | **HIGH** | prd.md, featurespec.md | Risks, F-400 |
| 9 | Voice <2s latency unrealistic | **HIGH** | featurespec.md | F-005, Section 17.1 |
| 10 | SQLite vector search limitations | **MEDIUM** | architecture.md | Section 3 (Data Volume) |
| 11 | Hetzner price increases | **MEDIUM** | prd.md, architecture.md | Cost tables |
| 12 | $30/mo margin risk from heavy users | **MEDIUM** | prd.md | Section 8 (Pricing) |
| 13 | Container security (gVisor) | **MEDIUM** | architecture.md | Section 5 (Security) |
| 14 | GDPR DPAs missing | **MEDIUM** | prd.md, featurespec.md | Security, Compliance |

---

## RECOMMENDED ACTION ITEMS (Priority Order)

1. **Immediately:** Contact Composio for enterprise pricing and apply for startup program
2. **Immediately:** Run independent ZeroClaw benchmarks under realistic workloads
3. **This week:** Restructure Edge Function to async fire-and-forget pattern
4. **This week:** Add AI data consent screen to onboarding flow design
5. **This week:** Pin NativeWind and Tailwind CSS versions, enable New Architecture
6. **Before MVP:** A/B test MiniMax M2.5 vs alternatives for reasoning-heavy tasks
7. **Before MVP:** Update all Hetzner cost tables with current pricing
8. **Before MVP:** Add usage-based guardrails and per-user cost tracking
9. **Before MVP:** Tier the scraping approach (official APIs first, Scrapling as fallback)
10. **Before MVP:** Prepare Apple App Store AI disclosure documentation
11. **Phase 2:** Replace Docker Swarm scaling plan with k3s
12. **Phase 2:** Evaluate gVisor for container sandboxing
13. **Phase 2:** Implement GDPR DPAs with all third-party processors
14. **Phase 2:** Adjust voice latency targets based on real-world testing

---

*— End of Technical Review —*
