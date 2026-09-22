# Technical Stack

## Idea-First AI-Native Social Network

**Target:** Web-first social platform
**Initial cost target:** ₹0/month
**Architecture goal:** Production-oriented from day one without paying for infrastructure
**Primary principle:** Keep every expensive or intelligent subsystem replaceable.

---

# 1. Recommended Stack at a Glance

| Layer                 | Technology                               | Purpose                                   |
| --------------------- | ---------------------------------------- | ----------------------------------------- |
| Language              | TypeScript                               | Entire application                        |
| Frontend              | React + Next.js 16                       | Web application                           |
| Styling               | Tailwind CSS                             | UI styling                                |
| Components            | Radix/shadcn-style components            | Accessible UI primitives                  |
| Package manager       | pnpm                                     | Dependency management                     |
| Runtime/hosting       | Cloudflare Workers                       | App/server runtime                        |
| Next.js deployment    | OpenNext initially                       | Run Next.js on Workers                    |
| Database              | Supabase PostgreSQL                      | Primary source of truth                   |
| ORM/data access       | Supabase JS + SQL/RPC                    | Simple Worker-compatible DB access        |
| Auth                  | Supabase Auth                            | Google/GitHub/email                       |
| Authorization         | PostgreSQL RLS                           | Database security                         |
| Vector search         | pgvector                                 | Similar Minds/search                      |
| File storage          | Cloudflare R2                            | Images/media                              |
| CDN                   | Cloudflare                               | Static/media delivery                     |
| Bot protection        | Cloudflare Turnstile                     | Signup/post abuse                         |
| Rate limiting         | Cloudflare Rate Limiting API             | Posting/API abuse                         |
| Async jobs            | Cloudflare Queues                        | AI analysis/background jobs               |
| Scheduled jobs        | Cloudflare Cron Triggers                 | Daily rewards/indexing                    |
| Stateful coordination | Durable Objects                          | Future rate-limit/live state              |
| Cache                 | Cloudflare Cache + KV selectively        | Hot feeds/results                         |
| AI inference          | Workers AI initially                     | Small server-side models                  |
| Local AI              | Ollama / llama.cpp                       | Development + heavy batch experimentation |
| Embeddings            | pgvector + embedding model               | Interest graph/similarity                 |
| Analytics             | Workers Analytics Engine + DB aggregates | Behavioral/product metrics                |
| Testing               | Vitest + Playwright                      | Unit/integration/E2E                      |
| Validation            | Zod                                      | API/input validation                      |
| Linting               | ESLint                                   | Code quality                              |
| Formatting            | Prettier                                 | Consistency                               |
| CI                    | GitHub Actions                           | Test/build/deploy                         |
| Git                   | GitHub                                   | Source control                            |
| PWA                   | Web App Manifest + Service Worker        | Installable mobile web app                |

---

# 2. Architecture

The application should look approximately like:

```text
                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │   CLOUDFLARE  │
                    │ DNS / CDN /    │
                    │ Turnstile      │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ CF WORKERS    │
                    │ Next.js       │
                    │ OpenNext      │
                    └───────┬───────┘
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
       SUPABASE         CLOUDFLARE      AI LAYER
       POSTGRES            R2           Workers AI
          │                 │               │
          │                 │          ┌────┴─────┐
          │                 │          │ Queues   │
          │                 │          └────┬─────┘
          │                 │               │
          ▼                 ▼               ▼
       pgvector          Media        AI analysis
                                         │
                              ┌──────────┼──────────┐
                              ▼          ▼          ▼
                          Content     Behavior   Provenance
```

The important rule:

> **Supabase/Postgres is the source of truth.**

Cloudflare cache, KV, Workers AI, queues and derived AI outputs are supporting systems.

---

# 3. Frontend

## React

Use React for:

* post composer
* feed
* comments
* reactions
* profile
* discovery
* admin dashboard
* real-time UI

No need for a second frontend framework.

---

# 4. Next.js

Use:

**Next.js 16**

with the App Router.

Use Next.js for:

* SSR
* public SEO pages
* profiles
* posts
* entity pages
* topic pages
* server-side rendering
* metadata
* route handlers
* server-side mutations

The public content pages are important because:

```text
example.com/post/123
example.com/entity/openai
example.com/topic/artificial-intelligence
example.com/@username
```

should be indexable.

---

# 5. Next.js + Cloudflare

Cloudflare's current documentation recommends **vinext** for new Next.js applications on Workers, while OpenNext remains supported and handles most Next.js features. However, Cloudflare currently labels vinext as beta, so for your first serious build I would keep the application as standard Next.js and use the OpenNext adapter initially, with the deployment adapter isolated so you can move to vinext later if its compatibility is right for the project.

Recommended:

```text
Next.js 16
      ↓
OpenNext
      ↓
Cloudflare Workers
```

Migration path:

```text
OpenNext
   ↓
vinext
```

without changing the application architecture.

---

# 6. TypeScript

Everything should be TypeScript.

Use strict mode:

```text
"strict": true
```

No JavaScript files in application code unless a tooling constraint requires one.

TypeScript should cover:

* frontend
* server
* database types
* AI interfaces
* moderation decisions
* telemetry
* API contracts

---

# 7. Package Manager

Use:

```text
pnpm
```

Advantages:

* fast
* disk efficient
* deterministic lockfile
* workspace support
* good monorepo path later

Initial project does not need a complicated monorepo.

---

# 8. UI

Use:

```text
Tailwind CSS
+
Radix primitives / shadcn-style components
```

Keep the visual design custom and minimal.

Avoid installing a massive UI framework.

The core product should feel:

* clean
* content-first
* fast
* readable
* mobile-friendly

---

# 9. Client State

Don't immediately introduce Redux.

Use:

### Server state

Next.js Server Components where possible.

### Interactive data

TanStack Query can be used for:

* feed pagination
* comments
* reactions
* notifications
* optimistic updates
* background refresh

### Local UI state

React state/hooks.

This keeps the state architecture simple.

---

# 10. Database

Use:

# PostgreSQL

through Supabase.

Postgres is the right foundation because your product has relationships everywhere:

```text
users
posts
comments
reactions
topics
entities
followers
interest graphs
reputation
rewards
provenance
moderation
```

A document database would make several of these relationships unnecessarily difficult.

---

# 11. Supabase

Use Supabase for:

* PostgreSQL
* Auth
* Row Level Security
* basic Storage where appropriate
* Realtime where useful
* Edge Functions only where they provide clear value

Supabase Auth integrates with PostgreSQL and RLS, making it suitable for this architecture.

The current free tier gives enough room for the MVP: 500 MB database, 1 GB storage, 5 GB egress, 50k MAU, 500k Edge Function invocations and 2M Realtime messages.

---

# 12. Database Access

Do not introduce a heavy ORM initially.

Use:

```text
Supabase JS
+
Postgres SQL
+
RPC/functions
+
typed generated database schema
```

Why?

Your application needs things like:

```text
feed ranking
reaction aggregation
daily reward selection
similarity queries
reputation calculations
provenance checks
```

Those are often easier to express directly in SQL.

---

# 13. RLS

PostgreSQL Row Level Security should be enabled from the beginning.

Examples:

### Users

A user can modify only their own profile.

### Posts

A user can edit only their own eligible posts.

### Reactions

A user can create/delete only their own reaction.

### Rewards

Users cannot directly modify points.

### Moderation

Only admin/service role can modify moderation decisions.

### Provenance

Users cannot modify historical provenance records.

This is critical.

---

# 14. pgvector

Use PostgreSQL's `pgvector` extension for embeddings and similarity search.

Supabase officially supports pgvector for storing embeddings and vector similarity queries.

This powers:

* Similar Minds
* Different Perspectives
* semantic search
* duplicate detection
* related posts
* related entities
* topic clustering

---

# 15. Vector Data

Store embeddings for:

### Posts

```text
post_embedding
```

### Users

Derived from their content/interaction profile.

```text
user_interest_embedding
```

### Topics

```text
topic_embedding
```

### Entities

```text
entity_embedding
```

Then:

```text
user A
   ↓
embedding
   ↓
nearest users
```

gives Similar Minds.

---

# 16. Media Storage

Do not put large media files into Postgres.

Use:

# Cloudflare R2

for:

* images
* avatars
* thumbnails
* eventually video

R2 currently includes 10 GB-month storage, 1 million Class A operations, 10 million Class B operations and free Internet egress in its free tier.

That makes it better suited than using your tiny Postgres storage quota for user media.

---

# 17. Supabase Storage

Keep Supabase Storage available for:

* very small files
* private user files
* temporary application files

But make R2 your intended long-term media layer.

Supabase's current free storage allowance is 1 GB.

---

# 18. Images

For MVP:

```text
Browser
   ↓
compress/resize
   ↓
R2
   ↓
Cloudflare CDN
```

Do not upload a 10 MB original phone photo if a 200–500 KB display image is sufficient.

Store:

```text
original
thumbnail
medium
```

only when needed.

---

# 19. Video

Do not build a full video pipeline in the initial version.

Initially allow:

* external video links
* small user videos only if necessary

Later introduce:

```text
R2
 ↓
queue
 ↓
transcoding worker
 ↓
multiple resolutions
 ↓
CDN
```

Video processing can become a significant cost later.

---

# 20. Authentication

Use:

# Supabase Auth

Support:

* Google
* GitHub
* email/password
* magic link

Supabase Auth supports social login, password login, magic links, OTP and other authentication mechanisms.

---

# 21. Session Handling

For Next.js:

```text
@supabase/ssr
```

Use secure cookie-based server/client session handling.

Never expose service-role keys to the browser.

---

# 22. Cloudflare Workers

Cloudflare Workers becomes your:

* edge server
* API layer
* middleware
* rate limiting layer
* cache layer
* AI gateway
* asynchronous job entrypoint

