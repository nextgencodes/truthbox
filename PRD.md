# Product Requirements Document

# IDEA-FIRST

## AI-Native Social Network for Human Ideas, Useful Information, and Meaningful Discussion

**Document version:** 1.0
**Status:** Final Product Draft
**Platform:** Web first, PWA-ready, native mobile later
**Initial infrastructure target:** ₹0/month
**Primary interface:** One intelligent post composer
**Core principle:** **User writes. Platform understands.**

---

# 1. Executive Summary

This product is a social network designed around **ideas rather than personal life**.

Users come to share:

* opinions
* discoveries
* useful information
* news
* analysis
* questions
* product experiences
* software reviews
* company observations
* projects
* research
* lessons
* warnings
* recommendations
* discussions

The platform intentionally avoids becoming another feed of:

* "look at me"
* lifestyle updates
* status signaling
* personal-brand spam
* engagement bait
* mass-produced AI posts

The central product philosophy is:

> **Share what you think, not your lifestyle.**

The platform is **AI-native**. Users should not need to understand the site's internal structure.

They should not have to manually choose:

* category
* post type
* hashtags
* topic
* sentiment
* source type
* whether their post is a review
* whether it is a question
* whether it is a news item

They simply write.

The platform's AI systems interpret the content and automatically determine what it is, where it belongs, how trustworthy it appears, whether it is useful, whether it is potentially AI-generated or automated, how it should be moderated, and how it should be distributed.

The product should feel almost like **vibe coding for social media**:

> The user supplies the raw thought.
> The platform handles the complexity.

---

# 2. Core Product Thesis

Current social networks primarily optimize around:

> **Who you are → What you post → How much attention you receive.**

This product should optimize around:

> **What you contribute → How useful it is → What discussion it creates → Who can benefit from it.**

The central loop is therefore:

```text
Think
  ↓
Write
  ↓
Platform understands
  ↓
Community discusses
  ↓
Useful information spreads
  ↓
Contributor receives impact
  ↓
Contributor builds reputation
```

Not:

```text
Post constantly
  ↓
Get attention
  ↓
Post more
  ↓
Become an influencer
```

---

# 3. Product Mission

### Mission

Create a social network where **human ideas, useful information, honest experiences, and constructive disagreement are more valuable than personal branding and content volume.**

### Long-term vision

Become a network where people can:

* discover useful ideas
* find people with similar interests
* find people with different perspectives
* discover trustworthy experiences
* evaluate products and services through community knowledge
* share information without becoming influencers
* build reputation through contribution rather than lifestyle
* preserve attribution to original contributors

---

# 4. Core Principles

## 4.1 Ideas over lifestyle

The platform is not designed around:

> "What did you do today?"

It is designed around:

> "What did you learn, discover, think about, test, question, or understand?"

---

## 4.2 Contribution over attention

A post does not become valuable merely because it receives many impressions.

The system should ask:

> Did this help someone?

---

## 4.3 Quality over quantity

Users may post as much as they want within platform limits.

However:

> **Only one contribution per user per reward period is eligible for the primary daily reward.**

The system analyzes all posts but selects the user's strongest contribution for reward purposes.

Therefore, publishing 50 posts does not provide 50 opportunities to farm rewards.

---

## 4.4 Human thought over synthetic volume

The platform does not need to prove that every sentence was written entirely without AI.

Instead, it should strongly discourage:

* bulk AI posting
* automated content farms
* generated engagement bait
* fake expertise
* repetitive generated opinions
* mass-generated news summaries

AI-assisted writing is not automatically prohibited.

The primary target is **low-effort synthetic volume**.

---

## 4.5 The platform understands the user

The user should not need to explain their content to the platform.

The platform should infer:

* topic
* intent
* content type
* entities
* claims
* emotional tone
* usefulness
* potential risk
* originality
* potential AI-generation
* source relevance
* relationship to previous posts
* likely audience

---

## 4.6 Useful disagreement is valuable

The platform should not reward only agreement.

A thoughtful disagreement can be more valuable than a like.

The system should encourage:

* disagreement
* requests for evidence
* requests for more detail
* corrections
* alternative perspectives

---

## 4.7 People should discover minds, not just profiles

The platform should help people discover:

* people with similar interests
* people working on similar things
* people discussing related subjects
* people who disagree constructively
* people who have complementary expertise

---

# 5. Target Audience

## Primary audience

People who enjoy:

* technology
* AI
* science
* software
* startups
* business
* gaming
* movies
* finance
* education
* research
* culture
* current events
* products and services
* thoughtful discussion

## Secondary audience

* developers
* students
* researchers
* founders
* engineers
* analysts
* journalists
* creators
* product users
* hobbyists
* domain experts

---

# 6. Product Positioning

Possible conceptual positioning:

### Traditional social media

**"What are you doing?"**

### Professional social media

**"What are you doing professionally?"**

### Generic discussion platforms

**"What does this community think?"**

### This platform

> **"What do you think, what have you learned, and what can other people gain from it?"**

Supporting phrase:

> **Share what you think. Not your lifestyle.**

Additional product statement:

> **You post. We understand.**

Long-term identity:

> **A social network for human ideas.**

---

# 7. What the User Sees

The interface should be extremely simple.

## Home

```text
------------------------------------------------
LOGO

For You     Following     Trending
------------------------------------------------

What's on your mind?

[ Write something...                         ]

                                            Post
------------------------------------------------

@alex
AI coding agents are changing...

Helpful   Not Helpful   More Detail   Disagree
Comment   Share
------------------------------------------------
```

The user does not choose:

* AI
* software
* opinion
* analysis
* topic
* hashtag

The AI handles those classifications.

---

# 8. The Intelligent Composer

The composer is the most important user-facing component.

## Primary UX

```text
What's on your mind?

[............................................]

                              [ Post ]
```

That's it.

The system should progressively become more intelligent without making the interface complicated.

---

# 9. Composer Behavior

While the user writes, the platform records privacy-conscious behavioral signals needed for:

* anti-bot detection
* automation detection
* composition analysis
* personalization
* AI-risk estimation
* abuse detection

Signals may include:

