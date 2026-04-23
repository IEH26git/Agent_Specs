# AGENT PRODUCT SPECIFICATION: Nate's Newsletter Digest Agent

**Version:** 1.0
**Date:** 2026-04-17
**Status:** FINAL

---

## Phase 1–3 Q&A Summary

### Phase 1: Domain and Workflow Selection

**CC:** Walk me through how you currently consume Nate's Newsletter — where does it arrive, how do you read it, and what do you actually do with it today?
**Ian:** Reads it in email, on Substack, or by watching a YouTube video version. When reading, he often captures key insights in a .md file or downloads a prompt kit attached to the article. Sometimes uses NotebookLM to pull full transcripts of the video. Has also captured screenshots of specific video frames.

**CC:** What counts as a "key takeaway"?
**Ian:** Most of Nate's newsletter pieces have the following parts in the introduction: "Here's what's inside," "This piece is about...," and a "LINK:" where you can download something related. Each of those parts of the intro point to something that is a possible takeaway. Ian will decide what matters to him.

**CC:** What is your "daily briefing"?
**Ian:** The Daily Note surfaced from context/memory when running `/start`. One of the things in that flow should be a reminder to read a .md that has been captured from reading Nate's latest post.

**CC:** What goes wrong if the agent does this poorly?
**Ian:** Could miss something important, or distort the message Nate was trying to communicate. Looking for something more than a subject line and title — like an abstract or digest. Ian might still go read the whole thing online, but wants to make sure he doesn't miss any key items by having that information in his path during the daily workflow.

---

### Phase 2: Decomposition Workshop

CC identified five steps in the workflow and classified each:

**Step 1 — Detect a new post**
Retrieval task — check for a new email from Nate's newsletter that hasn't been processed yet.
*Q A: Email trigger or Substack RSS?*
**Ian:** Email trigger. Apple Mail MCP is already wired up.

**Step 2 — Fetch the full post content**
Retrieval task — get post content from the email body (not a web fetch; full post is in the email for paid subscribers).
*Q B: Is the full post in the email or behind a Substack paywall?*
**Ian:** Full post is within the email body. Ian is a paid subscriber — no need to go to the website.

**Step 3 — Extract the structured intro**
Classification/reasoning task — identify the named sections ("Here's what's inside," "This piece is about...," "LINK:") and extract them faithfully. These sections are guidance for finding what's in the post; if exact headings are missing, look for structural equivalents.
*(No clarifying question needed — covered in Phase 1 answers.)*

**Step 4 — Generate the digest**
Reasoning task — for each item in the structured intro, locate the corresponding body content and write a substantive summary. The goal is not to restate the marketing teaser ("what nobody's talking about") but to capture what Nate actually argues or finds, drawn from the body. Each bullet must reference a specific claim or concept.
*Q C: How long should the digest be?*
**Ian:** A few bullets explaining what's behind the items listed under "Here's what's inside." The "Here's what's inside" language is marketing copy — engaging, but not informative. The substance is in the body. Each bullet should reference a specific claim or concept from the body, not restate the teaser.

**Step 5 — Write to file and surface in /start**
Coordination task — save the digest as a new dated section in a single persistent file and have `/start` reference it.
*Q D: Where should the file live, and how should /start surface it?*
**Ian:** One persistent file: `Content/NBJ/NBJ-Digest-Log.md` (create the folder). Each run prepends a new dated section above the previous one, so the file reads newest-first. At the end of a week, section headings would read: Friday — [date], Thursday — [date], etc. `/start` should show a reminder pointing to the file, not inline the digest content. The .md file itself contains the Substack article URL and the abstract.

---

### Phase 3: Trust and Failure Analysis

**Step 1 — Email detection**
*Q A: What does Nate's sender address or subject line look like? And if no new post exists, should the agent stay silent or note it?*
**Ian:** Sender: "Nate from Nate's Substack" \<natesnewsletter@substack.com\>. Should note when no new post is found rather than staying silent.