Cloudflare's Free plan currently provides 100,000 Worker requests/day.

That is more than enough for an early MVP.

---

# 23. Cloudflare Cache

Use edge caching for:

* public posts
* topic pages
* entity pages
* public profiles
* trending pages

Do not cache:

* private feeds
* private user data
* authentication responses
* moderation dashboards

unless keys are carefully controlled.

---

# 24. Cloudflare KV

Use KV sparingly.

Current Free limits are:

* 100,000 reads/day
* 1,000 writes/day
* 1 GB storage

which makes KV useful for relatively stable cached data, but not a primary database.

Good KV candidates:

```text
feature_flags
topic_metadata
hot_entity_cache
static configuration
expensive public summaries
```

Bad KV candidates:

```text
posts
comments
reactions
rewards
user profiles
```

Those belong in Postgres.

---

# 25. Rate Limiting

Use Cloudflare's Rate Limiting API.

Create separate buckets:

```text
login
signup
post
comment
reaction
follow
search
API
```

Example:

```text
POST /api/posts
```

has stricter limits than:

```text
GET /api/post/123
```

Cloudflare exposes a Worker-side Rate Limiting API specifically for route/resource-specific limits.

---

# 26. Turnstile

Use:

# Cloudflare Turnstile

for:

* signup
* suspicious login
* suspicious post activity
* bot challenges
* abuse escalation

Do not show CAPTCHA on every action.

It should be triggered by risk.

---

# 27. Queue System

Use:

# Cloudflare Queues

for anything that does not need to happen synchronously.

Cloudflare currently includes 10,000 queue operations/day on Workers Free.

When a user posts:

```text
POST
 ↓
save immediately
 ↓
queue analysis job
```

not:

```text
POST
 ↓
wait for 5 AI models
 ↓
wait for embeddings
 ↓
wait for moderation
 ↓
publish
```

The post should appear quickly.

---

# 28. Queue Types

Create logical queues:

```text
content-analysis
embedding-analysis
provenance-analysis
moderation-analysis
reward-calculation
notification-jobs
feed-maintenance
```

Initially these can share infrastructure.

---

# 29. Post Processing Pipeline

The real flow should be:

```text
USER POSTS
    │
    ▼
Validate
    │
    ▼
Write Post to Postgres
    │
    ▼
Generate content hash
    │
    ▼
Return success
    │
    └──────────────► Queue
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
        Content AI   Behavior AI   Provenance
             │           │           │
             └───────────┼───────────┘
                         ▼
                    Analysis DB
                         │
                         ▼
                    Feed Ranking
```

This gives you low latency.

---

# 30. AI Layer

The AI layer should NOT be tightly coupled to the web app.

Create an internal interface:

```ts
interface ContentAnalyzer {
  analyze(input: AnalysisInput): Promise<AnalysisResult>;
}
```

Then create implementations:

```text
RulesContentAnalyzer
WorkersAIContentAnalyzer
LocalModelContentAnalyzer
FutureModelAnalyzer
```

The app doesn't care which one is being used.

---

# 31. Workers AI

Use Cloudflare Workers AI for initial low-volume server-side inference.

Cloudflare currently offers 50+ open-source models through Workers AI and a free allocation of 10,000 neurons/day; usage beyond that requires the paid plan.

Therefore:

> **Do not send every operation through an LLM.**

Use AI strategically.

---

# 32. AI Workload Split

### Rules

Use for:

* rate limits
* paste detection
* post frequency
* obvious spam
* account abuse
* exact duplicates

### Lightweight ML

Use for:

* topic classification
* personal-life detection
* basic intent
* sentiment/context

### LLM

Use only where reasoning is useful:

* claim interpretation
* complex classification
* source relevance
* discussion summarization
* entity disambiguation

### Embedding model

Use for:

* semantic similarity
* Similar Minds
* semantic search
* duplicate detection

This dramatically reduces inference cost.

---

# 33. Local AI Development

For development:

```text
Ollama
or
llama.cpp
```

Run models locally.

Your local machine can act as the model experimentation environment.

Example:

```text
Next.js
      ↓
AI interface
      ↓
local Ollama
```

Then change one environment variable:

```text
AI_PROVIDER=workers
```

for production.

---

# 34. Why Local AI Matters

You can train/evaluate:

* AI-writing detector
* topic classifier
* personal-life classifier
* similarity models
* ranking models

without paying an API provider every time you test.

---

# 35. AI Service Interface

Use something like:

```text
src/lib/ai/
    provider.ts
    types.ts

    content/
        analyzer.ts

    moderation/
        analyzer.ts

    embeddings/
        provider.ts

    similarity/
        engine.ts

    risk/
        detector.ts
```

The rest of the app should never directly call a model.

---

# 36. Content Analysis Result

Standardize all AI output:

```ts
type ContentAnalysis = {
  topics: TopicScore[];
  entities: EntityMatch[];
  intent: ContentIntent;
  personalLifeScore: number;
  factualClaimScore: number;
  discussionScore: number;
  helpfulnessScore: number;
  originalityScore: number;
  safetyFlags: SafetyFlag[];
  confidence: number;
  modelVersion: string;
};
```