* composition start time
* composition end time
* active typing duration
* pauses
* typing bursts
* edits
* deletions
* replacement events
* paste occurrence
* paste size
* undo/redo behavior
* focus changes
* text growth
* publication timing

The system should not store arbitrary clipboard contents or a permanent raw keystroke history.

The preferred representation is derived telemetry.

Example:

```text
composition_time: 184 sec
active_typing_time: 142 sec
characters: 487
paste_events: 0
deletion_ratio: 0.14
revision_ratio: 0.21
pause_count: 8
```

---

# 10. Important AI-Detection Principle

The platform must never operate on:

> typing speed = human
> fast typing = AI

That is too simplistic.

Typing speed is only one signal.

A robust system combines:

```text
Composition behavior
+
Editing behavior
+
Paste behavior
+
Posting behavior
+
Account behavior
+
Content similarity
+
Language signals
+
Historical user behavior
+
Known automation indicators
```

The resulting output should be:

> **automation / AI-generation risk**

not an absolute declaration:

> **This user is using AI.**

---

# 11. AI Moderator

The AI Moderator is the invisible operating system of the platform.

It is responsible for understanding, organizing, protecting, and ranking content.

It is not one model.

It is an orchestration layer containing multiple specialized analyzers.

## High-level architecture

```text
                         USER
                           │
                           ▼
                      POST COMPOSER
                           │
                           ▼
                  ┌──────────────────┐
                  │  AI ORCHESTRATOR │
                  └────────┬─────────┘
                           │
       ┌───────────────────┼─────────────────────┐
       │                   │                     │
       ▼                   ▼                     ▼
 CONTENT ANALYZER    BEHAVIOR ANALYZER    PROVENANCE ENGINE
       │                   │                     │
       ▼                   ▼                     ▼
 topic / intent       typing / automation     originality
 claims               spam                    lineage
 entities             account behavior        duplicates
       │                   │                     │
       └───────────────────┼─────────────────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ DECISION ENGINE  │
                  └────────┬─────────┘
                           │
         ┌─────────────────┼─────────────────┐
         ▼                 ▼                 ▼
      PUBLISH            LIMIT             REVIEW
         │
         ▼
                   FEED RANKING
```

---

# 12. Content Understanding

For every post, the AI should infer internally:

### Topic

Examples:

* AI
* software
* science
* finance
* business
* gaming
* entertainment
* education
* politics
* cybersecurity
* hardware
* startups

Potentially thousands of topics.

---

## Intent

The platform may internally classify:

* opinion
* news
* analysis
* question
* discovery
* warning
* recommendation
* review
* project
* experiment
* experience
* explanation
* discussion
* announcement

Users are not required to select these.

---

## Entity detection

The AI identifies entities:

```text
Cursor
OpenAI
Microsoft
React
Python
Tesla
Reddit
iPhone
Game X
Movie Y
Company Z
```

The platform can attach a post automatically to the relevant entity page.

---

## Claim detection

The system determines whether the post contains:

* subjective opinion
* factual claim
* numerical claim
* personal experience
* recommendation
* potentially harmful accusation
* news claim
* scientific claim

This determines which additional systems should become active.

---

# 13. Automatic Topic Mapping

No mandatory hashtags.

Example:

User writes:

> "Claude Code is currently more useful to me than Cursor for modifying large repositories."

The platform internally assigns:

```text
AI
Developer Tools
Software Development
Claude
Cursor
Coding Agents
```

The user does not have to tag anything.

---

# 14. Automatic Personal-Life Detection

The AI moderator determines whether a post is primarily:

### Personal-status content

> "Had an amazing dinner today."

versus:

### Topic-relevant personal experience

> "I used five coding agents over the last month and this is what surprised me."

The second should be allowed.

The system should judge **whether the personal detail contributes useful information**.

The purpose is not to ban personal experiences.

It is to avoid turning the platform into a lifestyle feed.

---

# 15. AI-Generated Content Detection System

## Goal

Reduce:

* generated spam
* AI content farms
* bulk post generation
* fake expertise
* engagement bait
* automated accounts

## Do not attempt

Perfect attribution of every sentence to an AI provider.

That is not realistic.

---

# 16. AI-Risk Signals

## Behavioral signals

* huge paste
* extremely short composition time
* zero editing
* repeated posting
* high posting frequency
* synchronized behavior
* unusual session patterns
* abrupt behavioral changes
* automated interaction patterns

## Content signals

* duplicated phrases
* near-duplicate posts
* templated language
* repetitive structures
* repeated introductions
* repeated conclusions
* unusual similarity between many accounts
* generated-content patterns

## Account signals

* account age
* posting velocity
* topic switching
* repeated activity windows
* interaction ratio
* number of identical posts
* cross-account similarities

---

# 17. Personal Writing Baseline

The platform should eventually learn each user's normal behavioral baseline.

Example:

```text
USER BASELINE

Typical post:
200–700 chars

Typical composition:
2–6 minutes

Typical editing:
Moderate

Typical daily posts:
2–5

Typical pause pattern:
Normal

Typical paste:
Rare
```

A sudden deviation becomes a signal.

Example:

```text
Typical:
400 chars / 3 minutes

Current:
2,000 chars / 3 seconds
```

This does not mean the post is automatically blocked.

It increases the risk score.

---

# 18. AI-Risk Score

Each post receives:

```text
ai_generation_risk = 0–100
```

Initial implementation can be rule-based.

Later it can become a machine-learning model.

Example placeholder signals:

```text
large paste                +25
extreme composition speed  +20
mass posting               +20
duplicate content          +20
template repetition        +15
abnormal account behavior  +20
```

The actual weights are configurable and not exposed to users.

---

# 19. Risk-Based Actions

## Low risk

Publish normally.

## Medium risk

Publish normally, but possibly reduce recommendation weight.

## High risk

Apply additional anti-abuse friction or distribution limits.

## Very high risk

Potentially hold for moderation.

No permanent user punishment should occur solely because a detector thinks text may be AI-generated.

---

# 20. Human Override

Moderators need a dashboard showing:

```text
Post
Risk Score
Risk Signals
Account history
Posting velocity
Similar content
Community reports
```

Available actions:

* approve
* limit
* remove
* restore
* restrict account
* mark as false positive

