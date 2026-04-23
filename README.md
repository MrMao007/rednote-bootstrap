<p align="center">
  <img src="https://img.shields.io/badge/platform-xiaohongshu-FF2442?style=for-the-badge&logo=xiaohongshu&logoColor=white" alt="XiaoHongShu" />
  <img src="https://img.shields.io/badge/runtime-QoderWork-6366F1?style=for-the-badge" alt="QoderWork" />
  <img src="https://img.shields.io/badge/browser-agent--browser-00C853?style=for-the-badge" alt="agent-browser" />
  <img src="https://img.shields.io/badge/license-Apache%202.0-blue?style=for-the-badge" alt="License" />
</p>

<p align="center">
  <strong>English</strong> | <a href="./README.zh-CN.md">简体中文</a>
</p>

# rednote-bootstrap

**A self-evolving Agent Skill for XiaoHongShu (REDnote) operations.**

rednote-bootstrap is not another static "operations guide" — it is a living knowledge system. Every time you ask a new operations question, it automatically opens a browser, searches real posts on XiaoHongShu, reads image-heavy content through visual understanding, cross-validates information across multiple sources, and distills everything into a reusable sub-Skill. The next time you ask a similar question, it responds instantly from its knowledge base — no repeated research needed.

```
You: "What's the posting workflow on XiaoHongShu?"
  ↓
rednote-bootstrap: 3 rounds of keyword search → reads 6 top-liked posts → extracts structured knowledge → generates sub-Skill
  ↓
Sub-Skill saved permanently, reused instantly next time
```

---

## Core Philosophy

### "Research Once, Reuse Forever"

Content platform rules and best practices change constantly. The traditional approach — manually searching, reading posts, compiling notes — is time-consuming, quickly outdated, and not reusable. rednote-bootstrap automates and systematizes this entire process:

1. **Search-First**: Instead of relying on pre-trained knowledge, it fetches first-hand information directly from XiaoHongShu's live content
2. **Adaptive Depth**: Rather than mechanically reading a fixed number of posts, it dynamically decides when to stop based on information saturation
3. **Knowledge Distillation**: Fragments of knowledge scattered across multiple posts are cross-validated and distilled into structured, actionable guides
4. **Registry & Reuse**: All generated knowledge is registered as sub-Skills; subsequent queries hit the registry first — if matched, reuse immediately
5. **Incremental Updates**: If a new question partially matches an existing Skill, only the gap is researched and merged incrementally

This makes it a "learning operations assistant" — the more you use it, the more it knows, and the faster it responds.

### Why Not Just Ask an LLM?

XiaoHongShu operations knowledge has three characteristics that make pure LLM answers unreliable:

- **Highly time-sensitive**: Platform rules update frequently (e.g., mandatory AI content labeling introduced in 2026) — training data may be outdated
- **Practitioner-driven**: Many critical details live in creators' hands-on experience, not in official documentation
- **Embedded in images**: The core content of XiaoHongShu posts is often inside images (not plain text in the DOM), requiring visual understanding to extract

rednote-bootstrap uses browser automation + visual understanding to read real, up-to-date creator experiences directly from the platform, ensuring both accuracy and timeliness.

---

## Architecture

```
rednote-bootstrap/
├── SKILL.md                          # Entry point: routing logic + workflow definition
├── registry.json                     # Sub-Skill registry (auto-maintained)
├── LICENSE                           # Apache 2.0
├── xhs-auth-state.json               # XiaoHongShu auth state persistence (auto-generated)
├── reference/
│   ├── platforms.md                  # Platform adapter config (DOM structure, interaction patterns)
│   ├── search-first-skill.md         # Core research engine (5-phase workflow)
│   └── agent-browser/                # Browser automation capability (dependency)
├── templates/
│   └── sub-skill-template.md         # Sub-Skill generation template
└── generated-skills/                 # Generated sub-Skills (knowledge base, continuously growing)
    ├── xiaohongshu-publishing-guide/
    │   └── SKILL.md
    ├── xiaohongshu-daily-account-nurturing/
    │   └── SKILL.md
    └── .../
```

The system consists of three layers:

| Layer | Component | Responsibility |
|-------|-----------|----------------|
| **Routing** | `SKILL.md` + `registry.json` | Understand user intent, match existing sub-Skills or trigger new research |
| **Research Engine** | `search-first-skill.md` | 5-phase automated research (analyze → collect → evaluate → distill → register) |
| **Knowledge Base** | `generated-skills/` | Ever-growing collection of sub-Skills, each an independent executable operations guide |

---

## How It Works

### Five-Phase Research Engine

When a user asks a question that doesn't match any existing sub-Skill, the research engine kicks in:

```
Phase 1                Phase 2                Phase 3
Topic Analysis         Content Collection     Adaptive Depth Control
───────────────        ───────────────        ───────────────
Decompose into   ──→   Multi-round       ──→   Information saturation
3~5 sub-dimensions     search                  assessment
Generate search        Open & read each        Dimension coverage check
term strategies        post one by one         Contradiction detection
(direct/long-tail/     Screenshot + OCR        Dynamically decide
 reverse)              extraction              to continue or stop
        │                                            │
        │         Phase 5              Phase 4       │
        │         Registration         Knowledge     │
        │                              Distillation  │
        │         ───────────────      ───────────────
        └────────  Update registry ←── Deduplicate + cross-validate
                   Route future          Resolve conflicts
                   queries here          Annotate timeliness
                                        Generate sub-Skill
```

### Adaptive Depth Control

This is the core innovation of the research engine. Instead of collecting a fixed number of posts, it uses four metrics to dynamically assess "is this enough?":

| Metric | Meaning | Stop Threshold |
|--------|---------|----------------|
| Information Novelty Rate | New unique insights this round / Total insights extracted | < 20% → saturated |
| Dimension Coverage | Dimensions with substantial content / Total dimensions | > 80% → sufficient |
| Contradiction Detection | Do different posts present conflicting viewpoints? | Conflicts found → keep researching |
| Authority | Does the collection include official accounts or verified creators? | Lacking → need more sources |

Safety boundaries are also enforced: max 5 search rounds, max 30 posts, max 15 minutes per research session.

### Sub-Skill Routing

When a user asks a question again, the registry is checked first:

```
User Question
  │
  ├─ Exact topic match ──→ Load and execute sub-Skill directly
  │
  ├─ Keywords overlap > 70% ──→ Load sub-Skill, check if incremental update needed
  │
  └─ No match ──→ Launch five-phase research engine, generate new sub-Skill
```

Incremental update strategy for existing sub-Skills:

- Request falls within existing coverage → execute directly, no re-research
- New sub-dimensions identified → research only the gap, merge incrementally
- Sub-Skill older than 30 days → prompt user whether to refresh (platform rules may have changed)

---

## Capabilities

### Supported Operations Scenarios

Through the research engine, rednote-bootstrap can generate sub-Skills for any operations topic on demand. Here are examples already generated:

**📝 Post Publishing Workflow** — A complete 7-step guide from account setup to publishing, covering Creator Center entry tips, cover design, title formulas, hashtag strategies, and the latest 2026 rules including mandatory AI content labeling, CES scoring weights, traffic diversion red lines, and tiered penalty standards. Includes a golden posting time table for every niche and a checklist of 20 behaviors that trigger traffic throttling.

**🌱 Daily Account Nurturing** — Pentagon weight system breakdown, 7-day new account nurturing plan (pure browsing → trial posting → stable operations), daily interaction behavior guidelines, 8 account-damaging taboos, and dormant account revival workflow.

**📋 More Topics (On Demand)** — Prohibited word detection, niche competitor analysis, viral title writing, comment section engagement tactics, data analysis methodology... just ask, and it will research.

### Technical Capabilities

- **Browser Automation**: Controls Chrome via agent-browser with login state management, search, pagination, and screenshots
- **Visual Understanding**: Extracts text and structure from image-heavy posts through screenshots + multimodal comprehension
- **Platform Adaptation**: Built-in XiaoHongShu DOM structure mapping (search box, result list, post detail, image pagination) — works out of the box
- **Session Persistence**: Saves authentication state after first QR code login, automatically reuses it — no repeated logins

---

## Quick Start

### Prerequisites