Then models can be replaced later.

---

# 37. Behavioral AI

This system is separate from text analysis.

```text
BehaviorAnalyzer
```

Input:

```text
composition time
active typing time
pause distribution
edit ratio
paste events
publication velocity
account history
interaction behavior
```

Output:

```text
aiGenerationRisk
automationRisk
spamRisk
confidence
```

---

# 38. Do Not Send Raw Keystrokes

The browser should aggregate events locally.

For example:

```text
1000 keystroke events
       ↓
client-side feature extraction
       ↓
20 statistical features
       ↓
server
```

This improves:

* privacy
* bandwidth
* storage
* security

---

# 39. Composition Telemetry

Client-side collector:

```text
useCompositionTelemetry()
```

Tracks:

```text
sessionStart
activeTime
pauseCount
pauseDurations
pasteCount
pasteChars
deleteCount
replacementCount
undoCount
redoCount
characterGrowth
focusLoss
focusReturn
publishTime
```

Send only summarized features.

---

# 40. Event Schema

Example:

```ts
type CompositionFeatures = {
  sessionId: string;
  totalMs: number;
  activeMs: number;
  chars: number;
  pasteCount: number;
  pasteChars: number;
  deleteCount: number;
  replacementCount: number;
  pauseCount: number;
  meanPauseMs: number;
  maxPauseMs: number;
  editingRatio: number;
  publicationDelayMs: number;
};
```

---

# 41. AI-Generation Detection Architecture

Use several levels.

```text
Level 1:
behavioral rules

Level 2:
account behavior

Level 3:
text similarity

Level 4:
statistical/ML detector

Level 5:
provider-specific provenance when available
```

Combine:

```text
risk = ensemble(signals)
```

Never:

```text
AI detector says AI = true
therefore ban
```

---

# 42. Embeddings

Use embeddings for:

### Duplicate detection

```text
new post
 ↓
embedding
 ↓
nearest posts
```

### Similar Minds

```text
user profile
 ↓
embedding
 ↓
nearest users
```

### Different Perspectives

```text
shared interests
+
different semantic/opinion patterns
```

### Semantic search

```text
"people using local AI models"
```

rather than exact keyword matching.

---

# 43. Interest Profile Calculation

Do not simply average all posts equally.

A user's interest vector should consider:

```text
post frequency
reading time
saves
Helpful
comments
following
repeated topics
reactions
```

Negative feedback should also matter.

Example:

```text
user sees 100 AI posts
saves 20
comments on 8
ignores finance

AI interest rises
Finance interest falls
```

---

# 44. Similar Minds Algorithm

Start simple.

```text
similarity =
    50% topic overlap
  + 20% entity overlap
  + 15% reading overlap
  + 10% interaction overlap
  +  5% working-interest overlap
```

These weights are placeholders.

Later learn them from actual user behavior.

---

# 45. Different Perspectives

This should NOT mean:

> Pick someone who disagrees with everything.

Instead:

```text
high shared topic interest
+
different opinion patterns
+
constructive interaction
=
Different Perspective
```

This prevents the system from recommending rage accounts merely because they disagree.

---

# 46. Feed Ranking

Initial feed formula:

```text
score =
    relevance
  + freshness
  + helpfulness
  + discussion_quality
  + originality
  + save_rate
  + share_rate
  + source_quality
  + author_reputation
  - spam_risk
  - automation_risk
  - duplicate_risk
```

Do not over-engineer this initially.

Use a deterministic scoring function first.

---

# 47. Learned Ranking Later

When enough data exists:

```text
LightGBM/XGBoost
```

or another ranking model can learn:

```text
probability of useful engagement
```

rather than:

```text
probability of click
```

That distinction is important.

---

# 48. Recommendation Engine

Create:

```text
RecommendationEngine
```

It handles:

* posts
* topics
* people
* entities
* discussions

It should output reasons internally:

```text
similar_interest
similar_project
related_topic
different_perspective
```

The UI can eventually say:

> "You both frequently discuss local AI."

---

# 49. Reactions

Database:

```text
reaction_type:
  helpful
  not_helpful
  more_detail
  disagree
  curious
```

One user gets one active reaction of each type per content item, unless product rules change later.

---

# 50. Reaction Analytics

Store aggregate counts separately when scale becomes necessary.

Initially SQL can calculate:

```text
count(*)
group by reaction_type
```

Later:

```text
post_reaction_counts
```

can be maintained asynchronously.

---

# 51. Save / Bookmark

Separate table:

```text
saved_posts
```

Use saves heavily for:

* feed ranking
* user interest inference
* "Your saved ideas"
* helpfulness

---

# 52. Daily Reward System

Use a background job.

Every day:

```text
00:00 UTC
   ↓
collect user's eligible posts
   ↓
calculate contribution scores
   ↓
choose top contribution
   ↓
write daily_contribution
   ↓
award points
```