**Step 2 — Email body extraction**
CC noted: Parsing the email body from HTML is the riskiest retrieval step, as formatting can degrade the quality of input to all downstream steps.
*Q B: If the email body extracts poorly, should the agent fail loudly or make a best-effort attempt?*
**Ian:** Best-effort attempt and flag uncertainty.

**Step 3 — Intro section extraction**
CC noted: Some posts may not follow the standard structure — different formats, announcement posts, prompt kit drops, etc.
*Q C: What if a post doesn't follow the standard structure?*
**Ian:** Try to extract anyway. Those sections are guidance for finding what's in the post. If exact headings aren't found, look for things like them. The goal is to pull out the gist of what is in the post.

**Step 4 — Digest generation**
CC noted: The failure mode here is subtle — producing bullets that sound informative but just restate the teaser framing rather than the actual insight.
*Q D: Is there a quality bar to enforce — e.g., each bullet must reference a specific claim from the body?*
**Ian:** Absolutely. Do not just restate the "Here's what's inside" line. Each digest bullet should reference a specific claim or concept from the body.

**Step 5 — File write**
CC noted: Because the file is now cumulative (not one file per run), the duplicate check must parse the existing file for a section matching this post's date rather than checking for a filename.
*Q E: What happens if the agent runs twice against the same post?*
**Ian:** Second run should note that it has already run and skip making any output. No duplicate section written.

---

## 1. Problem Definition

**Current workflow (human process):**
Ian manually monitors his email for Nate's weekly newsletter. When a post arrives, he reads it in full — sometimes in email, sometimes on Substack, sometimes via a YouTube video version with NotebookLM transcript extraction. Key insights are captured ad hoc into .md files or by downloading attached prompt kits. There is no consistent capture format, no guaranteed review during the workday, and no integration with the daily workflow system. Estimated time: 20–45 min per post when read in full; 0 min when the post gets skipped due to time pressure.

**Target state:**
An autonomous agent detects each new post from natesnewsletter@substack.com, extracts the structured intro and corresponding body content, generates a substantive digest, and prepends it as a new dated section in `Content/NBJ/NBJ-Digest-Log.md`. `/start` surfaces a reminder pointing to that file. Ian still decides whether to read the full post — the agent ensures he never misses the gist.

**Success metrics:**
1. Every new Nate post has a corresponding digest section in NBJ-Digest-Log.md on the day it arrives in email.
2. Each digest bullet references a specific claim, finding, framework, or concept from the body — zero bullets that merely restate a "Here's what's inside" teaser.
3. `/start` surfaces a reminder when a new digest has been written.
4. Zero missed posts — no email from natesnewsletter@substack.com goes unprocessed for more than 24 hours.
5. Duplicate runs produce no duplicate output — second run logs and exits cleanly.

---

## 2. System Architecture

```
[Email Inbox]
      │
      ▼
┌─────────────────┐
│  Step 1         │  Retrieval
│  Email Detector │──── Search Apple Mail for unprocessed email
│                 │     from natesnewsletter@substack.com;
│                 │     check NBJ-Digest-Log.md for duplicate
└────────┬────────┘
         │ email ID(s)
         ▼
┌─────────────────┐
│  Step 2         │  Retrieval
│  Content        │──── Extract plain text body + Substack URL
│  Extractor      │     from email HTML
└────────┬────────┘
         │ body text + URL
         ▼
┌─────────────────┐
│  Step 3         │  Classification / Reasoning
│  Structure      │──── Locate "Here's what's inside,"
│  Detector       │     "This piece is about...," "LINK:"
│                 │     or structural equivalents
└────────┬────────┘
         │ labeled sections
         ▼
┌─────────────────┐
│  Step 4         │  Reasoning
│  Digest         │──── For each item, find body content,
│  Generator      │     write substantive 1–2 sentence bullet
└────────┬────────┘
         │ digest bullets
         ▼
┌─────────────────────────────┐
│  Step 5                     │  Coordination
│  File Writer                │──── Prepend new dated section to
│  (NBJ-Digest-Log.md)        │     NBJ-Digest-Log.md (create on first run)
└─────────────────────────────┘
         │ file updated
         ▼
┌─────────────────────────────┐
│  /start reminder            │  Surface
│  (existing skill)           │──── Note new digest in daily briefing
└─────────────────────────────┘

        ★ Human checkpoint: Ian reviews NBJ-Digest-Log.md before acting.
          No automated actions taken beyond writing the file.
```

