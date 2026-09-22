# Full Application Architecture & Database Schema

## Idea-First AI-Native Social Network

**Version:** 1.0
**Architecture:** Modular monolith + asynchronous AI workers
**Database:** PostgreSQL / Supabase
**Frontend:** Next.js + React + TypeScript
**Edge/API:** Cloudflare Workers
**Storage:** Cloudflare R2
**Vector search:** pgvector
**AI:** Replaceable provider layer
**Initial goal:** ₹0 infrastructure cost

---

# 1. The Complete Architecture

The application should be divided into these layers:

```text
                         USER
                           │
                           ▼
                    ┌──────────────┐
                    │   NEXT.JS    │
                    │   FRONTEND   │
                    └──────┬───────┘
                           │
                    HTTPS / API
                           │
                           ▼
                    ┌──────────────┐
                    │  CLOUDFLARE  │
                    │ Workers/Edge │
                    └──────┬───────┘
                           │
                 ┌─────────┴──────────┐
                 │                    │
                 ▼                    ▼
          Application Layer      Background Jobs
                 │                    │
                 │                Cloudflare Queue
                 │                    │
                 ▼                    ▼
          ┌────────────┐        ┌──────────────┐
          │ PostgreSQL │        │  AI Workers  │
          │  Supabase  │        │ Moderation   │
          └─────┬──────┘        │ Embeddings   │
                │               │ Provenance   │
                │               └──────┬───────┘
                │                      │
        ┌───────┼────────┐             │
        │       │        │             │
        ▼       ▼        ▼             ▼
     pgvector  Auth     RLS      Analysis tables
        │
        ▼
 Similar Minds / Search /
 Recommendation / Detection
```

External/static data:

```text
                  CLOUDFLARE R2
                       │
             images / media / avatars
                       │
                       ▼
                  CDN DELIVERY
```

---

# 2. Main Architectural Rule

The user should experience one simple product:

```text
What's on your mind?

[...........................................]

                        Post
```

Internally, the system may perform:

```text
authentication
content validation
AI classification
topic extraction
entity extraction
personal-life detection
AI-generation risk analysis
spam analysis
provenance
embedding
feed ranking
recommendation
reward evaluation
```

The user should not have to understand any of it.

---

# 3. Repository Architecture

I recommend a single repository initially.

```text
idea-network/
│
├── app/
│   ├── (public)/
│   │   ├── page.tsx
│   │   ├── post/
│   │   ├── topic/
│   │   ├── entity/
│   │   └── search/
│   │
│   ├── (auth)/
│   │   ├── login/
│   │   ├── signup/
│   │   └── callback/
│   │
│   ├── home/
│   ├── profile/
│   ├── settings/
│   ├── notifications/
│   │
│   ├── admin/
│   │   ├── moderation/
│   │   ├── users/
│   │   ├── posts/
│   │   ├── ai/
│   │   └── rewards/
│   │
│   └── api/
│       ├── posts/
│       ├── comments/
│       ├── reactions/
│       ├── users/
│       ├── topics/
│       ├── entities/
│       ├── feed/
│       ├── search/
│       ├── notifications/
│       └── admin/
│
├── components/
│   ├── composer/
│   ├── posts/
│   ├── comments/
│   ├── reactions/
│   ├── feed/
│   ├── profile/
│   ├── discovery/
│   ├── entities/
│   └── shared/
│
├── lib/
│   ├── auth/
│   ├── db/
│   ├── ai/
│   ├── moderation/
│   ├── provenance/
│   ├── ranking/
│   ├── recommendations/
│   ├── reputation/
│   ├── rewards/
│   ├── search/
│   ├── storage/
│   ├── telemetry/
│   └── validation/
│
├── server/
│   ├── posts/
│   ├── comments/
│   ├── feeds/
│   ├── profiles/
│   ├── moderation/
│   ├── recommendations/
│   ├── rewards/
│   └── notifications/
│
├── workers/
│   ├── queues/
│   ├── cron/
│   ├── ai/
│   ├── embeddings/
│   └── provenance/
│
├── database/
│   ├── migrations/
│   ├── functions/
│   ├── seed/
│   └── types/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── scripts/
│
├── public/
│
├── middleware.ts
├── next.config.ts
├── wrangler.toml
├── package.json
├── pnpm-lock.yaml
└── tsconfig.json
```

---

# 4. Architectural Boundary

The application should have four major domains.

```text
SOCIAL DOMAIN
    users
    posts
    comments
    reactions
    follows
    bookmarks

INTELLIGENCE DOMAIN
    AI analysis
    embeddings
    interests
    similarity
    ranking
    moderation

TRUST DOMAIN
    provenance
    reports
    moderation
    reputation
    rewards

DISCOVERY DOMAIN
    topics
    entities
    search
    recommendations
```

The domains share PostgreSQL, but their code should remain separated.

---

# 5. Database Technology

Use:

```text
PostgreSQL
+
pgvector
+
Supabase Auth
+
Row Level Security
```

PostgreSQL is the source of truth.

Do not put important application state only in:

* Cloudflare KV
* cache
* AI output
* frontend state
* embeddings

Everything important must eventually be represented in Postgres.

---

# 6. Database Extensions

Initial migration:

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS vector;
```

Optional later:

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
```

`pg_trgm` becomes useful for fuzzy text/entity search.

---

# 7. Identity Model

Supabase Auth owns authentication.

PostgreSQL application tables reference:

```text
auth.users.id
```

Do not duplicate passwords or authentication credentials in your own tables.

Your table:

```text
profiles
```

stores application-level user data.

---

# 8. USERS / PROFILES

```sql
CREATE TABLE profiles (
    id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,

    username TEXT NOT NULL,
    display_name TEXT,
    bio TEXT,

    avatar_url TEXT,

    account_status TEXT NOT NULL DEFAULT 'active'
        CHECK (account_status IN (
            'active',
            'restricted',
            'suspended',
            'deleted'
        )),

    trust_level INTEGER NOT NULL DEFAULT 0,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX profiles_username_lower_idx
ON profiles (LOWER(username));

CREATE INDEX profiles_status_idx
ON profiles (account_status);
```

---

# 9. USER PREFERENCES

The user can customize the experience without manually defining their interests.