Moderator decisions should become training/evaluation data later.

---

# 21. Community Feedback

Users do not simply "like" or "dislike."

Primary reactions:

### Helpful

> This gave me useful information.

### Not Helpful

> This did not provide value.

### More Details

> Interesting, but I need evidence/context/explanation.

### Disagree

> I disagree with the conclusion.

### Curious

> I want to learn more.

Optional:

### Share

Sharing remains a distribution action rather than a reaction.

---

# 22. Why These Reactions Matter

A traditional like provides limited information.

The new reaction system creates semantic feedback.

Example:

```text
Helpful       820
More Details  310
Disagree      192
Not Helpful    20
```

This tells both:

* the author
* the AI system

much more than:

```text
1,342 likes
```

---

# 23. AI Response to Community Feedback

The AI can detect patterns.

Example:

A post receives:

```text
Helpful: 400
More Details: 350
Disagree: 80
```

The system can infer:

> High interest, but readers want more support.

The platform may automatically offer the author:

> **People found this interesting but requested more detail. Add context?**

This should be optional.

---

# 24. Helpful Reactions as a Ranking Signal

A "Helpful" reaction should generally carry more weight than a lightweight reaction.

Potential hierarchy:

```text
View
   ↓
Read
   ↓
Helpful
   ↓
Save
   ↓
Share
   ↓
Meaningful reply
```

The exact weights should be learned later.

---

# 25. Save / Bookmark

Allow users to save useful posts.

A save is an important long-term value signal.

Potential feed ranking should consider:

* helpful
* saves
* shares
* meaningful replies

rather than relying almost entirely on likes.

---

# 26. Comments

Comments should be treated as first-class contributions.

A strong comment can be more valuable than the original post.

The AI can identify comments that:

* add evidence
* correct information
* provide examples
* introduce alternative viewpoints
* answer questions
* provide resources

Those comments should gain contribution reputation.

---

# 27. Discussion Quality

The system should distinguish:

### High-quality discussion

> "I agree with this, but benchmark X shows a different result. Here's the paper."

from:

### Low-quality engagement

> "Facts 🔥"

> "This."

> "Lmao."

Short comments aren't automatically bad, but the system should not give disproportionate reward for shallow interaction.

---

# 28. Daily Contribution Reward

This is a core economic/product mechanism.

## Rule

> **Only one contribution per user per reward period is eligible for the primary reward.**

For example, daily:

```text
User posts 1 → evaluated
User posts 5 → all evaluated
User posts 50 → all evaluated

But:

Only the highest-scoring eligible contribution
is selected for the daily reward.
```

---

# 29. Why This Matters

It removes the incentive:

> More posts = more rewards.

A user cannot easily farm the system by generating 100 posts.

The question becomes:

> What was the most useful thing I contributed today?

---

# 30. Daily Contribution Selection

The AI evaluates each eligible contribution using multiple dimensions.

Potential internal factors:

```text
helpfulness
originality
discussion value
evidence
awareness
empathy
relevance
community response
usefulness
novelty
quality of disagreement
source quality
long-term value
```

It selects:

```text
Daily Contribution
```

rather than:

```text
Most viral post
```

---

# 31. Impact Score

Internally, the platform can calculate:

```text
impact_score = f(
    helpfulness,
    useful_feedback,
    meaningful_discussion,
    saves,
    shares,
    originality,
    evidence,
    awareness,
    empathy,
    source_quality,
    novelty,
    user_trust,
    spam_penalty,
    automation_penalty
)
```

The exact formula should remain flexible.

---

# 32. Important Reward Principle

Do not publish:

> +10 points for empathy
> +5 points for sources
> +3 points for comments

If users know the exact formula, they will optimize for it.

Instead:

> **The AI evaluates overall contribution quality.**

The exact scoring model can evolve.

---

# 33. Contribution Points

The daily contribution can generate points.

Example:

```text
Daily Contribution
       ↓
Impact Score
       ↓
Contribution Points
```

Points can later be used for:

* badges
* profile customization
* feature unlocks
* early product access
* community privileges
* beta programs
* event access
* coupons
* software discounts
* credits

---

# 34. Rewards Marketplace

The initial MVP does not need real rewards.

Build the ledger first.

Later:

```text
Contribution Points
        ↓
Rewards Marketplace
        ↓
Software discounts
Developer credits
Courses
Events
Subscriptions
Products
Partner offers
```

Companies can eventually sponsor rewards.

This means the platform does not need to finance all rewards itself.

---

# 35. Company and Software Reviews

The platform should support reviews automatically.

The user does not select:

> "This is a review."

They simply write:

> "I've used Cursor for six months. It's excellent for..."

The AI understands:

```text
Entity: Cursor
Content: Review
Topic: Developer Tools
Experience: Personal
```

The post can automatically appear on the **Cursor page**.

---

# 36. Entity Pages

The platform should automatically create pages for commonly discussed entities.

Examples:

```text
OpenAI
Microsoft
Cursor
GitHub
React
Python
Tesla
iPhone
Notion
Netflix
```

Entity pages contain:

* discussions
* reviews
* opinions
* questions
* news
* useful posts
* trends
* recurring complaints
* positive feedback
* alternative perspectives

---

# 37. Reviews Must Preserve Context

Instead of:

```text
Cursor
★★★★☆
```

the platform should emphasize:

> **Why did this person think this?**

Example:

> "Great for refactoring, but agent mode becomes unreliable on very large repositories."

This creates searchable, contextual community knowledge.

---

# 38. Verified Experience

Later, users can receive contextual markers such as:

* Used recently
* Long-term user
* Developer experience
* Customer experience
* Project experience
* Verified purchase
* Verified project

These markers should never be claimed without a legitimate verification mechanism.

---

# 39. Commercial Disclosure

Users should disclose relevant relationships when applicable.

Examples:

* employee
* founder
* paid reviewer
* sponsored access
* affiliate relationship
* gifted product

The AI can detect potential conflicts and request disclosure where appropriate.

---

# 40. Company Feedback

The platform should not become purely positive/negative review spam.

Companies can eventually:

* claim their entity page
* respond to reviews
* correct factual information
* provide official statements
* publish updates

Responses must be visibly identified as company representatives.