**Model tier:** Steps 1–2 are tool calls (no LLM needed). Steps 3–4 require a capable model (Sonnet-class). Step 5 is a file read/write.

---

## 3. Specification for Each Step

### Step 1 — Email Detector

| Field | Value |
|:------|:------|
| Task | Search Apple Mail for emails from natesnewsletter@substack.com not yet processed by this agent |
| Input | Contents of NBJ-Digest-Log.md (to check for existing sections by date); Apple Mail inbox |
| Output | List of email IDs to process; empty list if none |
| Hard constraints | Match sender exactly: natesnewsletter@substack.com |
| Soft guidelines | Process oldest unprocessed email first if multiple exist |
| No new post | Log: "No new post from Nate's Newsletter found." Exit cleanly. |
| Duplicate check | Parse NBJ-Digest-Log.md for a section matching this post's received date. If found, log: "Already processed [date]. No output written." and exit. |

### Step 2 — Content Extractor

| Field | Value |
|:------|:------|
| Task | Parse email body HTML into clean plain text; extract the Substack post URL |
| Input | Email ID from Step 1 |
| Output | Plain text body string; Substack URL string |
| Hard constraints | Do not truncate body content |
| Soft guidelines | Strip navigation links, footer boilerplate, unsubscribe text |
| Edge case: URL not found | Log warning: "Substack URL not found in email body." Proceed with body only; digest section will note URL as unavailable. |
| Edge case: Garbled HTML | Flag: "Email body extraction may be incomplete — review digest for gaps." Proceed with best effort. |

### Step 3 — Structure Detector

| Field | Value |
|:------|:------|
| Task | Locate structural intro sections in the body text |
| Primary targets | "Here's what's inside," "This piece is about...," "LINK:" |
| Fallback | If exact headings absent, locate any enumerated list or labeled section that describes post contents |
| Input | Plain text body from Step 2 |
| Output | Dict of labeled sections with their content |
| Hard constraints | Do not fabricate structure; only extract what is present |
| Edge case: Non-standard format | Extract best-available structural signal; note in digest: "Standard structure not found — extracted from [description of what was found]." |

### Step 4 — Digest Generator

| Field | Value |
|:------|:------|
| Task | For each item in the extracted structure, find the corresponding passage in the body and write a 1–2 sentence substantive bullet |
| Input | Labeled sections from Step 3 + full body text from Step 2 |
| Output | 3–7 digest bullets |
| Hard constraints | Each bullet must state a specific claim, finding, framework, or concept from the body. Restating the "Here's what's inside" teaser line is not acceptable. |
| Soft guidelines | Use plain language. Attribute the insight to Nate where appropriate ("Nate argues...," "The framework he describes..."). |
| Edge case: Body doesn't elaborate | Note inline: "[Not elaborated in this post]" |
| Quality check | Before finalizing, verify each bullet would be meaningless without the body — if it stands on its own as a vague teaser, rewrite it. |

### Step 5 — File Writer

| Field | Value |
|:------|:------|
| Task | Prepend a new dated section to NBJ-Digest-Log.md; create the file on first run |
| File | Content/NBJ/NBJ-Digest-Log.md (single persistent file) |
| Section heading format | `## [Day of week] — YYYY-MM-DD` (e.g., `## Friday — 2026-04-18`) |
| Insert position | Above all existing digest sections, below the file header block |
| Input | Digest bullets, Substack URL, post date, any flags from earlier steps |
| Hard constraints | Never overwrite or modify existing sections. If a section for this date already exists, log and exit. |
| Output format | See template below |