This can run using Cloudflare Cron + Queue.

---

# 53. Daily Reward Does NOT Mean Daily Posting Requirement

Users should never feel:

> "I must post every day."

They can receive:

* contribution points
* zero points
* no penalty

depending on whether they contributed something meaningful.

This keeps the system aligned with quality.

---

# 54. Reward Ledger

Never store only:

```text
user.points = 5120
```

Instead maintain:

```text
reward_ledger
```

with immutable transactions:

```text
+50 daily contribution
-500 reward redemption
+20 correction
```

Then calculate balance.

This makes fraud investigation possible.

---

# 55. Entity System

Use an `entities` table:

```text
id
name
slug
type
description
canonical_url
embedding
created_at
```

Types:

```text
company
product
software
website
book
movie
game
research
person
organization
technology
```

---

# 56. Automatic Entity Resolution

When a post says:

> Cursor

the AI needs to determine:

```text
Cursor IDE
```

rather than another similarly named entity.

Store:

```text
entity_aliases
```

and embedding/search metadata.

---

# 57. Entity Pages

Dynamic Next.js pages:

```text
/entity/[slug]
```

Sections:

```text
Overview
Recent posts
Reviews
Questions
Discussions
Trending opinions
```

---

# 58. Review System

Reviews do not need separate UI initially.

Treat a review as normal content with inferred entity association.

Example:

```text
POST
 ↓
AI detects
 ↓
Entity = Cursor
Intent = Review
Experience = Long-term
```

The same post appears in the user's feed and the Cursor entity page.

---

# 59. Provenance

At publication:

```text
normalized_content
+
media hashes
+
creator ID
+
timestamp
```

produce:

```text
content_hash
```

Use:

```text
SHA-256
```

---

# 60. Internal Hash Chain

Each content record can reference:

```text
previous_provenance_hash
```

Example:

```text
post A
hash A

post B
hash B + hash A

post C
hash C + hash B
```

This creates a tamper-evident chain.

---

# 61. Public Timestamping

Later:

```text
daily post hashes
 ↓
Merkle tree
 ↓
Merkle root
 ↓
external timestamp
```

Potential technology:

```text
OpenTimestamps
```

This avoids putting every post directly onto a blockchain.

---

# 62. Why Not Ethereum/Polygon/etc. Initially?

Because the user should not need:

* wallet
* gas
* token
* blockchain account

The blockchain should eventually verify provenance invisibly.

---

# 63. Search

Initial:

```text
Postgres full-text search
```

with:

```text
tsvector
GIN indexes
```

Later:

```text
semantic search
+
pgvector
```

Combined search:

```text
keyword relevance
+
semantic similarity
```

---

# 64. Notifications

Use database records initially:

```text
notifications
```

Types:

```text
helpful
more_detail
disagree
comment
share
follow
similar_mind
discussion
company_response
```

For future real-time delivery, Supabase Realtime can handle it; the current free tier includes 2M messages and 200 peak connections.

---

# 65. Real-Time

Do NOT make the entire feed real-time.

Real-time is needed for:

* notification count
* comment updates
* live discussion additions
* moderation status
* possibly collaborative/admin views

The main feed can use normal HTTP + caching.

---

# 66. Analytics

Use two levels.

### Product analytics

Postgres aggregates:

```text
daily_active_users
posts
comments
reactions
retention
```

### High-volume behavioral telemetry

Cloudflare Analytics Engine.

Cloudflare currently lists 100,000 data points/day and 10,000 read queries/day on the Workers Free plan, with pricing documented for later billing.

Use it for aggregated metrics, not source-of-truth application data.

---

# 67. What Goes Into Postgres

Keep important permanent data:

* users
* posts
* comments
* reactions
* follows
* topics
* entities
* moderation decisions
* rewards
* provenance
* model decisions
* interest profiles

---

# 68. What Goes Into Analytics Engine

Use for:

* composition feature distributions
* average typing duration
* posting velocity distributions
* model latency
* risk-score distributions
* feed impressions
* aggregate clicks
* system health

---

# 69. Background Scheduling

Cloudflare supports Cron Triggers.

Use them for:

```text
hourly trend calculations
nightly reward selection
daily interest graph updates
embedding maintenance
provenance batching
cleanup jobs
```

The Free Workers plan currently allows up to five Cron Triggers per account.

---

# 70. Durable Objects

Do not use Durable Objects for your entire application.

Use them later for things needing strongly coordinated state:

* live discussion rooms
* hot counters
* distributed locks
* advanced rate limiting
* real-time moderation sessions

Cloudflare currently provides SQLite-backed Durable Objects on the Free plan with 100,000 requests/day and 5 GB total free storage.

---

# 71. API Architecture

Use:

```text
/app
/api
```

with clear domain modules.

Example:

```text
POST   /api/posts
GET    /api/posts/:id
DELETE /api/posts/:id

POST   /api/posts/:id/comments
POST   /api/posts/:id/reactions

POST   /api/users/:id/follow
GET    /api/users/:id/similar

GET    /api/feed
GET    /api/trending
GET    /api/search
```