---

# 41. Similar Minds

One of the platform's major discovery features.

The AI learns a user's interest graph based on:

* posts
* reading behavior
* comments
* saved content
* topics
* discussions
* projects
* reviews
* recurring subjects
* positive engagement
* negative engagement
* long-term interest

The user does not fill out a personality questionnaire.

---

# 42. Similar Minds Feature

Profile:

```text
SIMILAR MINDS

@alex      91% interest overlap
@rahul      87%
@maria      84%
@sam        81%
@john       79%
```

These percentages should be described as similarity estimates, not psychological facts.

---

# 43. Similarity Dimensions

Similarity may consider:

```text
topic overlap
interest overlap
reading overlap
discussion overlap
project overlap
entity overlap
content preference
activity patterns
```

The system can eventually use embeddings for content and interests.

---

# 44. Working On Similar Things

A separate discovery surface:

## People Working on Similar Things

Example:

> You frequently discuss small local LLMs.

The platform finds:

```text
@alex
Building a local assistant

@john
Optimizing LLMs for edge hardware

@maria
Working on Raspberry Pi inference
```

This is more useful than generic:

> People you may know.

---

# 45. Different Perspectives

The system should intentionally expose valuable disagreement.

Example:

```text
SIMILAR MINDS
People with overlapping interests

DIFFERENT PERSPECTIVES
People who frequently disagree with your views
but care about the same subjects
```

The goal is not political polarization.

The goal is:

> **Different conclusions about shared subjects.**

---

# 46. Perspective Matching

Potential dimensions:

```text
similar interests
different opinions
shared topics
constructive interaction history
high-quality disagreement
```

The platform should avoid recommending accounts purely because they are controversial.

It should prioritize constructive difference.

---

# 47. User Interest Graph

Internally:

```text
                  USER
                   │
       ┌───────────┼───────────┐
       │           │           │
      AI        Robotics    Software
       │           │           │
    Agents       Edge AI    Dev Tools
       │           │           │
    LLMs       Hardware      GitHub
```

This graph powers:

* feed
* Similar Minds
* Different Perspectives
* topic discovery
* entity pages
* recommendations

---

# 48. Feed Design

Three primary views:

## For You

AI-personalized.

## Following

People/topics/entities followed by the user.

## Trending

High-quality discussions with unusual recent activity.

Potential future surfaces:

* Similar Minds
* Different Perspectives
* Discoveries
* Reviews
* Questions

---

# 49. Feed Ranking

Ranking should consider:

```text
topic relevance
user interest
helpful reactions
meaningful comments
saves
shares
discussion depth
originality
freshness
source quality
account reputation
spam risk
automation risk
duplicate risk
```

Pure engagement should not dominate.

---

# 50. Engagement Quality

The feed should distinguish:

### High-quality

* useful comments
* detailed disagreement
* evidence
* saves
* helpful reactions
* meaningful shares

### Low-quality

* repetitive comments
* engagement bait
* bot replies
* mass reactions
* copied posts
* artificial activity

---

# 51. Provenance System

The platform should automatically establish internal content provenance.

For each post:

```text
post_id
creator_id
created_at
content_hash
media_hash
previous_record_hash
```

This creates a tamper-evident internal record.

---

# 52. Original Post Recognition

When someone submits content similar to existing content:

```text
New post
   ↓
Similarity engine
   ↓
Existing content?
   ↓
Yes
   ↓
Determine relationship
```

Possible relationship:

* exact repost
* near-duplicate
* quote
* commentary
* derivative content
* independent similar thought

The platform should not automatically assume plagiarism.

---

# 53. Attribution

If a user shares an existing post:

```text
Originally posted by @alex
```

should remain visible.

If the second user adds original commentary:

```text
@bob
"This is interesting because..."

Originally posted by @alex
```

The second user gets credit for their commentary without erasing the first creator.

---

# 54. Blockchain Strategy

Blockchain should initially be **infrastructure**, not product branding.

Do not require:

* wallet
* token
* cryptocurrency
* transaction fees
* NFTs

for normal users.

Initial system:

```text
content
 ↓
hash
 ↓
hash chain
```

Later:

```text
batch hashes
 ↓
Merkle tree
 ↓
Merkle root
 ↓
external timestamping
```

The platform can eventually use a public timestamping mechanism such as OpenTimestamps.

---

# 55. What Blockchain Can and Cannot Prove

The platform can establish:

> This content was registered on this platform at this time.

It cannot prove:

> This person invented the idea in the world.

The UI should therefore use:

> **First registered on this network**

rather than:

> **Original inventor.**

---

# 56. AI Provenance

The platform should eventually support known AI-generated content signals such as provider provenance/watermarks when technically available.

This remains a supporting signal.

It should never be the only detector.

---

# 57. Moderation

The AI moderator continuously evaluates:

* spam
* harassment
* threats
* scams
* misinformation indicators
* impersonation
* malicious links
* commercial spam
* copied content
* synthetic account behavior
* abusive behavior

The system should support:

```text
allow
limit
label
request context
hold
remove
account restrict
```

---

# 58. Claims About Companies and People

Because the platform allows opinions and reviews, it needs an elevated policy for factual accusations.

The system should distinguish:

### Opinion

> "I think this company has poor customer support."

### Experience

> "I contacted support three times and received no response."

### Serious factual allegation

> "This company is committing fraud."

The third category needs significantly stronger moderation and evidence handling.

---

# 59. News and Factual Claims

The platform should automatically detect likely news statements.

Example:

> "Company X announced Y today."

The system can identify:

```text
factual claim
entity
date
potential source requirement
```

Later the platform can integrate source retrieval and claim verification.

The user should not need to manually categorize the post.

---

# 60. Source Handling

Users can simply paste a URL naturally:

> "Interesting development from Google: https://..."

The system automatically creates:

```text
Source card
Title
Publisher
Publication date
Relevant topic
```

The user does not need to fill a citation form.

---

# 61. AI Moderator Feedback to User

The AI can intervene only when useful.

Examples:

### Unsupported factual statement

> "This looks like a factual claim. Would you like to add a source?"

### Possible duplicate

> "This looks similar to an existing post. Would you like to reference it?"