**NBJ-Digest-Log.md structure (cumulative, newest first):**
```markdown
# Nate's Newsletter — Digest Log

---

## [Day] — YYYY-MM-DD

**Post:** [Post Title if extractable, otherwise "Nate's Newsletter [date]"]
**Source:** [Substack URL] *(or: URL not found in email)*
[Flag line if body extraction was uncertain]

### Digest

- [Bullet 1: specific claim or concept from body]
- [Bullet 2: specific claim or concept from body]
- [Bullet 3: specific claim or concept from body]

---

## [Day] — YYYY-MM-DD  ← previous entry

...
```

---

## 4. Evaluation Framework

### Per-step test cases

**Step 1 — Email Detector**
| Case | Input | Expected Output |
|:-----|:------|:----------------|
| Common | New email from natesnewsletter@substack.com, no prior section in log | Returns email ID |
| Edge | No new email since last run | Logs "No new post found," exits |
| Adversarial | Agent runs twice on same day | Second run finds existing section in log, logs "Already processed," exits without writing |

**Step 2 — Content Extractor**
| Case | Input | Expected Output |
|:-----|:------|:----------------|
| Common | Standard HTML email body | Clean plain text + URL |
| Edge | Email with heavy image/table formatting | Best-effort plain text + uncertainty flag |
| Adversarial | Email body contains only "View in browser" link, no inline content | Flags failure clearly; does not write an empty section |

**Step 3 — Structure Detector**
| Case | Input | Expected Output |
|:-----|:------|:----------------|
| Common | Body contains "Here's what's inside" with 4 bullets | Returns 4 labeled items |
| Edge | Non-standard post (e.g., announcement, prompt kit drop) | Extracts best-available structure with note |
| Adversarial | Body is one long essay with no headings | Returns single-section fallback; digest is shorter |

**Step 4 — Digest Generator**
| Case | Input | Expected Output |
|:-----|:------|:----------------|
| Common | 4 "Here's what's inside" items, all elaborated in body | 4 substantive bullets, each grounded in body content |
| Edge | One item not elaborated in body | 3 substantive bullets + 1 "[Not elaborated]" note |
| Adversarial | Body elaboration is itself vague/marketing-heavy | Bullet captures the most concrete claim available; does not amplify vagueness |

**Step 5 — File Writer**
| Case | Input | Expected Output |
|:-----|:------|:----------------|
| Common | First run, file does not exist | Creates NBJ-Digest-Log.md with header + first section |
| Edge | Second post of the week | New section prepended above existing section; existing section unchanged |
| Adversarial | Duplicate run same day | Detects existing section for this date, logs and exits; file unchanged |

**System-level evaluation criteria:**
- Digest can be read in under 2 minutes and gives Ian enough context to decide whether to read the full post.
- No bullet is a restatement of a teaser line.
- New section appears in NBJ-Digest-Log.md on the day the email arrives.
- File structure remains valid after multiple runs — sections are in reverse-chronological order.

**Monitoring metrics:**
- Count of sections in NBJ-Digest-Log.md per month vs. known Nate post frequency (weekly = ~4/month)
- Ian's subjective quality rating after first 4 runs

**Quality drift detection:**
After 8 runs, Ian spot-checks 2 digest sections against the original post to confirm bullets are grounded in body content and not drifting toward summary-of-teasers.

---

## 5. Trust Boundary Map

| Sub-task | Cost of Error | Reversibility | Frequency | Verification Method | Oversight Level |
|:---------|:-------------|:-------------|:---------|:-------------------|:---------------|
| Email detection | Low — missed post | Reversible (re-run) | Weekly | Ian notices missing section in log | Fully automated |
| Content extraction | Medium — garbled input corrupts digest | Reversible (re-run with flag) | Weekly | Agent flags uncertainty | Automated with flagging |
| Structure detection | Low — fallback produces shorter digest | Reversible | Weekly | Agent notes non-standard structure | Fully automated |
| Digest generation | Medium — distorted insight | Reversible (regenerate section) | Weekly | Ian reviews digest before acting | Automated; Ian reviews output |
| File write | Low — duplicate or missing section | Reversible | Weekly | Section presence check | Fully automated |