---

# 72. Internal APIs

AI should not be publicly exposed.

Examples:

```text
/internal/analyze-post
/internal/analyze-behavior
/internal/generate-embedding
/internal/calculate-reward
/internal/rebuild-interest-profile
```

These routes require internal authentication.

---

# 73. Validation

Use:

# Zod

for every external input.

Example:

```ts
PostCreateSchema
CommentCreateSchema
ReactionSchema
ProfileUpdateSchema
```

Never trust frontend validation.

---

# 74. Security

Core security mechanisms:

```text
Supabase RLS
+
server-side authorization
+
Turnstile
+
Cloudflare rate limiting
+
input validation
+
content sanitization
+
secure cookies
+
CSP
+
CSRF protection where applicable
```

---

# 75. Content Sanitization

Never render arbitrary HTML from user posts.

Store plain text/structured content.

If markdown is eventually supported:

```text
Markdown
 ↓
sanitize
 ↓
safe HTML
```

No raw HTML.

---

# 76. Links

When users post URLs:

```text
URL
 ↓
validate
 ↓
normalize
 ↓
safe fetch
 ↓
metadata extraction
```

Do not allow arbitrary server-side fetching without SSRF protection.

---

# 77. Moderation Data Model

Have separate tables:

```text
moderation_events
moderation_scores
user_risk_profiles
content_flags
admin_actions
appeals
```

Do not overwrite history.

You need to know:

> Why did the platform make this decision?

---

# 78. Model Versioning

Every AI result should record:

```text
model_name
model_version
prompt_version
rules_version
feature_version
timestamp
```

Example:

```text
content_model = topic-v3
risk_model = behavior-v2
prompt = moderation-17
```

This will become extremely important when false positives appear.

---

# 79. AI Evaluation Dataset

Maintain a private evaluation dataset containing:

```text
human-authored
AI-generated
AI-assisted
pasted
rewritten
paraphrased
mass-generated
spam
legitimate fast typists
```

This dataset should be versioned.

---

# 80. AI Detector Evaluation

Metrics:

```text
precision
recall
F1
false-positive rate
false-negative rate
ROC-AUC
calibration
```

But importantly:

## Track false positives separately by user behavior class.

For example:

```text
fast typists
mobile users
accessibility users
voice-to-text users
copy editors
```

A system that wrongly penalizes these users would be unacceptable.

---

# 81. Voice Input

Important:

Someone may dictate a post using phone voice input.

Their typing telemetry becomes:

```text
almost zero typing
large text insertion
```

That looks like AI-generated text.

Therefore:

> **Typing behavior must never be treated as definitive evidence.**

The composer should detect legitimate input modes where possible:

```text
typing
speech-to-text
IME/composition
paste
drag/drop
```

and the risk system must account for them.

---

# 82. Mobile Keyboard Behavior

Mobile keyboards also behave differently from desktop keyboards.

Do not use one universal threshold.

Maintain baselines by:

```text
device class
input method
browser
```

---

# 83. Accessibility

The anti-AI system must not punish:

* screen readers
* voice dictation
* alternative keyboards
* accessibility tools
* users who compose slowly
* users who type extremely quickly

This is a core technical requirement.

---

# 84. Feed Cache

Use a layered model:

```text
Postgres
   ↓
Feed generation
   ↓
Cloudflare cache
   ↓
Browser
```

For personalized feeds:

```text
user-specific query
```

For Trending:

```text
precomputed
+
cached
```

For entity pages:

```text
cached
```

---

# 85. Feed Pagination

Use cursor pagination.

Do NOT use:

```text
?page=50
```

for an infinite social feed.

Use:

```text
?cursor=<opaque-token>
```

based on:

```text
rank
created_at
post_id
```

This performs much better as the database grows.

---

# 86. Database Indexes

At minimum:

```text
posts(created_at)
posts(user_id, created_at)
posts(status, created_at)
comments(post_id, created_at)
reactions(post_id, reaction_type)
follows(follower_id)
follows(following_id)
post_topics(post_id, topic_id)
post_entities(post_id, entity_id)
```

Add vector indexes after enough embeddings exist.

---

# 87. Database Partitioning

Do NOT partition everything initially.

When scale requires it, the largest candidates are:

```text
analytics_events
composition_features
moderation_events
notifications
```

Core social data should stay simple until actual scale demands more complexity.

---

# 88. Testing Stack

Use:

### Unit

```text
Vitest
```

Test:

* scoring
* classification adapters
* reward selection
* hash generation
* ranking
* validation

### E2E

```text
Playwright
```

Test:

* signup
* posting
* reactions
* comments
* following
* daily rewards
* moderation

### Type checking

```text
tsc --noEmit
```

---

# 89. CI

GitHub Actions pipeline:

```text
install
 ↓
lint
 ↓
typecheck
 ↓
unit tests
 ↓
build
 ↓
Playwright
 ↓
deploy
```