### Community asks for evidence

> "Several readers requested more detail."

### Potential personal-life content

> "This looks mostly like a personal status update. Consider adding what others can learn from it."

The user can still publish where policy permits.

---

# 62. AI Should Feel Helpful, Not Policing

The AI moderator should not constantly interrupt.

The principle is:

> **Invisible by default. Helpful when necessary.**

Bad:

> "You classified this incorrectly."

Good:

> "This looks like a software review. I've added it to the relevant topic automatically."

---

# 63. Reputation System

No single follower count should define reputation.

The system should track contribution dimensions.

Possible profile signals:

```text
Original contributions
Useful contributions
Helpful reactions received
Discussion contributions
Reviews
Discoveries
Evidence-supported posts
Long-term topic expertise
Community trust
```

Do not display an overly complicated numerical score initially.

---

# 64. Reputation Should Be Topic-Specific

A user might have strong reputation in:

```text
AI
Software Development
```

and little reputation in:

```text
Finance
Medicine
Law
```

The system should avoid implying universal expertise.

---

# 65. Daily Contribution Profile

A profile could show:

```text
THIS MONTH

31 daily contributions
18 highly helpful
12 useful discoveries
9 detailed discussions
4 software reviews
```

This supports the identity:

> **Contributor**

rather than:

> **Influencer**

---

# 66. Rewards Must Avoid Spam

Hard rule:

> Posting more should not linearly increase reward.

A user posting 100 times does not get 100 reward opportunities.

Instead:

```text
All posts
  ↓
Scoring
  ↓
Best eligible contribution
  ↓
One daily reward
```

---

# 67. Reward Eligibility

Potential disqualifying conditions:

* spam
* duplicate posting
* automated account behavior
* policy violations
* manipulation
* artificial engagement
* proven copied content presented as original

The system should use grace periods and appeals for uncertain cases.

---

# 68. Rewards Evolution

### Phase 1

No financial rewards.

Users receive:

* contribution points
* profile badges
* recognition
* contribution history
* feature access

### Phase 2

Partner rewards:

* software discounts
* subscriptions
* developer credits
* educational discounts
* conference access

### Phase 3

Commercial ecosystem:

* sponsored rewards
* company challenges
* beta access
* community testing
* research programs

---

# 69. Partner Bounties

Future example:

A software company wants feedback from developers.

They create:

> "Test our API and share your experience."

The platform provides participating users:

* free credits
* beta access
* discounts

The content remains subject to disclosure requirements.

The company cannot buy positive reviews.

---

# 70. Product / Company Research

The platform could eventually offer companies aggregated insight:

```text
What users say about product X
Most common complaints
Most requested features
Positive themes
Negative themes
Emerging issues
```

Only aggregated / appropriately privacy-protected insights should be sold.

---

# 71. Gamification

Gamification should reinforce:

> contribution

not:

> posting addiction.

Potential badges:

* Helpful Contributor
* Great Explainer
* Useful Reviewer
* Source Finder
* Constructive Challenger
* Early Discoverer
* Community Helper

Avoid badges such as:

* Most Active Poster
* 100 Posts Today

Those encourage quantity.

---

# 72. Notifications

Useful notifications:

> Someone marked your post Helpful.

> Someone requested more detail.

> Your post was referenced in a discussion.

> Someone with similar interests posted something you may like.

> You and @alex have overlapping interests.

> A company you follow has responded to a discussion.

Avoid excessive:

> Someone liked something.

notifications.

---

# 73. Search

Search should work across:

* posts
* users
* topics
* companies
* products
* software
* discussions

Eventually support semantic search:

> "Find posts from developers who have actually used local LLMs."

The platform can search based on meaning rather than exact words.

---

# 74. Similar Minds Search

Future natural-language search:

> "Find people interested in edge AI and tiny language models."

The AI returns relevant people based on actual contribution history.

---

# 75. Content Discovery Search

Example:

> "Show me useful discussions about Cursor from developers."

The platform combines:

* entity
* topic
* user behavior
* content intent
* experience signals
* usefulness

---

# 76. Profile Design

The profile should emphasize:

```text
@username

Interested in:
AI · Robotics · Software

Currently exploring:
Local models · Agents

Contributions:
Original posts
Reviews
Discoveries
Discussions

Similar Minds
Different Perspectives
```

Avoid making the biography the center of the profile.

---

# 77. Following Model

Users may follow:

* people
* topics
* entities
* discussions

They should not be forced into person-first social graphs.

Examples:

> Follow AI

> Follow OpenAI

> Follow Cursor

> Follow @alex

---

# 78. Following Topics

Topic follow is critical.

Someone can build a rich feed without following hundreds of personalities.

Example:

```text
Following:

AI
Robotics
Open Source
Developer Tools
Science
```

---

# 79. Discussions as Objects

A high-quality discussion should become a persistent object.

Example:

```text
Question
 ↓
50 responses
 ↓
3 major viewpoints
 ↓
Best evidence
 ↓
Community conclusion
```

The AI can summarize long discussions.

---

# 80. AI Discussion Summary

For long threads:

```text
Discussion Summary

Main argument:
...

Arguments supporting:
...

Arguments against:
...

Evidence:
...

Unresolved questions:
...
```

This allows users to enter a discussion without reading 500 comments.

---

# 81. AI Does Not Replace Human Discussion

The AI summarizes.

Humans generate the actual viewpoints.

The AI should not flood the thread with its own fake opinions.

---

# 82. AI-Generated Replies

Initially, users should not be able to deploy autonomous bots that reply at scale.

Future AI-assisted replies may exist, but must clearly distinguish:

* human
* AI-assisted
* automated agent

The platform should avoid synthetic users overwhelming humans.

---

# 83. Anti-Bot Architecture

Use:

* Cloudflare Turnstile
* rate limiting
* account velocity monitoring
* interaction pattern detection
* duplicate detection
* suspicious session detection

No need for expensive third-party services initially.

---

# 84. Privacy Model

The platform may analyze activity occurring **inside the platform**.

It must not monitor:

* unrelated websites
* unrelated browser activity
* files on the user's device
* applications outside the platform
* passwords
* unrestricted clipboard history

Typing telemetry should be privacy-preserving.