---

## 6. Failure Mode Analysis

| Failure Pattern | Applies? | Most Likely Location | Detection | Mitigation |
|:----------------|:---------|:--------------------|:----------|:-----------|
| Context degradation | Low | Step 4 (long post exceeds context) | Digest bullets thin out or cut off | Chunk body into sections before passing to Step 4 |
| Specification drift | Medium | Step 4 | Bullets drift back toward teaser language over time | Spot-check every 8 runs; quality prompt is explicit |
| Sycophantic confirmation | Low | Step 4 | Bullets affirm Nate's framing without adding substance | Prompt explicitly requires specific claims, not endorsement |
| Tool selection errors | Low | Steps 1–2 | Wrong email matched; wrong MCP tool called | Hard sender constraint; explicit tool selection in prompt |
| Cascade failure | Medium | Step 2 → 4 | Poor extraction → meaningless digest | Flag propagates; Ian sees warning before reading |
| Silent failure | High risk | Step 1 (no new post, no log) | Ian doesn't know agent ran | Always log outcome — "found" or "not found" — never silent |

---

## 7. Cost Model

All estimates assume Claude Sonnet-class model. Steps 1–2 and 5 are tool calls with negligible token cost.

| Sub-task | Estimated Tokens (input + output) | Model Tier | Notes |
|:---------|:----------------------------------|:-----------|:------|
| Step 3 — Structure detection | ~2,000 in / 300 out | Sonnet | Full email body + extraction prompt |
| Step 4 — Digest generation | ~3,000 in / 400 out | Sonnet | Body + structure + generation prompt |
| **Total per run** | **~5,700 tokens** | | |

**Cost per execution (approximate, 2026 pricing):**
- Input: 5,000 tokens × $3/M = ~$0.015
- Output: 700 tokens × $15/M = ~$0.011
- **Per run: ~$0.026**

**At volume:**
- 4 runs/month (weekly post): ~$0.10/month
- 100× daily volume: not applicable (single newsletter, single publisher)
- 10,000× daily volume: not applicable

**Break-even vs. human cost:**
Current human cost: 20–45 min/post to read and capture manually. At any reasonable hourly rate, $0.026/run is not a meaningful cost consideration.

---

## 8. What This Spec Demonstrates

This specification was built using the NBJ Spec Starter Kit and evidences the following market-premium AI skills:

| Skill | Evidence in This Spec |
|:------|:----------------------|
| **Specification precision** | Each step has explicit input/output format, hard constraints, edge cases, and success criteria precise enough to build from |
| **Evaluation design** | Section 4 includes per-step test cases (common, edge, adversarial) and a system-level quality drift detection method |
| **Decomposition for delegation** | Workflow broken into 5 discrete steps, each with a defined model tier, tool requirement, and handoff point |
| **Trust boundary design** | Section 5 maps every sub-task to cost of error, reversibility, and oversight level — not all steps need human review |
| **Cost economics** | Section 7 estimates token usage per step, cost per run, and break-even against current human time investment |

Skills not heavily exercised here (by design — this is a single-author, low-stakes agent):
- **Multi-agent coordination** — single linear pipeline, no parallel agents needed
- **Regulatory/compliance design** — no legal or safety constraints on a personal newsletter digest

---

# Agent is built. Here's what was created:                                             
                                                                                       
- .claude/commands/nbj-digest.md — the /nbj-digest skill (5-step pipeline: find emails → get body → detect structure → generate digest → write log)                  
- Content/NBJ/NBJ-Digest-Log.md — will be created on first run                       
- /start updated — loads NBJ-Digest-Log.md and shows a reminder in Pointers if a digest exists within the last 7 days                                                 
                                                                                       
## Testing

Run /nbj-digest to test it against your current inbox. Make sure Mail is open before running.                  