- [QoderWork](https://qoder.com) desktop app
- [agent-browser](https://github.com/anthropics/agent-browser): `npm i -g agent-browser && agent-browser install`
- Chrome / Chromium browser

### Installation

Clone this repository into QoderWork's Skills directory:

```bash
git clone https://github.com/anthropics/rednote-bootstrap.git \
  ~/.qoderwork/skills/rednote-bootstrap
```

Or install the `.skill` file directly within QoderWork.

### Usage

Simply ask questions in QoderWork — the Skill loads automatically:

```
You: What's the XiaoHongShu posting workflow?
You: How do I nurture a new account?
You: What are the prohibited words on XiaoHongShu?
You: How to write viral titles?
```

First use requires scanning a QR code with the XiaoHongShu app or WeChat (one-time only — auth state is saved automatically).

---

## Sub-Skill Structure

Every auto-generated sub-Skill follows a unified template, ensuring reliability and traceability:

```markdown
---
name: {skill_id}
description: {description}
topic: {topic}
createdAt: {creation_date}
researchDepth: {depth} (analyzed N posts)
---

# {Topic}

> Auto-generated by content-researcher on YYYY-MM-DD
> Sources: N posts from XiaoHongShu, cross-validated
> Confidence: High / Medium / Low
> Last updated: YYYY-MM-DD

## Domain Overview          ← 2-3 paragraphs for global context
## Core Knowledge Base      ← Organized by dimension, each with confidence level
## Executable Workflows     ← Not just "what" but "how to do it"
## Tools & Resources        ← Useful tools discovered during research
## Common Pitfalls          ← Lessons from mistakes and misconceptions
## Timeliness Notes         ← Which info may expire and when
## Sources                  ← Title, author, likes, date of every post — fully traceable
## Knowledge Gaps           ← Honestly flags what the research couldn't cover
```

Key design decisions:

- **Confidence annotation**: Insights mentioned by 3+ posts are marked "high confidence"; single-source insights marked "low"
- **Conflict resolution**: Contradictory viewpoints are presented side by side, never forcibly resolved
- **Timeliness annotation**: Notes which platform rule version the info is based on, warns about potential expiry
- **Knowledge gaps**: Honestly acknowledges what the research didn't cover, rather than fabricating answers

---

## Distillation Rules

The distillation process from raw posts to sub-Skills follows five rules:

1. **Deduplication**: Merge repeated insights across different posts
2. **Cross-Validation**: Multi-source confirmed insights get high-confidence labels
3. **Conflict Resolution**: Contradictory viewpoints presented in parallel, no forced arbitration
4. **Timeliness Annotation**: Flag information shelf-life, warn about potential expiry
5. **Actionability**: Transform vague advice into concrete, step-by-step operations

This ensures the output is not "information dumping" but verified, directly executable knowledge.

---

## Design Decisions

**Why use a browser instead of an API?** XiaoHongShu has no public content search API and enforces strict anti-crawling measures. Real browser automation via agent-browser, browsing as a normal user, is the most stable and reliable approach.

**Why take screenshots instead of reading DOM text?** Because the core content of XiaoHongShu posts is heavily embedded in images, not as plain text in the DOM. Screenshots + visual understanding (OCR / multimodal models) are required to extract complete information.

**Why generate Skills instead of answering directly?** Generating reusable sub-Skills has three advantages: (1) zero-latency response for similar future questions; (2) knowledge can be incrementally updated instead of starting over; (3) users can review, edit, and share structured knowledge.

**Why adaptive depth instead of fixed counts?** Different topics have vastly different information densities — a prohibited word list may need 15 posts to cover comprehensively, while a specific workflow may only need 3. Adaptive depth avoids both "under-researching" and "over-collecting."

---

## Platform Adaptation

Currently provides primary support for XiaoHongShu (REDnote). `reference/platforms.md` defines the complete platform adapter configuration:

- DOM selectors and interaction patterns for the search flow
- Parsing rules for search result lists
- Content extraction patterns for post detail pages
- Image post pagination mechanism
- Content quality signals (like thresholds, comment counts, recency, verification badges)

The architecture is designed to be extensible — new content platforms can be supported by adding new platform configurations.

---

## Contributing

Contributions are welcome:

- **New platform adapters**: Write `platforms.md` adapter configs for other content platforms
- **Template improvements**: Improve the knowledge distillation template structure
- **Engine enhancements**: Optimize the saturation assessment model, search term generation strategies
- **Sub-Skill sharing**: Share your high-quality generated sub-Skills with the community

---

## License

[Apache License 2.0](./LICENSE)