Store derived behavioral features instead of unnecessary raw keystrokes.

The privacy policy must clearly explain telemetry.

---

# 85. Technical Architecture

## Frontend

Recommended:

```text
Next.js
TypeScript
Tailwind CSS
PWA support
```

---

## Backend

Initial:

```text
Next.js server/API
or
Cloudflare Workers
```

---

## Database

```text
PostgreSQL
Supabase
```

---

## Authentication

```text
Supabase Auth
Google
GitHub
Email
```

---

## Bot Protection

```text
Cloudflare Turnstile
```

---

## CDN / Infrastructure

```text
Cloudflare
```

The initial system should target a **free-tier MVP**.

Free-tier quotas and commercial-use terms must be rechecked before production launch because provider policies can change.

---

# 86. No-Cost MVP Media Strategy

Do not initially become a video-hosting company.

MVP content:

* text
* links
* small images

For external video:

* YouTube
* other externally hosted media URLs

Large-scale video hosting should be deferred.

---

# 87. Core Backend Services

The codebase should be modular.

```text
/auth
/posts
/comments
/reactions
/topics
/entities
/feed
/search
/moderation
/ai
/telemetry
/risk
/provenance
/reputation
/rewards
/notifications
/admin
```

---

# 88. AI Service Interfaces

Define interfaces from day one.

```text
ContentAnalyzer

BehaviorAnalyzer

AIRiskDetector

SpamDetector

SafetyAnalyzer

ProvenanceAnalyzer

SimilarityEngine

EntityResolver

SourceAnalyzer

FeedRanker

RewardScorer

RecommendationEngine
```

Initial versions can be simple.

They should be replaceable.

---

# 89. AI Orchestrator

The AI Orchestrator decides which analyzer needs to run.

Example:

```text
new post
 ↓
content analysis
 ↓
entity detection
 ↓
topic analysis
 ↓
risk analysis
 ↓
provenance check
 ↓
ranking features
```

Not every post requires every expensive operation.

---

# 90. Model Strategy

Because the project initially has no money:

### MVP

Use:

* deterministic rules
* lightweight open-source models
* local inference where practical
* database heuristics
* embeddings only where feasible

Avoid relying on paid APIs.

### Future

Potentially introduce:

* stronger local LLM
* specialized classifiers
* embedding models
* multimodal models
* AI provenance systems
* larger ranking models

The interface to the rest of the application should stay unchanged.

---

# 91. Initial AI Model Responsibilities

The first version should prioritize:

### Small / cheap model

For:

* classification
* topic extraction
* entity extraction
* simple moderation

### Rules

For:

* rate limits
* duplicate detection
* paste behavior
* posting velocity

### Embeddings

Optional initially.

Useful later for:

* Similar Minds
* duplicate detection
* semantic search
* entity clustering
* discussion clustering

---

# 92. Database Model

## users

```text
id
username
display_name
email
created_at
status
reputation_state
```

## posts

```text
id
user_id
content
created_at
updated_at
visibility
status
content_hash
previous_hash
```

## post_analysis

```text
post_id
topics
entities
intent
personal_life_score
spam_score
ai_risk_score
originality_score
helpfulness_prediction
discussion_score
claim_score
analysis_version
created_at
```

## comments

```text
id
post_id
user_id
content
created_at
status
```

## reactions

```text
user_id
content_id
reaction_type
created_at
```

Reaction types:

```text
helpful
not_helpful
more_detail
disagree
curious
```

---

# 93. Behavioral Telemetry

## composition_sessions

```text
id
user_id
post_id
started_at
published_at
device_class
browser_class
```

## composition_features

```text
session_id
active_time
total_time
character_count
paste_count
paste_chars
delete_count
undo_count
redo_count
pause_count
editing_ratio
typing_rate_features
```

No unnecessary raw keystroke archive.

---

# 94. User Behavioral Profile

```text
user_id
typical_composition_time
typical_post_length
typical_editing_ratio
typical_post_velocity
typical_topic_distribution
baseline_version
updated_at
```

This profile should be statistical, not an identity/authentication system.

---

# 95. Provenance Tables

```text
content_provenance

post_id
content_hash
media_hash
previous_hash
registered_at
timestamp_proof
```

Future fields:

```text
merkle_root
external_timestamp
proof_uri
```

---

# 96. Similarity System

Potential tables:

```text
user_interest_embedding
post_embedding
entity_embedding
topic_embedding
```

Potential relationships:

```text
user → topic
user → entity
user → user
post → post
post → entity
```

The system should periodically recompute similarities as users' interests evolve.

---

# 97. Rewards

```text
daily_contributions

user_id
date
selected_post_id
impact_score
reward_points
status
```

```text
reward_ledger

user_id
transaction_type
points
reference_id
created_at
```

The ledger must be append-only.

---

# 98. Anti-Gaming Reward Rules

Detect:

* self-interaction
* engagement rings
* repeated reciprocal reactions
* fake accounts
* automated comments
* suspicious sharing
* coordinated voting
* mass-account behavior

Rewards should be reversible if manipulation is discovered.

---

# 99. Admin Dashboard

The admin system is mandatory from the first functional version.

It should allow:

### User search

```text
username
account age
risk state
activity
```

### Post search

```text
post ID
user
topic
risk
status
```

### AI analysis

```text
AI risk
spam risk
safety risk
originality
similarity
```

### Actions

```text
approve
remove
restrict
restore
reset score
mark false positive
```

---

# 100. AI Decision Explanation

For internal moderators:

```text
Decision: Distribution limited

Reasons:
- unusually high post velocity
- 98% similarity to previous post
- suspicious coordinated reactions
- large-scale automation indicators
```

Do not expose sensitive internal thresholds.

---

# 101. Logging

Every significant AI decision should be logged:

```text
decision_id
object_id
model_version
rules_version
input_feature_version
decision
confidence
timestamp
```

This is essential for debugging false positives.

---

# 102. Version Everything

AI-related systems must have versions.

Examples:

```text
content_analyzer_v1
risk_engine_v1
ranking_v1
reward_model_v1
```

When a model changes, old decisions remain traceable.

---

# 103. A/B Testing

Future experiments:

* different feed ranking
* different reaction weighting
* reward model
* Similar Minds algorithm
* Different Perspectives algorithm
* moderation thresholds

Never change all scoring simultaneously without logging the experiment.

---

# 104. Key Product Metrics

## Contribution quality

```text
helpful rate
save rate
share rate
meaningful comment rate
discussion depth
```

## Community health

```text
returning users
retention
unique contributors
new contributors
topic diversity
cross-user interaction
```

## AI/spam health

```text
suspected automation
confirmed automation
false positive rate
duplicate rate
spam rate
```

## Reward quality

```text
reward concentration
percentage of users earning rewards
repeat daily contributors
reward abuse
```

---

# 105. Most Important Metric

The platform should eventually optimize for:

## **Useful Contribution Rate**

Definition:

> Percentage of meaningful contributions that receive strong "Helpful", Save, Share, or constructive discussion signals.

This is more aligned with the product than raw daily active users.

---

# 106. Secondary North-Star Metric

## **People Helped**

Potentially estimated from:

* helpful feedback
* saved content
* useful shares
* repeated references
* successful follow-up interactions

Example:

> **You helped 48 people this week.**

This is psychologically different from:

> You got 48 likes.

---

# 107. Content Quality Metrics

Track:

```text original contribution ratio
duplicate ratio
AI-risk distribution
spam ratio
helpful ratio
more-detail ratio
disagreement ratio
source attachment rate
```

---

# 108. Growth Strategy Built Into Product

The product should naturally generate shareable URLs.

Every post:

```text
/post/[id]
```

Every entity:

```text
/entity/[slug]
```

Every topic:

```text
/topic/[slug]
```

Every public profile:

```text
/@username
```

This provides long-term SEO/discovery potential.

---

# 109. Public Content

Public posts should be indexable where appropriate.

Search engines can discover:

* useful discussions
* software reviews
* product opinions
* answers
* research discussions
* news conversations

This can become a major organic acquisition channel.

---

# 110. Avoid SEO Spam

Do not generate thousands of empty entity pages.

Only create rich public pages when sufficient real content exists.

---

# 111. MVP Scope

## Must build

### Authentication

* email
* Google
* GitHub

### Social

* profile
* posting
* commenting
* sharing
* following

### AI

* topic detection
* intent detection
* entity detection
* personal-life classification
* spam detection
* basic AI-risk scoring

### Behavioral

* composition telemetry
* paste signals
* editing signals
* velocity signals

### Engagement

* Helpful
* Not Helpful
* More Details
* Disagree
* Curious
* Save
* Share

### Discovery

* For You
* Following
* Trending
* Similar Minds

### Trust

* provenance hash
* duplicate detection
* reporting
* admin panel

### Rewards

* daily best contribution selection
* points ledger
* basic contribution history

---

# 112. MVP Placeholder Systems

Architect now, implement simply initially:

* advanced AI detector
* source verification
* external timestamps
* blockchain anchoring
* advanced semantic similarity
* learned ranking model
* advanced recommendation
* verified purchases
* company dashboards
* sponsored rewards
* coupon marketplace
* advanced multimodal moderation

---

# 113. Explicitly Avoid in MVP

Do not build:

* DMs
* stories
* livestreaming
* crypto wallets
* tokens
* NFTs
* large video hosting
* paid creator subscriptions
* influencer marketplace
* complex advertising
* native mobile applications
* autonomous AI accounts
* dozens of reactions

Keep the first version focused.

---

# 114. Phase 1 — Foundation

Build:

```text
Auth
Profiles
Post
Comment
Follow
Topics
Basic feed
Basic moderation
```

At this stage, the product is usable even without advanced AI.

---

# 115. Phase 2 — AI-Native Layer

Add:

```text
automatic classification
topic inference
entity detection
personal-life detection
AI-risk signals
spam detection
smart feed
automatic tags
```

---

# 116. Phase 3 — Contribution System

Add:

```text
Helpful
Not Helpful
More Details
Disagree
Curious
Impact
Daily Contribution
Points
```

This phase changes the social behavior of the platform.

---

# 117. Phase 4 — Discovery

Add:

```text
Similar Minds
Different Perspectives
Working on Similar Things
Entity Pages
Semantic Search
```

---

# 118. Phase 5 — Provenance

Add:

```text
content hashes
duplicate recognition
content lineage
repost attribution
external timestamping
```

---

# 119. Phase 6 — Ecosystem

Add:

```text
company profiles
product pages
verified experience
partner rewards
coupon marketplace
beta programs
research challenges
```

---

# 120. Phase 7 — Advanced AI

Eventually:

```text
learned AI detector
behavioral model
advanced moderation
semantic source verification
multimodal analysis
personalized ranking
advanced interest graph
discussion synthesis
```

---

# 121. Example User Journey

## Day 1

User joins.

They select nothing.

They simply see:

```text
What are you interested in?
```

The system learns naturally from interactions.

---

## User makes first post

> "I've been testing local LLMs on low-power hardware and most people underestimate how much quantization changes the experience."

The AI automatically identifies:

```text
AI
LLMs
Edge Computing
Hardware
Opinion / Experience
```

No user input required.

---

## Community responds

```text
Helpful
420

More Details
89

Disagree
62
```

Someone comments:

> "What quantization levels did you test?"

Another:

> "I saw the same thing with Q4_K_M."

Now the post has become a useful discussion.

---

## AI identifies similar minds

The system discovers:

```text
@alice
@bob
@sam
```

who regularly discuss similar topics.

It recommends them.

---

## Daily reward

The user may have made 8 posts.

The AI evaluates all 8.

Only one is selected:

> **Daily Contribution**

The user gets contribution points.

---

# 122. Another Example: Software Review

User writes:

> "I've used Notion for years. It's great for organizing research, but once the workspace gets large I find navigation frustrating."

AI identifies:

```text
Entity: Notion
Type: Review
Experience: Long-term
Topics:
  Productivity
  Software
  Knowledge Management
```

The post automatically appears on:

```text
Notion
```

entity page.

Other users can mark:

```text
Helpful
More Details
Disagree
```

Now the platform is building a contextual knowledge base.

---

# 123. Another Example: News

User writes:

> "Google just announced X. This could make local inference much easier."

AI identifies:

```text
News
Analysis
Google
AI
Local inference
```

The source is automatically detected if attached.

The post can appear in:

```text
AI
Google
Local AI
```

---

# 124. Another Example: Scam Awareness

User writes:

> "Be careful with fake APIs pretending to provide free model access. I found several websites asking for API keys."

The AI identifies:

```text
Awareness
Cybersecurity
AI
Potential safety information
```

It can increase distribution because the information may help others, subject to verification.

The user does not need to choose "awareness."

---

# 125. Another Example: Pure Lifestyle

User writes:

> "Went to the beach today."

The AI recognizes:

```text
Primarily personal-life content
```

The system may:

* reduce feed distribution
* suggest adding useful context
* deprioritize the post

depending on the final community rules.

It should not necessarily delete it automatically.

---

# 126. Another Example: Personal Experience With Value

User writes:

> "I went to the beach during monsoon season and discovered the area has dangerous rip currents. Here's what travelers should know."

Now the system identifies:

```text
Personal experience
Safety information
Travel information
Awareness
```

This is valuable despite being personal.

---

# 127. Cultural Rule

The platform should teach users:

> **Your life can appear in a post. Your life is just not the point of the post.**

This is a much better cultural rule than banning personal language.

---

# 128. The Platform's Social Norm

Traditional:

> "Look at me."

This platform:

> **"Look at this."**

Traditional:

> "Give me attention."

This platform:

> **"Maybe this will help you."**

Traditional:

> "I have 100 posts."

This platform:

> **"I made one useful contribution today."**

---

# 129. AI-Native Social Experience

The AI should silently handle:

```text
"Where does this belong?"
"Is this useful?"
"Is this copied?"
"Is this likely automated?"
"Who might care?"
"Who thinks similarly?"
"Who disagrees?"
"Is there a source?"
"Is this a review?"
"Does this need context?"
"Who should see it?"
"Should this be rewarded?"
```

The user simply writes.

---

# 130. Design Principle: Complexity Belongs to the Platform

The UI should be simple.

The backend can be extremely sophisticated.

This should be a deliberate inversion:

```text
USER
Simple interface
One composer

           ↓

PLATFORM
Complex intelligence
```

Not:

```text
USER
20 settings
10 categories
5 tags
3 forms

           ↓

PLATFORM
Dumb feed
```

---

# 131. Long-Term Competitive Advantage

The potential moat is not simply:

> AI moderator

or:

> blockchain

or:

> rewards.

It is the combination of:

```text
Human-oriented content culture
+
behavior-aware AI moderation
+
useful semantic reactions
+
content provenance
+
interest graph
+
Similar Minds
+
Different Perspectives
+
entity knowledge
+
daily contribution economics
```

Together these produce a different type of network.

---

# 132. Long-Term Product Loop

```text
                USER HAS AN IDEA
                       │
                       ▼
                    WRITES
                       │
                       ▼
              AI UNDERSTANDS IT
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
    TOPICS          ENTITIES         PEOPLE
       │               │                │
       ▼               ▼                ▼
   DISCOVERY        REVIEWS       SIMILAR MINDS
                                        │
                                        ▼
                               DIFFERENT PERSPECTIVES
       │               │                │
       └───────────────┼────────────────┘
                       ▼
                  DISCUSSION
                       │
              ┌────────┼─────────┐
              ▼        ▼         ▼
           Helpful   Detail   Disagree
              │        │         │
              └────────┼─────────┘
                       ▼
                 COMMUNITY VALUE
                       │
                       ▼
                DAILY CONTRIBUTION
                       │
                       ▼
                    REWARD
                       │
                       ▼
                BETTER REPUTATION
                       │
                       ▼
              MORE USEFUL DISCOVERY
```

---

# 133. North-Star Product Question

Every major product decision should answer:

> **Does this help users discover, understand, discuss, or contribute something useful?**

If the answer is no, the feature should be questioned.

---

# 134. Non-Negotiable Product Rules

### Rule 1

No reward for posting volume.

### Rule 2

Only one primary contribution reward per user per day.

### Rule 3

AI detection is behavioral and probabilistic, not a simplistic "AI detector."

### Rule 4

The user should not be required to manually classify their content.

### Rule 5

Personal experiences are allowed when they provide broader value.

### Rule 6

Useful disagreement is valuable.

### Rule 7

The platform should help users discover both similar and different perspectives.

### Rule 8

Content attribution should persist through reposts and derivatives.

### Rule 9

AI should reduce user complexity rather than add more forms.

### Rule 10

The platform should optimize for usefulness, not maximum posting.

---

# 135. Definition of Success

The product is successful when a user opens the platform and thinks:

> "I learned something."

Then:

> "I found someone who thinks like me."

Then:

> "I found someone who disagrees with me but has an interesting argument."

Then:

> "I want to contribute something useful too."

That is the behavioral loop the product should create.

---

# 136. Final Product Definition

## What is it?

An **AI-native social network for human ideas, useful information, discussion, discovery, and contribution.**

## What makes it different?

The platform is deliberately built around:

**Contribution instead of self-promotion.**

**Useful feedback instead of likes.**

**Daily impact instead of posting volume.**

**Similar minds instead of follower farming.**

**Different perspectives instead of echo chambers.**

**Provenance instead of content theft.**

**AI assistance for the platform instead of AI-generated social spam.**

## What does the user do?

Almost nothing beyond:

> **Write what they think.**

## What does the platform do?

Almost everything else.

It understands the content, organizes it, moderates it, finds relevant people, identifies useful information, tracks provenance, detects suspicious automation, recommends discussions, evaluates contribution, and rewards the most useful contribution rather than the highest posting volume.

---

# 137. One-Sentence Product Definition

> **A social network where people share what they think, the AI understands everything else, and the community rewards people for being useful rather than being loud.**

---

# 138. Core Tagline Candidates

Primary:

> **Share what you think. Not your lifestyle.**

Secondary:

> **You post. We understand.**

Alternative:

> **Less self-promotion. More contribution.**

Alternative:

> **What can you contribute today?**

Long-term philosophical statement:

> **Ideas travel. Credit stays.**