Use separate staging and production environments.

---

# 90. Environment Variables

Use:

```text
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY

R2_ACCOUNT_ID
R2_ACCESS_KEY_ID
R2_SECRET_ACCESS_KEY
R2_BUCKET

TURNSTILE_SITE_KEY
TURNSTILE_SECRET_KEY

WORKERS_AI_MODEL
AI_PROVIDER

APP_URL
```

Secrets never enter Git.

---

# 91. Environments

### Local

```text
Next.js
Supabase dev
Ollama
local mocks
```

### Staging

```text
Cloudflare
Supabase staging
Workers AI
R2 staging bucket
```

### Production

```text
Cloudflare
Supabase production
R2 production
Workers AI
```

Use separate databases for staging and production.

---

# 92. Repository Structure

I would start with **one repository**, not a massive monorepo.

```text
idea-network/
│
├── app/
│   ├── (public)/
│   ├── (auth)/
│   ├── home/
│   ├── post/
│   ├── topic/
│   ├── entity/
│   ├── profile/
│   ├── settings/
│   └── admin/
│
├── components/
│   ├── composer/
│   ├── post/
│   ├── feed/
│   ├── comments/
│   ├── reactions/
│   ├── profile/
│   └── discovery/
│
├── lib/
│   ├── supabase/
│   ├── cloudflare/
│   ├── auth/
│   ├── db/
│   ├── ai/
│   ├── moderation/
│   ├── provenance/
│   ├── ranking/
│   ├── rewards/
│   ├── similarity/
│   └── telemetry/
│
├── server/
│   ├── posts/
│   ├── feed/
│   ├── moderation/
│   ├── rewards/
│   └── recommendations/
│
├── database/
│   ├── migrations/
│   ├── functions/
│   └── seeds/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── public/
│
├── workers/
│   ├── queues/
│   ├── cron/
│   └── ai/
│
├── scripts/
│
├── wrangler.toml
├── next.config.ts
├── package.json
├── pnpm-lock.yaml
└── tsconfig.json
```

---

# 93. Why Not Microservices?

Do not start with:

```text
auth-service
post-service
feed-service
AI-service
notification-service
```

That is unnecessary complexity for your stage.

Start with a modular monolith.

The boundaries are logical, not physical.

---

# 94. AI Worker Separation

When AI workload grows, split it physically:

```text
web application
        │
        ▼
queue
        │
        ▼
AI worker
```

The web app remains lightweight.

This also means a failed AI worker does not prevent users from posting.

---

# 95. Failure Strategy

AI failure must never mean:

> Website doesn't work.

If AI is unavailable:

```text
User posts
 ↓
Basic moderation
 ↓
Post published
 ↓
AI analysis queued
```

The system processes it later.

This is extremely important.

---

# 96. Graceful AI Degradation

If Workers AI quota is exhausted:

```text
AI unavailable
     ↓
fallback rules
     ↓
queue for later
```

If embeddings unavailable:

```text
keyword/topic similarity
```

If advanced moderation unavailable:

```text
basic moderation + report
```

Never make one AI service a hard dependency for posting.

---

# 97. Observability

Every request should have:

```text
request_id
user_id
route
latency
status
```

For AI:

```text
analysis_id
model
model_version
latency
success
failure
```

For queues:

```text
job_id
queue
attempt
status
duration
```

---

# 98. Error Tracking

Initially:

```text
Cloudflare logs
+
structured application logs
```

Later add an error-monitoring provider if necessary.

Don't add ten observability products before the platform has users.

---

# 99. Cost Architecture

Initial target:

```text
Cloudflare Workers          ₹0
Cloudflare R2               ₹0
Cloudflare Turnstile        ₹0
Cloudflare Queues           ₹0
Supabase                    ₹0
GitHub                      ₹0
Workers AI                  ₹0 within free allocation
```

Supabase's current free tier is $0 and provides the database/Auth/storage capabilities noted above.

Cloudflare Workers AI currently provides 10,000 free neurons/day.

Cloudflare Queues currently provides 10,000 free operations/day.

R2 currently provides 10 GB-month plus free egress in its free tier.

---

# 100. Important Free-Tier Constraint

Do not architect around:

> "I can run an LLM for every post forever for free."

You cannot assume that.

The correct design is:

```text
cheap deterministic processing
        +
small model analysis
        +
async queue
        +
local experimentation
```

AI usage can then scale independently when the project eventually earns money.

---

# 101. AI Inference Strategy

## Level 0

No model:

```text
rules
regex
statistics
SQL
```

## Level 1

Small classifier:

```text
topic
intent
personal-life
spam
```

## Level 2

Embedding:

```text
similarity
search
interest graph
```

## Level 3

LLM:

```text
complex moderation
claim analysis
summaries
```

This layered approach is the key to maintaining ₹0 cost initially.

---

# 102. Local Development AI

Use:

```text
Ollama
```

or:

```text
llama.cpp
```