```sql
CREATE TABLE user_preferences (
    user_id UUID PRIMARY KEY
        REFERENCES profiles(id) ON DELETE CASCADE,

    language TEXT NOT NULL DEFAULT 'en',

    timezone TEXT,

    content_language TEXT[] NOT NULL DEFAULT ARRAY['en'],

    allow_ai_personalization BOOLEAN NOT NULL DEFAULT TRUE,

    allow_similar_minds BOOLEAN NOT NULL DEFAULT TRUE,

    allow_different_perspectives BOOLEAN NOT NULL DEFAULT TRUE,

    show_sensitive_content BOOLEAN NOT NULL DEFAULT FALSE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

# 10. TOPICS

Topics are created/maintained by the platform.

Users do not need to create or select them when posting.

```sql
CREATE TABLE topics (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    parent_topic_id BIGINT REFERENCES topics(id),

    name TEXT NOT NULL,
    slug TEXT NOT NULL UNIQUE,

    description TEXT,

    status TEXT NOT NULL DEFAULT 'active'
        CHECK (status IN (
            'active',
            'hidden',
            'merged'
        )),

    embedding vector(768),

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

# 11. ENTITIES

Entities represent things being discussed.

Examples:

* OpenAI
* Cursor
* Microsoft
* Python
* Tesla
* iPhone
* Netflix

```sql
CREATE TABLE entities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    name TEXT NOT NULL,
    slug TEXT NOT NULL UNIQUE,

    entity_type TEXT NOT NULL
        CHECK (entity_type IN (
            'person',
            'company',
            'product',
            'software',
            'website',
            'organization',
            'technology',
            'book',
            'movie',
            'game',
            'research',
            'other'
        )),

    description TEXT,
    canonical_url TEXT,

    status TEXT NOT NULL DEFAULT 'active'
        CHECK (status IN (
            'active',
            'hidden',
            'merged'
        )),

    embedding vector(768),

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX entities_name_idx
ON entities USING GIN (to_tsvector('simple', name));
```

---

# 12. ENTITY ALIASES

```sql
CREATE TABLE entity_aliases (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    entity_id UUID NOT NULL REFERENCES entities(id) ON DELETE CASCADE,

    alias TEXT NOT NULL,

    UNIQUE(entity_id, alias)
);

CREATE INDEX entity_alias_lookup_idx
ON entity_aliases (LOWER(alias));
```

---

# 13. USER FOLLOWS USER

```sql
CREATE TABLE user_follows (
    follower_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    following_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (follower_id, following_id),

    CHECK (follower_id <> following_id)
);

CREATE INDEX user_follows_following_idx
ON user_follows (following_id);
```

---

# 14. USER FOLLOWS TOPIC

```sql
CREATE TABLE topic_follows (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    topic_id BIGINT NOT NULL REFERENCES topics(id) ON DELETE CASCADE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, topic_id)
);

CREATE INDEX topic_follows_topic_idx
ON topic_follows (topic_id);
```

---

# 15. USER FOLLOWS ENTITY

```sql
CREATE TABLE entity_follows (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    entity_id UUID NOT NULL REFERENCES entities(id) ON DELETE CASCADE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, entity_id)
);

CREATE INDEX entity_follows_entity_idx
ON entity_follows (entity_id);
```

---

# 16. POSTS

The post itself should stay relatively simple.

```sql
CREATE TABLE posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    author_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    content TEXT NOT NULL,

    status TEXT NOT NULL DEFAULT 'published'
        CHECK (status IN (
            'draft',
            'processing',
            'published',
            'limited',
            'hidden',
            'removed'
        )),

    visibility TEXT NOT NULL DEFAULT 'public'
        CHECK (visibility IN (
            'public',
            'followers'
        )),

    content_hash TEXT NOT NULL,

    published_at TIMESTAMPTZ,

    edited_at TIMESTAMPTZ,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX posts_author_created_idx
ON posts (author_id, created_at DESC);

CREATE INDEX posts_status_created_idx
ON posts (status, created_at DESC);

CREATE INDEX posts_published_idx
ON posts (published_at DESC)
WHERE status = 'published';

CREATE INDEX posts_content_search_idx
ON posts
USING GIN (
    to_tsvector('english', content)
);
```

---

# 17. POST VERSIONS

This is important for provenance.

When someone edits a post, don't simply overwrite history.

```sql
CREATE TABLE post_versions (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    version_number INTEGER NOT NULL,

    content TEXT NOT NULL,

    content_hash TEXT NOT NULL,

    edited_by UUID NOT NULL REFERENCES profiles(id),

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    UNIQUE(post_id, version_number)
);

CREATE INDEX post_versions_post_idx
ON post_versions (post_id, version_number DESC);
```

---

# 18. POST TOPICS

The AI assigns topics.

```sql
CREATE TABLE post_topics (
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    topic_id BIGINT NOT NULL REFERENCES topics(id) ON DELETE CASCADE,

    confidence REAL NOT NULL DEFAULT 0,

    source TEXT NOT NULL DEFAULT 'ai',

    PRIMARY KEY (post_id, topic_id)
);

CREATE INDEX post_topics_topic_idx
ON post_topics (topic_id, post_id);
```

---

# 19. POST ENTITIES

```sql
CREATE TABLE post_entities (
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    entity_id UUID NOT NULL REFERENCES entities(id) ON DELETE CASCADE,

    confidence REAL NOT NULL DEFAULT 0,

    mention_count INTEGER NOT NULL DEFAULT 1,

    PRIMARY KEY (post_id, entity_id)
);

CREATE INDEX post_entities_entity_idx
ON post_entities (entity_id, post_id);
```

---

# 20. POST MEDIA

Files live in R2.

The database stores metadata.

```sql
CREATE TABLE post_media (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    media_type TEXT NOT NULL
        CHECK (media_type IN (
            'image',
            'video',
            'audio',
            'document'
        )),

    storage_key TEXT NOT NULL,

    mime_type TEXT,

    file_size BIGINT,

    width INTEGER,
    height INTEGER,

    duration_ms BIGINT,

    content_hash TEXT,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX post_media_post_idx
ON post_media (post_id);
```

---

# 21. POST SOURCES

A post may contain a source.

```sql
CREATE TABLE post_sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    url TEXT NOT NULL,

    canonical_url TEXT,

    title TEXT,

    publisher TEXT,

    site_name TEXT,

    published_at TIMESTAMPTZ,

    fetched_at TIMESTAMPTZ,

    source_quality_score REAL,

    verification_status TEXT NOT NULL DEFAULT 'unknown'
        CHECK (verification_status IN (
            'unknown',
            'verified',
            'questionable',
            'unavailable'
        )),

    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX post_sources_post_idx
ON post_sources (post_id);
```

---

# 22. POST RELATIONSHIPS

This handles:

* quote
* response
* derivative
* repost
* reference

```sql
CREATE TABLE post_relations (
    source_post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    target_post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    relation_type TEXT NOT NULL
        CHECK (relation_type IN (
            'quote',
            'response',
            'derived_from',
            'repost_of',
            'reference'
        )),

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (
        source_post_id,
        target_post_id,
        relation_type
    ),

    CHECK (source_post_id <> target_post_id)
);

CREATE INDEX post_relations_target_idx
ON post_relations (target_post_id, relation_type);
```

---

# 23. SHARES

A share is a user action.

A repost can therefore preserve the original post rather than creating a duplicate original.

```sql
CREATE TABLE post_shares (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    commentary_post_id UUID REFERENCES posts(id) ON DELETE SET NULL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    UNIQUE(user_id, post_id)
);

CREATE INDEX post_shares_post_idx
ON post_shares (post_id, created_at DESC);
```

---

# 24. COMMENTS

```sql
CREATE TABLE comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    author_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    parent_comment_id UUID REFERENCES comments(id) ON DELETE CASCADE,

    content TEXT NOT NULL,

    status TEXT NOT NULL DEFAULT 'published'
        CHECK (status IN (
            'published',
            'hidden',
            'removed'
        )),

    content_hash TEXT NOT NULL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX comments_post_idx
ON comments (post_id, created_at ASC);

CREATE INDEX comments_parent_idx
ON comments (parent_comment_id, created_at ASC);

CREATE INDEX comments_author_idx
ON comments (author_id, created_at DESC);
```

---

# 25. POST REACTIONS

The platform does not use generic likes/dislikes.

A user has one current reaction per post.

```sql
CREATE TABLE post_reactions (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    reaction_type TEXT NOT NULL
        CHECK (reaction_type IN (
            'helpful',
            'not_helpful',
            'more_detail',
            'disagree',
            'curious'
        )),

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, post_id)
);

CREATE INDEX post_reactions_post_type_idx
ON post_reactions (post_id, reaction_type);

CREATE INDEX post_reactions_user_idx
ON post_reactions (user_id, created_at DESC);
```

---

# 26. COMMENT REACTIONS

Same concept for comments.

```sql
CREATE TABLE comment_reactions (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    comment_id UUID NOT NULL REFERENCES comments(id) ON DELETE CASCADE,

    reaction_type TEXT NOT NULL
        CHECK (reaction_type IN (
            'helpful',
            'not_helpful',
            'more_detail',
            'disagree',
            'curious'
        )),

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, comment_id)
);
```

---

# 27. BOOKMARKS

```sql
CREATE TABLE post_bookmarks (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, post_id)
);

CREATE INDEX post_bookmarks_user_idx
ON post_bookmarks (user_id, created_at DESC);
```

---

# 28. BLOCKS

```sql
CREATE TABLE user_blocks (
    blocker_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    blocked_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (blocker_id, blocked_id),

    CHECK (blocker_id <> blocked_id)
);
```

---

# 29. MUTING

```sql
CREATE TABLE user_mutes (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    muted_user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,

    muted_topic_id BIGINT REFERENCES topics(id) ON DELETE CASCADE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    CHECK (
        (muted_user_id IS NOT NULL AND muted_topic_id IS NULL)
        OR
        (muted_user_id IS NULL AND muted_topic_id IS NOT NULL)
    )
);
```

---

# 30. REPORTS

```sql
CREATE TABLE content_reports (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    reporter_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    post_id UUID REFERENCES posts(id) ON DELETE CASCADE,

    comment_id UUID REFERENCES comments(id) ON DELETE CASCADE,

    reason TEXT NOT NULL,

    details TEXT,

    status TEXT NOT NULL DEFAULT 'open'
        CHECK (status IN (
            'open',
            'reviewing',
            'resolved',
            'dismissed'
        )),

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    CHECK (
        (post_id IS NOT NULL AND comment_id IS NULL)
        OR
        (post_id IS NULL AND comment_id IS NOT NULL)
    )
);
```

---

# 31. AI POST ANALYSIS

This table stores the AI's interpretation.

```sql
CREATE TABLE post_ai_analysis (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    model_name TEXT NOT NULL,
    model_version TEXT NOT NULL,

    intent TEXT,

    personal_life_score REAL,
    helpfulness_score REAL,
    originality_score REAL,
    discussion_score REAL,
    factual_claim_score REAL,

    spam_score REAL,
    ai_generation_risk REAL,
    automation_risk REAL,
    safety_score REAL,

    confidence REAL,

    analysis JSONB NOT NULL DEFAULT '{}'::jsonb,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX post_ai_analysis_post_idx
ON post_ai_analysis (post_id, created_at DESC);
```

Keep historical analyses rather than overwriting them.

---

# 32. AI CLAIMS

For posts containing claims.

```sql
CREATE TABLE post_claims (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    claim_text TEXT NOT NULL,

    claim_type TEXT
        CHECK (claim_type IN (
            'factual',
            'numerical',
            'scientific',
            'news',
            'personal_experience',
            'opinion',
            'accusation'
        )),

    support_status TEXT DEFAULT 'unknown'
        CHECK (support_status IN (
            'unknown',
            'supported',
            'unsupported',
            'disputed'
        )),

    confidence REAL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

# 33. POST EMBEDDINGS

```sql
CREATE TABLE post_embeddings (
    post_id UUID PRIMARY KEY REFERENCES posts(id) ON DELETE CASCADE,

    embedding vector(768) NOT NULL,

    model_name TEXT NOT NULL,
    model_version TEXT NOT NULL,

    source_hash TEXT NOT NULL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Later:

```sql
CREATE INDEX post_embeddings_hnsw_idx
ON post_embeddings
USING hnsw (embedding vector_cosine_ops);
```

The embedding dimension `768` is a placeholder. It must match whichever embedding model you actually select.

---

# 34. USER INTEREST SCORES

The system should maintain interpretable interest scores.

```sql
CREATE TABLE user_topic_scores (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    topic_id BIGINT NOT NULL REFERENCES topics(id) ON DELETE CASCADE,

    score REAL NOT NULL DEFAULT 0,

    confidence REAL NOT NULL DEFAULT 0,

    interaction_count INTEGER NOT NULL DEFAULT 0,

    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, topic_id)
);

CREATE INDEX user_topic_scores_top_idx
ON user_topic_scores (user_id, score DESC);
```

---

# 35. USER ENTITY INTEREST

```sql
CREATE TABLE user_entity_scores (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    entity_id UUID NOT NULL REFERENCES entities(id) ON DELETE CASCADE,

    score REAL NOT NULL DEFAULT 0,

    interaction_count INTEGER NOT NULL DEFAULT 0,

    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, entity_id)
);

CREATE INDEX user_entity_scores_top_idx
ON user_entity_scores (user_id, score DESC);
```

---

# 36. USER EMBEDDING

This represents the user's current interest space.

```sql
CREATE TABLE user_interest_embeddings (
    user_id UUID PRIMARY KEY REFERENCES profiles(id) ON DELETE CASCADE,

    embedding vector(768) NOT NULL,

    model_name TEXT NOT NULL,
    model_version TEXT NOT NULL,

    source_version INTEGER NOT NULL DEFAULT 1,

    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

# 37. SIMILAR MINDS

Do not calculate nearest users from scratch every time someone opens a profile.

Precompute candidates.

```sql
CREATE TABLE user_similarities (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    similar_user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    similarity_score REAL NOT NULL,

    overlap_type TEXT[] NOT NULL DEFAULT '{}',

    calculated_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, similar_user_id),

    CHECK (user_id <> similar_user_id)
);

CREATE INDEX user_similarities_top_idx
ON user_similarities (user_id, similarity_score DESC);
```

Example `overlap_type`:

```text
AI
robotics
software
same_project
same_topics
same_entities
```

---

# 38. DIFFERENT PERSPECTIVES

This is separate from Similar Minds.

```sql
CREATE TABLE user_perspectives (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    perspective_user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    shared_interest_score REAL NOT NULL,

    perspective_difference_score REAL NOT NULL,

    constructive_score REAL NOT NULL,

    final_score REAL NOT NULL,

    calculated_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, perspective_user_id),

    CHECK (user_id <> perspective_user_id)
);
```

The algorithm looks for:

```text
shared interests
+
different conclusions
+
constructive discussion
```

rather than simple controversy.

---

# 39. USER BEHAVIOR PROFILE

```sql
CREATE TABLE user_behavior_profiles (
    user_id UUID PRIMARY KEY REFERENCES profiles(id) ON DELETE CASCADE,

    typical_post_length REAL,
    typical_composition_seconds REAL,
    typical_active_typing_seconds REAL,

    typical_edit_ratio REAL,
    typical_delete_ratio REAL,

    typical_paste_ratio REAL,

    typical_post_velocity REAL,

    baseline_version INTEGER NOT NULL DEFAULT 1,

    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

# 40. COMPOSITION SESSIONS

This is the anti-AI behavioral layer.

```sql
CREATE TABLE composition_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    post_id UUID REFERENCES posts(id) ON DELETE SET NULL,

    input_mode TEXT NOT NULL DEFAULT 'typing'
        CHECK (input_mode IN (
            'typing',
            'voice',
            'paste',
            'mixed',
            'unknown'
        )),

    started_at TIMESTAMPTZ NOT NULL,
    published_at TIMESTAMPTZ,

    total_duration_ms BIGINT,

    active_duration_ms BIGINT,

    character_count INTEGER,

    word_count INTEGER,

    paste_count INTEGER DEFAULT 0,

    pasted_character_count INTEGER DEFAULT 0,

    delete_count INTEGER DEFAULT 0,

    replacement_count INTEGER DEFAULT 0,

    undo_count INTEGER DEFAULT 0,

    redo_count INTEGER DEFAULT 0,

    pause_count INTEGER DEFAULT 0,

    mean_pause_ms REAL,

    max_pause_ms REAL,

    editing_ratio REAL,

    typing_rate REAL,

    focus_loss_count INTEGER DEFAULT 0,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX composition_sessions_user_idx
ON composition_sessions (user_id, created_at DESC);

CREATE INDEX composition_sessions_post_idx
ON composition_sessions (post_id);
```

---

# 41. Important Privacy Decision

Do NOT create a table containing every keystroke.

Do not store:

```text
key A
key I
key backspace
key space
...
```

Instead collect features:

```text
typing speed
pause distribution
editing ratio
paste count
composition duration
```

The browser can aggregate telemetry before sending it.

---

# 42. RISK SCORES

```sql
CREATE TABLE content_risk_scores (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    ai_generation_risk REAL NOT NULL DEFAULT 0,
    automation_risk REAL NOT NULL DEFAULT 0,
    spam_risk REAL NOT NULL DEFAULT 0,
    safety_risk REAL NOT NULL DEFAULT 0,

    decision TEXT,

    model_version TEXT,
    rules_version TEXT,

    explanation JSONB NOT NULL DEFAULT '{}'::jsonb,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX content_risk_scores_post_idx
ON content_risk_scores (post_id, created_at DESC);
```

---

# 43. MODERATION CASES

```sql
CREATE TABLE moderation_cases (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    post_id UUID REFERENCES posts(id) ON DELETE CASCADE,

    comment_id UUID REFERENCES comments(id) ON DELETE CASCADE,

    user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,

    source TEXT NOT NULL
        CHECK (source IN (
            'ai',
            'report',
            'admin',
            'automated_rule'
        )),

    severity TEXT NOT NULL
        CHECK (severity IN (
            'low',
            'medium',
            'high',
            'critical'
        )),

    status TEXT NOT NULL DEFAULT 'open'
        CHECK (status IN (
            'open',
            'reviewing',
            'resolved',
            'dismissed'
        )),

    reason TEXT,

    evidence JSONB NOT NULL DEFAULT '{}'::jsonb,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    resolved_at TIMESTAMPTZ
);
```

---

# 44. MODERATION ACTIONS

Never overwrite moderation history.

```sql
CREATE TABLE moderation_actions (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    case_id UUID NOT NULL REFERENCES moderation_cases(id) ON DELETE CASCADE,

    actor_type TEXT NOT NULL
        CHECK (actor_type IN (
            'ai',
            'admin',
            'system'
        )),

    actor_id UUID REFERENCES profiles(id),

    action TEXT NOT NULL
        CHECK (action IN (
            'allow',
            'limit',
            'label',
            'hide',
            'remove',
            'restore',
            'restrict_user'
        )),

    reason TEXT,

    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

# 45. CONTRIBUTION EVALUATIONS

This stores how each post performed against the contribution model.

```sql
CREATE TABLE contribution_evaluations (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    evaluation_date DATE NOT NULL,

    helpfulness REAL DEFAULT 0,
    originality REAL DEFAULT 0,
    awareness REAL DEFAULT 0,
    empathy REAL DEFAULT 0,
    discussion_quality REAL DEFAULT 0,
    evidence_quality REAL DEFAULT 0,
    usefulness REAL DEFAULT 0,

    community_feedback_score REAL DEFAULT 0,

    spam_penalty REAL DEFAULT 0,
    automation_penalty REAL DEFAULT 0,

    impact_score REAL NOT NULL DEFAULT 0,

    model_version TEXT NOT NULL,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX contribution_evaluations_user_date_idx
ON contribution_evaluations (user_id, evaluation_date DESC);

CREATE INDEX contribution_evaluations_post_idx
ON contribution_evaluations (post_id);
```

---

# 46. DAILY CONTRIBUTIONS

This enforces the one-rewarded-post-per-day concept.

```sql
CREATE TABLE daily_contributions (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    contribution_date DATE NOT NULL,

    selected_post_id UUID REFERENCES posts(id),

    impact_score REAL NOT NULL,

    reward_points INTEGER NOT NULL DEFAULT 0,

    status TEXT NOT NULL DEFAULT 'awarded'
        CHECK (status IN (
            'pending',
            'awarded',
            'reversed'
        )),

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, contribution_date)
);

CREATE INDEX daily_contributions_post_idx
ON daily_contributions (selected_post_id);
```

This guarantees:

```text
user + date = one reward
```

---

# 47. REWARD LEDGER

Never just increment a points counter.

Use an immutable ledger.

```sql
CREATE TABLE reward_ledger (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    points INTEGER NOT NULL,

    transaction_type TEXT NOT NULL
        CHECK (transaction_type IN (
            'daily_contribution',
            'bonus',
            'redemption',
            'reversal',
            'admin_adjustment'
        )),

    reference_id TEXT,

    description TEXT,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX reward_ledger_user_idx
ON reward_ledger (user_id, created_at DESC);
```

Current balance:

```sql
SELECT COALESCE(SUM(points), 0)
FROM reward_ledger
WHERE user_id = $1;
```

For scale, maintain a cached balance later.

---

# 48. REWARD CATALOG

```sql
CREATE TABLE reward_catalog (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    name TEXT NOT NULL,

    description TEXT,

    points_cost INTEGER NOT NULL,

    reward_type TEXT NOT NULL
        CHECK (reward_type IN (
            'coupon',
            'subscription',
            'credit',
            'badge',
            'access',
            'event',
            'other'
        )),

    provider_name TEXT,

    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,

    is_active BOOLEAN NOT NULL DEFAULT TRUE,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

# 49. REWARD REDEMPTIONS

```sql
CREATE TABLE reward_redemptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    reward_id UUID NOT NULL REFERENCES reward_catalog(id),

    points_spent INTEGER NOT NULL,

    status TEXT NOT NULL DEFAULT 'pending'
        CHECK (status IN (
            'pending',
            'fulfilled',
            'cancelled',
            'failed'
        )),

    fulfillment_data JSONB NOT NULL DEFAULT '{}'::jsonb,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

# 50. REPUTATION

Keep reputation separate from rewards.

```sql
CREATE TABLE user_reputation (
    user_id UUID PRIMARY KEY REFERENCES profiles(id) ON DELETE CASCADE,

    contribution_score REAL NOT NULL DEFAULT 0,

    helpfulness_score REAL NOT NULL DEFAULT 0,

    discussion_score REAL NOT NULL DEFAULT 0,

    originality_score REAL NOT NULL DEFAULT 0,

    evidence_score REAL NOT NULL DEFAULT 0,

    trust_score REAL NOT NULL DEFAULT 0,

    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

# 51. TOPIC-SPECIFIC REPUTATION

Avoid saying someone is universally an expert.

```sql
CREATE TABLE user_topic_reputation (
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    topic_id BIGINT NOT NULL REFERENCES topics(id) ON DELETE CASCADE,

    reputation_score REAL NOT NULL DEFAULT 0,

    evidence_score REAL NOT NULL DEFAULT 0,

    helpfulness_score REAL NOT NULL DEFAULT 0,

    contribution_count INTEGER NOT NULL DEFAULT 0,

    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    PRIMARY KEY (user_id, topic_id)
);
```

This allows:

```text
@alex
AI reputation: high
Robotics reputation: medium
Finance reputation: unknown
```

without giving them a fake universal expertise score.

---

# 52. NOTIFICATIONS

```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,

    actor_id UUID REFERENCES profiles(id) ON DELETE SET NULL,

    type TEXT NOT NULL,

    post_id UUID REFERENCES posts(id) ON DELETE CASCADE,

    comment_id UUID REFERENCES comments(id) ON DELETE CASCADE,

    payload JSONB NOT NULL DEFAULT '{}'::jsonb,

    read_at TIMESTAMPTZ,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX notifications_user_idx
ON notifications (user_id, created_at DESC);
```

---

# 53. CONTENT EVENTS

Not every raw feed impression needs to live forever in PostgreSQL.

For important interactions:

```sql
CREATE TABLE content_events (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    user_id UUID REFERENCES profiles(id) ON DELETE SET NULL,

    post_id UUID REFERENCES posts(id) ON DELETE CASCADE,

    event_type TEXT NOT NULL,

    value NUMERIC,

    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX content_events_user_time_idx
ON content_events (user_id, created_at DESC);

CREATE INDEX content_events_post_time_idx
ON content_events (post_id, created_at DESC);
```

At larger scale, raw analytics should move to an analytics system.

---

# 54. FEED / RANKING SNAPSHOTS

Do not materialize everyone's feed at first.

For Trending and other globally useful lists, maintain snapshots.

```sql
CREATE TABLE ranking_snapshots (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    snapshot_type TEXT NOT NULL
        CHECK (snapshot_type IN (
            'trending',
            'topic',
            'entity'
        )),

    reference_id TEXT,

    generated_at TIMESTAMPTZ NOT NULL,

    items JSONB NOT NULL
);
```

Example:

```json
{
  "items": [
    {
      "post_id": "...",
      "score": 87.4
    }
  ]
}
```

---

# 55. FEATURE FLAGS

```sql
CREATE TABLE feature_flags (
    key TEXT PRIMARY KEY,

    enabled BOOLEAN NOT NULL DEFAULT FALSE,

    config JSONB NOT NULL DEFAULT '{}'::jsonb,

    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Examples:

```text
enable_ai_analysis
enable_ai_risk_scoring
enable_similar_minds
enable_daily_rewards
enable_provenance
enable_external_timestamping
```

---

# 56. AI MODEL REGISTRY

```sql
CREATE TABLE ai_models (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    model_name TEXT NOT NULL,

    provider TEXT NOT NULL,

    model_version TEXT NOT NULL,

    task TEXT NOT NULL,

    active BOOLEAN NOT NULL DEFAULT FALSE,

    configuration JSONB NOT NULL DEFAULT '{}'::jsonb,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    UNIQUE(provider, model_name, model_version, task)
);
```

This lets you switch models without rewriting application logic.

---

# 57. PROVENANCE

Core provenance record:

```sql
CREATE TABLE content_provenance (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,

    version_number INTEGER NOT NULL,

    content_hash TEXT NOT NULL,

    media_hash TEXT,

    previous_provenance_hash TEXT,

    registered_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    external_timestamp TEXT,

    timestamp_provider TEXT,

    UNIQUE(post_id, version_number)
);

CREATE INDEX provenance_hash_idx
ON content_provenance (content_hash);

CREATE INDEX provenance_previous_idx
ON content_provenance (previous_provenance_hash);
```

---

# 58. PROVENANCE BATCHES

For future Merkle-tree/blockchain timestamping:

```sql
CREATE TABLE provenance_batches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    merkle_root TEXT NOT NULL,

    item_count INTEGER NOT NULL,

    timestamp_provider TEXT,

    external_proof TEXT,

    status TEXT NOT NULL DEFAULT 'pending'
        CHECK (status IN (
            'pending',
            'submitted',
            'confirmed',
            'failed'
        )),

    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),

    confirmed_at TIMESTAMPTZ
);
```

---

# 59. ADMIN AUDIT LOG

```sql
CREATE TABLE admin_audit_log (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,

    admin_user_id UUID REFERENCES profiles(id),

    action TEXT NOT NULL,

    target_type TEXT,

    target_id TEXT,

    details JSONB NOT NULL DEFAULT '{}'::jsonb,

    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX admin_audit_time_idx
ON admin_audit_log (created_at DESC);
```

---

# 60. COMPLETE RELATIONSHIP MAP

The important relationships look like this:

```text
AUTH.USERS
    │
    ▼
PROFILES
    │
    ├────────────── USER FOLLOWS ───────────────┐
    │                                           │
    ├── POSTS                                   ▼
    │     │                                  PROFILES
    │     │
    │     ├── POST TOPICS ───────────────► TOPICS
    │     │
    │     ├── POST ENTITIES ─────────────► ENTITIES
    │     │
    │     ├── POST MEDIA
    │     ├── POST SOURCES
    │     ├── POST VERSIONS
    │     ├── POST RELATIONS ────────────► POSTS
    │     ├── POST REACTIONS
    │     ├── COMMENTS
    │     ├── BOOKMARKS
    │     ├── AI ANALYSIS
    │     ├── RISK SCORES
    │     ├── PROVENANCE
    │     └── EMBEDDINGS
    │
    ├── USER TOPIC SCORES
    ├── USER ENTITY SCORES
    ├── USER EMBEDDING
    ├── USER BEHAVIOR PROFILE
    ├── COMPOSITION SESSIONS
    ├── SIMILAR MINDS
    ├── DIFFERENT PERSPECTIVES
    ├── REPUTATION
    ├── REWARDS
    └── NOTIFICATIONS
```

---

# 61. The Post Lifecycle

This is the most important backend flow.

## Step 1 — User writes

Browser collects composition features locally.

```text
typing
pause
edit
paste
delete
voice
```

---

## Step 2 — User clicks Post

Frontend:

```text
POST /api/posts
```

Payload:

```json
{
  "content": "...",
  "media": [],
  "composition": {
    "inputMode": "typing",
    "totalMs": 184000,
    "activeMs": 142000,
    "pasteCount": 0,
    "deleteCount": 17,
    "editingRatio": 0.14
  }
}
```

---

# 62. Server Processing

The API performs only fast operations:

```text
authenticate
validate
check account status
check rate limit
sanitize content
generate content hash
create post
create composition session
create provenance record
queue analysis
return post
```

The user should not wait for AI analysis.

---

# 63. Immediate Response

Return:

```json
{
  "postId": "...",
  "status": "processing"
}
```

Frontend can immediately show the post.

---

# 64. Queue Job

The worker receives:

```json
{
  "type": "analyze_post",
  "postId": "..."
}
```

---

# 65. AI Analysis Pipeline

```text
POST
 │
 ▼
Content Analyzer
 │
 ├── intent
 ├── topics
 ├── entities
 ├── claims
 ├── personal-life
 ├── helpfulness
 └── discussion potential
 │
 ▼
Behavior Analyzer
 │
 ├── typing
 ├── editing
 ├── paste
 ├── account baseline
 └── automation
 │
 ▼
Similarity
 │
 ├── duplicate
 ├── derived
 └── related
 │
 ▼
Safety / Moderation
 │
 ▼
Provenance
 │
 ▼
Embedding
 │
 ▼
Ranking Features
```

---

# 66. Why This Should Be Asynchronous

Suppose AI takes:

```text
800 ms
+
500 ms
+
1 sec
+
embedding
```

You do not want:

```text
User click Post
      ↓
wait 3 seconds
      ↓
post appears
```

Instead:

```text
User click Post
      ↓
DB write
      ↓
post appears
      ↓
AI works in background
```

---

# 67. Frontend Data Flow

For normal pages:

```text
Next.js Server Component
        │
        ▼
server service
        │
        ▼
Postgres
```

For interactive components:

```text
React Client Component
        │
        ▼
API Route / Server Action
        │
        ▼
service layer
        │
        ▼
Postgres
```

---

# 68. Service Layer

Do not put SQL directly in every route handler.

For example:

```text
app/api/posts/route.ts
```

should call:

```text
server/posts/createPost.ts
```

which calls:

```text
lib/db/posts.ts
```

Conceptually:

```text
HTTP
 ↓
route
 ↓
service
 ↓
repository/data access
 ↓
Postgres
```

---

# 69. Example Create Post Flow

```text
route.ts
   ↓
createPost()
   ↓
validatePost()
   ↓
authorizePost()
   ↓
checkRateLimit()
   ↓
createContentHash()
   ↓
insertPost()
   ↓
insertComposition()
   ↓
insertProvenance()
   ↓
enqueueAnalysis()
```

---

# 70. Repository Layer

Keep database operations grouped.

```text
lib/db/
    posts.ts
    comments.ts
    profiles.ts
    reactions.ts
    follows.ts
    topics.ts
    entities.ts
    rewards.ts
    moderation.ts
```

The service layer should not know SQL syntax unless necessary.

---

# 71. API Endpoints

## Posts

```text
POST   /api/posts
GET    /api/posts/:id
PATCH  /api/posts/:id
DELETE /api/posts/:id
```

## Comments

```text
POST   /api/posts/:id/comments
GET    /api/posts/:id/comments
PATCH  /api/comments/:id
DELETE /api/comments/:id
```

## Reactions

```text
PUT    /api/posts/:id/reaction
DELETE /api/posts/:id/reaction
```

## Saves

```text
PUT    /api/posts/:id/save
DELETE /api/posts/:id/save
```

## Follow

```text
PUT    /api/users/:id/follow
DELETE /api/users/:id/follow
```

---

# 72. Discovery API

```text
GET /api/feed
GET /api/trending
GET /api/topics
GET /api/topics/:slug
GET /api/entities/:slug
GET /api/search
```

---

# 73. Similar Minds API

```text
GET /api/users/:id/similar
GET /api/users/:id/different-perspectives
GET /api/users/:id/working-on
```

---

# 74. Rewards API

```text
GET /api/me/contributions
GET /api/me/rewards
GET /api/me/reward-history
POST /api/rewards/:id/redeem
```

Users should never have an API like:

```text
POST /api/me/add-points
```

Obviously.

All points originate server-side.

---

# 75. Admin APIs

```text
GET  /api/admin/moderation
POST /api/admin/moderation/:id/action

GET  /api/admin/users/:id
GET  /api/admin/posts/:id

GET  /api/admin/ai/risk
GET  /api/admin/rewards
```

All protected by admin roles and server-side authorization.

---

# 76. Frontend Page Structure

## Homepage

```text
/home
```

Contains:

```text
Top navigation
Composer
For You / Following / Trending
Feed
```

---

## Post

```text
/post/[id]
```

Contains:

```text
post
reactions
comments
related posts
source
provenance
entity/topic context
```

---

## Profile

```text
/@username
```

Contains:

```text
contributions
topics
reviews
similar minds
different perspectives
```

---

## Entity

```text
/entity/[slug]
```

Contains:

```text
overview
latest discussions
reviews
questions
news
related entities
```

---

# 77. Feed Query Strategy

For the first version, don't create a feed row for every user/post pair.

That would explode storage.

Instead:

```text
user preferences
+
topic scores
+
following
+
recent posts
+
ranking
```

are queried to generate candidates.

Then rank the candidates.

---

# 78. Candidate Generation

Example candidate sources:

```text
1. followed users
2. followed topics
3. followed entities
4. Similar Minds interests
5. recent trending posts
6. similar-post embeddings
7. new/discovery content
```

Then:

```text
candidate pool
      ↓
deduplicate
      ↓
filter blocked/muted
      ↓
rank
      ↓
return 30
```

---

# 79. Feed Ranking Architecture

```text
Candidate Generation
        ↓
Policy Filtering
        ↓
Quality Filtering
        ↓
Ranking
        ↓
Diversification
        ↓
Pagination
```

---

# 80. Feed Diversification

Do not show:

```text
AI
AI
AI
AI
AI
```

even if AI is the user's strongest interest.

The system should mix:

```text
high-interest topics
similar topics
discovery
different perspectives
```

This prevents an overly repetitive feed.

---

# 81. Similar Minds Calculation

Run periodically.

```text
user embeddings
      ↓
pgvector nearest neighbors
      ↓
remove blocked users
      ↓
remove inactive users
      ↓
apply interest overlap
      ↓
store top N
```

For example:

```text
Top 100 candidates
       ↓
filter
       ↓
Top 5 shown to user
```

---

# 82. Different Perspectives Calculation

Find:

```text
shared topic score > threshold
```

then:

```text
opinion difference > threshold
```

then:

```text
constructive interaction > threshold
```

This makes the match meaningful.

---

# 83. Working-on-Similar-Things

Use:

* project-related entities
* repeated topic combinations
* content phrases
* self-described project context
* links to GitHub/project sites
* recurring discussions

The AI produces a semantic project profile.

---

# 84. Company / Software Review Architecture

No separate "review app".

Instead:

```text
normal post
      ↓
AI classification
      ↓
entity detected
      ↓
review intent detected
      ↓
entity page association
```

This makes every post reusable across the network.

---

# 85. Example

User:

> "I've used GitHub Actions for two years. It's powerful, but debugging failed workflows is painful."

Stored as:

```text
post
 ├── entity: GitHub
 ├── entity: GitHub Actions
 ├── intent: review
 ├── experience: long-term
 ├── topic: developer tools
 └── personal-life score: low
```

The user did nothing except type the sentence.

---

# 86. Entity-Level Aggregate Data

Eventually maintain:

```text
entity_metrics
```

with:

```text
discussion_count
review_count
helpful_count
positive_experience_count
negative_experience_count
question_count
trend_score
```

Do not reduce it to just a star rating.

---

# 87. Notification Architecture

When an event happens:

```text
helpful reaction
      ↓
notification job
      ↓
notification row
      ↓
optional realtime update
```

For larger volume, notifications should be queued rather than synchronously inserted during the user request.

---

# 88. Reward Calculation Flow

At the end of the reward window:

```text
fetch user's eligible posts
      ↓
exclude removed/spam posts
      ↓
calculate contribution scores
      ↓
choose highest
      ↓
insert daily_contribution
      ↓
insert reward_ledger
```

Unique constraint ensures one contribution per date.

---

# 89. Reward Abuse Protection

Before award:

```text
check:
- self-reactions
- suspicious engagement
- bot activity
- account coordination
- duplicate content
- moderation status
```

If suspicious:

```text
reward = pending
```

until reviewed.

---

# 90. AI Moderator Architecture

The moderator should be an orchestrator:

```text
ModerationOrchestrator
│
├── ContentSafetyAnalyzer
├── SpamDetector
├── AIRiskDetector
├── PersonalLifeDetector
├── ClaimAnalyzer
├── DuplicateDetector
└── AccountBehaviorAnalyzer
```

Each returns structured data.

The orchestrator produces:

```text
ModerationDecision
```

---

# 91. Moderation Decision Object

Conceptually:

```ts
type ModerationDecision = {
  action:
    | "allow"
    | "limit"
    | "label"
    | "review"
    | "remove";

  confidence: number;

  reasons: string[];

  scores: {
    safety: number;
    spam: number;
    aiRisk: number;
    automation: number;
  };

  modelVersions: string[];
};
```

---

# 92. AI Provider Abstraction

Do not write:

```ts
await openai.chat.completions.create(...)
```

throughout the codebase.

Instead:

```ts
const result = await ai.content.analyze(input);
```

Provider selection happens underneath.

---

# 93. AI Provider Structure

```text
lib/ai/
│
├── index.ts
├── types.ts
│
├── providers/
│   ├── mock.ts
│   ├── workers-ai.ts
│   └── local.ts
│
├── content/
│   ├── analyzer.ts
│   └── prompts.ts
│
├── moderation/
│   ├── analyzer.ts
│   └── rules.ts
│
├── embeddings/
│   ├── provider.ts
│   └── similarity.ts
│
└── behavior/
    ├── analyzer.ts
    └── features.ts
```

---

# 94. AI Failure Handling

If AI fails:

```text
post remains published
```

unless the content already triggers deterministic safety rules.

AI analysis can be retried.

This means the AI layer is:

> asynchronous intelligence

not:

> the single point of failure for posting.

---

# 95. Background Queues

Recommended queues:

```text
post-analysis
embedding-analysis
moderation
provenance
notifications
reward-calculation
interest-rebuild
trend-calculation
source-fetch
```

---

# 96. Cron Jobs

Suggested schedules:

```text
every 5 minutes
    trend calculation

hourly
    interest score update

daily
    daily contribution selection

daily
    similarity rebuild

daily
    provenance batch

weekly
    reputation recalculation

weekly
    cleanup / retention jobs
```

Actual schedules can change after measurement.

---

# 97. File Upload Flow

Do not send large files through Next.js.

Use:

```text
Browser
   ↓
request upload URL
   ↓
server authorizes
   ↓
R2 signed upload
   ↓
browser uploads directly to R2
   ↓
metadata returned
   ↓
post references media
```

This avoids unnecessary application bandwidth.

---

# 98. Content Hashing

For text:

```text
normalize
 ↓
UTF-8
 ↓
SHA-256
```

For media:

```text
binary
 ↓
SHA-256
```

Later use perceptual hashes for modified videos/images.

---

# 99. Duplicate Detection

Start with:

```text
exact content hash
```

Then add:

```text
normalized text similarity
```

Then:

```text
embedding similarity
```

Then for media:

```text
perceptual hash
```

This is a progressive system.

---

# 100. No Need for Blockchain Database

The database remains:

```text
Postgres
```

Blockchain/timestamping is simply an external verification layer.

The normal application does not query blockchain for every post.

---

# 101. Search Architecture

Initial:

```text
Postgres full-text search
```

Later:

```text
keyword search
+
pgvector
+
entity/topic filtering
```

Example:

> "developers complaining about AI coding tools"

can eventually become a semantic query instead of a literal keyword search.

---

# 102. Privacy by Architecture

Sensitive behavior should have separate retention rules.

For example:

```text
raw composition feature:
retain limited time

aggregated user baseline:
retain longer

moderation decision:
retain long term

raw event data:
analytics retention policy
```

Do not keep raw behavioral data forever.

---

# 103. RLS Strategy

At minimum:

### Public

Anyone can read:

* published public posts
* public comments
* public profiles
* public topics
* public entities

### User

Can modify:

* own profile
* own posts
* own comments
* own reactions
* own bookmarks

### System

Can modify:

* AI analysis
* reputation
* rewards
* provenance
* moderation

### Admin

Can inspect/manage:

* moderation
* reports
* account restrictions
* reward corrections
* system configuration

---

# 104. Critical Security Rule

Never let the browser directly write:

```text
reward_ledger
user_reputation
post_ai_analysis
content_risk_scores
content_provenance
```

Those are server/system-owned tables.

---

# 105. Postgres Functions

Use SQL/RPC for operations that benefit from atomicity.

Examples:

```text
create_post_transaction()
apply_reaction()
calculate_reward_balance()
select_daily_contribution()
record_provenance()
```

Particularly:

### Reaction

The operation should atomically:

```text
remove previous reaction
insert/update new reaction
update counters if using cached counters
```

---

# 106. Cached Counters

Initially calculate:

```text
COUNT(*)
```

for:

* reactions
* comments
* shares

At scale, add:

```text
post_metrics
```

```sql
CREATE TABLE post_metrics (
    post_id UUID PRIMARY KEY REFERENCES posts(id) ON DELETE CASCADE,

    helpful_count INTEGER NOT NULL DEFAULT 0,
    not_helpful_count INTEGER NOT NULL DEFAULT 0,
    more_detail_count INTEGER NOT NULL DEFAULT 0,
    disagree_count INTEGER NOT NULL DEFAULT 0,
    curious_count INTEGER NOT NULL DEFAULT 0,

    comment_count INTEGER NOT NULL DEFAULT 0,
    share_count INTEGER NOT NULL DEFAULT 0,
    bookmark_count INTEGER NOT NULL DEFAULT 0,

    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

These can be asynchronously maintained.

---

# 107. Why Counters Matter

Feed ranking should not repeatedly scan millions of reactions.

Instead:

```text
post
 ↓
post_metrics
 ↓
ranking
```

This becomes important at scale.

---

# 108. Similar Minds Performance

Do not calculate:

```text
every user × every other user
```

That is O(n²).

Instead:

```text
pgvector nearest neighbor
+
interest filtering
+
small candidate set
```

Then rank those candidates.

---

# 109. Feed Performance

The first performance architecture should be:

```text
DB candidate query
      ↓
small candidate set
      ↓
ranking in application
      ↓
cache
```

Not:

```text
load 1 million posts
      ↓
AI rank all 1 million
```

---

# 110. AI Cost Control

AI processing should be staged.

### Cheap first

```text
hash
rules
SQL
```

### Then small models

```text
classification
embeddings
```

### Then expensive reasoning

```text
complex claims
summaries
```

This is the core cost strategy.

---

# 111. What Happens When the Platform Grows

The architecture can evolve:

```text
Stage 1
Modular monolith
        ↓
Stage 2
Dedicated AI workers
        ↓
Stage 3
Dedicated ranking service
        ↓
Stage 4
Dedicated search/vector service if needed
        ↓
Stage 5
Selective microservices
```

Do not begin at Stage 5.

---

# 112. What I Would Actually Build First

The first engineering milestone should contain exactly this:

```text
1. Next.js app
2. Supabase authentication
3. profiles
4. posts
5. comments
6. reactions
7. follows
8. basic feed
9. topics
10. entities
11. composition telemetry
12. basic AI classification
13. basic AI-risk rules
14. provenance hash
15. admin moderation panel
16. daily contribution calculation
17. points ledger
```

Then:

```text
18. pgvector
19. Similar Minds
20. Different Perspectives
21. semantic search
22. entity pages
23. advanced AI moderation
24. external provenance timestamping
25. reward marketplace
```

---

# 113. MVP Request Flow Example

### User posts

```text
Browser
  │
  ▼
Cloudflare
  │
  ▼
Next.js API
  │
  ├── Auth
  ├── Rate limit
  ├── Validate
  ├── Hash
  │
  ▼
Postgres
  │
  ├── posts
  ├── post_versions
  ├── provenance
  └── composition_sessions
  │
  ▼
Response immediately
  │
  ▼
Queue
  │
  ├── AI classification
  ├── AI risk
  ├── topics
  ├── entities
  ├── embedding
  ├── moderation
  └── ranking
```

---

# 114. Frontend Component Architecture

```text
components/
│
├── composer/
│   ├── PostComposer
│   ├── CompositionTelemetry
│   ├── ComposerActions
│   └── UploadButton
│
├── posts/
│   ├── PostCard
│   ├── PostContent
│   ├── PostReactions
│   ├── PostSource
│   ├── PostProvenance
│   └── PostShare
│
├── comments/
│   ├── CommentList
│   ├── CommentItem
│   └── CommentComposer
│
├── discovery/
│   ├── SimilarMinds
│   ├── DifferentPerspectives
│   ├── Trending
│   └── Topics
│
├── profile/
│   ├── ProfileHeader
│   ├── ContributionStats
│   ├── Interests
│   └── Reputation
│
└── entities/
    ├── EntityHeader
    ├── EntityReviews
    └── EntityDiscussions
```

---

# 115. Composer Architecture

The composer is especially important.

```text
PostComposer
│
├── editor
├── composition telemetry
├── media uploader
├── submit
└── submission state
```

No category selector.

No hashtag requirement.

No "choose post type".

---

# 116. Composer State Machine

```text
idle
  ↓
editing
  ↓
submitting
  ↓
published
  ↓
processing
  ↓
analyzed
```

Failure:

```text
submitting
   ↓
error
   ↓
retry
```

---

# 117. Smart UI Feedback

The AI should only interrupt the user when needed.

Examples:

```text
"Your post appears to contain a factual claim. Add a source?"
```

or:

```text
"People found this useful but several requested more detail."
```

Do not constantly display AI scores.

---

# 118. Important UX Rule

Never show:

```text
AI Risk: 78%
```

to normal users.

Never make users feel they need to prove they're human.

The goal is:

> quiet anti-abuse infrastructure.

---

# 119. Admin UI Is Where Scores Belong

Admins can see:

```text
AI risk
Spam risk
Automation risk
Safety risk
Originality
Duplicate similarity
```

Users generally should not.

---

# 120. Development Order

I would implement in this order:

## Phase 1

```text
project setup
auth
profiles
database
posts
comments
reactions
follows
```

## Phase 2

```text
feed
topics
entities
search
notifications
```

## Phase 3

```text
composition telemetry
behavior profiles
provenance
moderation
```

## Phase 4

```text
AI classification
AI risk
embeddings
Similar Minds
```

## Phase 5

```text
daily contribution
points
reputation
rewards
```

## Phase 6

```text
entity review pages
Different Perspectives
semantic search
```

---

# 121. Final Technology Boundary

The system should finally look like this:

```text
                         ┌──────────────────┐
                         │     BROWSER      │
                         │   React / Next   │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ CLOUDFLARE EDGE  │
                         │ CDN / Turnstile  │
                         │ Rate Limiting    │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │    NEXT.JS API   │
                         │  Modular Server  │
                         └────────┬─────────┘
                                  │
         ┌────────────────────────┼───────────────────────┐
         │                        │                       │
         ▼                        ▼                       ▼
 ┌───────────────┐        ┌───────────────┐       ┌───────────────┐
 │   PostgreSQL  │        │  Cloudflare   │       │ Cloudflare R2 │
 │    Supabase   │        │    Queues     │       │    Storage    │
 └───────┬───────┘        └───────┬───────┘       └───────────────┘
         │                        │
         │                        ▼
         │                ┌──────────────────┐
         │                │   AI WORKERS     │
         │                │                  │
         │                │ Content          │
         │                │ Behavior         │
         │                │ Moderation       │
         │                │ Embeddings       │
         │                │ Provenance       │
         │                │ Ranking          │
         │                └────────┬─────────┘
         │                         │
         ├─────────────────────────┘
         │
         ▼
 ┌───────────────────────────────────────────┐
 │                 PGVECTOR                  │
 │                                           │
 │ Similar Minds                             │
 │ Different Perspectives                    │
 │ Semantic Search                           │
 │ Duplicate Detection                       │
 └───────────────────────────────────────────┘
```

---

# 122. The Most Important Engineering Principle

There should be a strict distinction between:

### Source of truth

```text
PostgreSQL
```

### Derived data

```text
AI classifications
embeddings
ranking
similarity
reputation aggregates
```

### Temporary/cache data

```text
Cloudflare Cache
KV
```

### External verification

```text
timestamping / blockchain
```

If a derived system disappears, the application should be able to rebuild it from Postgres.

That property is extremely important.

---

# 123. Final Stack

### Frontend

```text
Next.js
React
TypeScript
Tailwind
PWA
```

### API/backend

```text
Next.js Route Handlers
Server Actions
Service Layer
Repository Layer
Cloudflare Workers
```

### Database

```text
PostgreSQL
Supabase
RLS
pgvector
```

### Storage

```text
Cloudflare R2
```

### Async

```text
Cloudflare Queues
Cron Triggers
```

### AI

```text
Provider abstraction
Workers AI
Local Ollama/llama.cpp
Embedding model
Future ML models
```

### Security

```text
Supabase Auth
RLS
Turnstile
Rate limiting
Input validation
Content sanitization
```

### Testing

```text
Vitest
Playwright
ESLint
Prettier
TypeScript
GitHub Actions
```

---

# 124. One-Sentence Architecture

> **Next.js is the user-facing application, Cloudflare is the edge and async platform, PostgreSQL is the source of truth, pgvector is the intelligence memory, and the AI layer is a replaceable set of workers that understand everything the user posts.**

# 125. Most Important Database Decision

Do **not** try to make one giant `posts` table contain everything.

Keep:

```text
posts
post_analysis
post_topics
post_entities
post_embeddings
post_provenance
post_metrics
post_sources
post_media
post_versions
```

separate.

That gives you a simple core object while allowing the intelligence system to evolve independently.

# 126. Most Important Backend Decision

Do **not** make AI processing part of the synchronous posting request.

The user's request should be:

```text
validate
→ save
→ hash
→ queue
→ respond
```

Everything intelligent happens afterward.

This gives you a fast platform even when your AI models become more sophisticated.

# 127. Most Important Product/Engineering Decision

The user interacts with:

```text
ONE POST COMPOSER
```

The system internally handles:

```text
classification
topic
entity
source
moderation
AI-risk
spam
provenance
similarity
ranking
reputation
reward
```

That is the technical implementation of your core product philosophy:

> **The user supplies the thought. The platform handles the complexity.**