for development.

Model candidates can change over time.

The important abstraction is:

```text
AIProvider
```

not the particular model.

---

# 103. AI Provider Interface

Example:

```ts
interface AIProvider {
  classify(input: string): Promise<ClassificationResult>;
  embed(input: string): Promise<number[]>;
  moderate(input: string): Promise<ModerationResult>;
}
```

Implement:

```text
WorkersAIProvider
LocalAIProvider
MockAIProvider
```

---

# 104. Testing AI

During development:

```text
MockAIProvider
```

should return deterministic data.

This allows normal application tests to run without consuming AI inference.

---

# 105. Model Registry

Maintain:

```text
ai_models
```

with:

```text
name
provider
version
task
active
fallback
```

Example:

```text
topic-classifier-v1
behavior-risk-v1
embedding-v1
moderator-v1
```

---

# 106. Feature Flags

Use environment/database feature flags:

```text
ENABLE_AI_ANALYSIS
ENABLE_AI_RISK_SCORING
ENABLE_EMBEDDINGS
ENABLE_SIMILAR_MINDS
ENABLE_PROVENANCE
ENABLE_DAILY_REWARDS
ENABLE_REALTIME
```

This makes new subsystems deployable independently.

---

# 107. What Should Be Permanent From Day One

Even in the first prototype, build the interfaces for:

```text
ContentAnalyzer
BehaviorAnalyzer
AIRiskDetector
SimilarityEngine
ProvenanceEngine
RecommendationEngine
RewardEngine
ModerationEngine
```

Their first implementations can be extremely simple.

This is where you prevent future rewrites.

---

# 108. What Can Be Simplified Initially

Safe to keep simple:

```text
feed ranking
entity resolution
AI classification
embeddings
company review verification
blockchain anchoring
advanced moderation
reward formula
```

Do not simplify the database boundaries or security model.

---

# 109. Production Architecture Eventually

The long-term architecture can evolve into:

```text
                         CLOUDFLARE
                             │
                    ┌────────┴────────┐
                    │                 │
                 WEB APP            CACHE
                    │
                    ▼
                API LAYER
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
       POSTGRES   R2      QUEUES
          │                   │
          │             ┌─────┼─────┐
          │             ▼     ▼     ▼
          │           AI    RANK  INDEX
          │
          ▼
       PGVECTOR
          │
     ┌────┴────────┐
     ▼             ▼
SIMILAR MINDS   SEARCH
```

---

# 110. The Most Important Architectural Decision

The entire system should treat the following as separate layers:

```text
CONTENT
BEHAVIOR
AI ANALYSIS
PROVENANCE
REPUTATION
RECOMMENDATION
REWARD
```

Do not create one giant:

```text
"AI moderator"
```

class that does everything.

Instead:

```text
AI Moderator
    ├── ContentAnalyzer
    ├── BehaviorAnalyzer
    ├── SpamDetector
    ├── SafetyAnalyzer
    ├── ProvenanceAnalyzer
    └── DecisionEngine
```

This will make the platform much easier to evolve.

---

# 111. Final Recommended Stack

## Frontend

```text
Next.js 16
React
TypeScript
Tailwind CSS
Radix/shadcn-style components
TanStack Query
PWA
```

## Backend

```text
Cloudflare Workers
OpenNext initially
Cloudflare Cache
Cloudflare Rate Limiting
Cloudflare Turnstile
Cloudflare Queues
Cloudflare Cron
```

## Database

```text
Supabase
PostgreSQL
RLS
pgvector
Supabase Auth
```

## Storage

```text
Cloudflare R2
```

## AI

```text
Workers AI
Ollama
llama.cpp
provider abstraction
```

## AI intelligence

```text
rules
+
lightweight classifiers
+
embeddings
+
LLM reasoning
```

## Data processing

```text
Queues
Cron
Analytics Engine
```

## Testing

```text
Vitest
Playwright
TypeScript
ESLint
Prettier
```

## Deployment

```text
GitHub
   ↓
GitHub Actions
   ↓
Cloudflare Workers
```

---

# 112. Stack Philosophy

The platform should have:

### Cheap permanent infrastructure

```text
Cloudflare
Supabase
Postgres
R2
```

### Replaceable intelligence

```text
AI Provider
Embedding Provider
Ranking Model
Moderation Model
```

### Permanent application interfaces

```text
AI
Moderation
Provenance
Similarity
Rewards
Ranking
```

### Simple user experience

```text
One composer
One feed
Automatic understanding
```

---

# 113. Final Architecture Principle

The most important rule for the codebase is:

> **Never let today's free model become tomorrow's architectural dependency.**

Your user-facing application should not know whether content was classified by:

* a rule
* a small local model
* Workers AI
* an open-source model
* a future commercial model

It should only receive:

```text
ContentAnalysis
ModerationResult
SimilarityResult
RecommendationResult
RewardResult
```

That gives you the freedom to build the entire first version for effectively zero infrastructure cost while keeping the architecture ready for a much larger social network.
