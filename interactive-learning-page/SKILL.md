---
name: interactive-learning-page
description: Generate a self-contained interactive HTML learning page from URLs, PDFs, documents, notes, repositories, transcripts, or other source material. Use when the user asks to study, learn, master, review, prep for an exam/interview, create a study guide, quiz page, flashcard-like HTML tool, Feynman-technique self-test, or interactive learning page in any language, including Chinese requests such as "学习", "掌握", or "复习".
---

# Interactive Learning Page Generator

Turn source material into a teaching *system*, not a formatted article. The HTML/CSS/JS is just the delivery mechanism — the actual work is instructional design: chunking content into digestible modules, forcing active-recall checkpoints after every concept, and making progress visible.

A page that is only "accurate + pretty" but has no checkpoints is not a learning tool — it's a nicely typeset article. Do not ship that.

## Before writing any code

1. **If a frontend/visual-design reference skill is available in the current environment, consult it** for visual design process guidance. If none is available (many environments won't have one bundled), rely on the design-token process in Step 5 below — it's self-contained and doesn't require external reference material.
2. **Ingest the source material** — see Step 0 below for format-specific handling.
3. **Determine the target language** — see Step 0a below. Do this before writing any copy; retrofitting a page from one language to another after the fact means redoing every string.
4. **Decide the learner's anchor context.** Ask (or infer from memory/conversation) what background the learner wants analogies grounded in — e.g. "infra background (Redis/ES/distributed systems), prepping for AI Agent interviews". Generic analogies ("it's like a librarian...") are a fallback, not the target.
   - **When there's no signal and asking isn't an option** — single-turn generation, an approved test run, a request routed in from another system — don't block, and don't retreat to generic analogies either. Pick the *adjacent practitioner domain*: the field someone reading this particular source most likely already works in (systems/engineering source → backend & infra: caching, CI/CD, code review, routing, ablation; finance source → accounting/ops; biology source → lab workflow). Commit to one specific anchor instead of hedging across three.
   - **Say so on the home view**, in one line — e.g. "类比锚定在后端/基础设施背景；若不符，告诉我可整体替换". An unstated assumption reads as a wrong analogy; a stated one reads as a configurable default.
   - Keep analogies in the module data array (one `analogy` field per module) so re-anchoring is a bounded edit, not a rewrite.

## Execution mode and pacing

Default to a staged workflow so large pages do not stall:

1. Ingest the source and produce a compact outline: source sections, proposed modules, prerequisites, interaction inventory, and output language.
2. Ask for confirmation before writing the full HTML unless the user explicitly asks to generate the final artifact in the same turn, is running an approved test, or has already approved a plan.
3. Once generation is authorized, create the output file early with a working shell: layout, navigation, data arrays, rendering functions, persistence helpers, and placeholder module data.
4. Fill content iteratively in bounded passes: modules first, then quick checks, then Feynman lab, final exam, cheatsheet, and polish.
5. After each substantial file write, run a quick validation appropriate to the environment (for example: file exists, HTML contains closing tags, browser loads without script errors, core interactions work).

Never spend a long uninterrupted stretch composing a complete page in memory without creating or updating the target artifact. A partial but runnable page is better than no file; expand it until it satisfies the self-check.

## Step 0a — Language and locale

The page's language is **not** automatically the source material's language, and it's not hardcoded to any one language either — it's whatever the learner needs.

- **Default**: the language the person is writing to you in, in the current conversation. If they've been chatting in Spanish, generate the page in Spanish even if the source PDF is in English — you're already re-explaining everything in your own words (Step "Content fidelity" below), so writing that explanation in a different language than the source is the same operation, just with a translation step folded in.
- **Explicit override wins**: if they ask for a specific output language that differs from the conversation language ("give me an English version of this for my study group"), honor that instead.
- **If genuinely ambiguous** (e.g. a multilingual conversation, or a request routed in from another system with no clear language signal), ask — one short question, don't guess and generate the wrong 45-module page.
- **Everything learner-facing must be in the target language, with no leftover mix**: sidebar labels, buttons, view titles, module prose, quiz questions/options/explanations, Feynman prompts and expert checklists, verdict messages, the cheatsheet — all of it. A page that's 90% Spanish with English button labels because those were copy-pasted from a template reads as broken, not bilingual. Internal code (variable names, CSS classes, code comments) stays in English as normal engineering practice — that's not learner-facing and doesn't need localizing.
- **Keep proper nouns and established technical terms as-is** even when everything around them is translated — protocol/product names (MCP, SWE-bench), API/parameter names, code identifiers, and terms of art that practitioners in that field use in their original form regardless of language. Translating "MCP" into a local-language paraphrase would make the page *less* useful for real-world/interview use, not more.
- **Script and layout**:
  - Pick font-family fallback stacks that actually cover the target script, not just a Latin-plus-CJK default. As a reference: Chinese → `"PingFang SC","Microsoft YaHei","Noto Sans SC"`; Japanese → `"Hiragino Sans","Yu Gothic","Noto Sans JP"`; Korean → `"Apple SD Gothic Neo","Malgun Gothic","Noto Sans KR"`; Arabic/Hebrew → a native UI font plus `"Noto Sans Arabic"`/`"Noto Sans Hebrew"`; Thai → `"Noto Sans Thai"`; Devanagari-based languages (Hindi, Marathi, ...) → `"Noto Sans Devanagari"`; Cyrillic (Russian, Ukrainian, ...) generally renders fine on a standard sans stack but verify bold weights aren't missing. Always end the stack with a generic `sans-serif`/`monospace` fallback.
  - A stylized display/monospace face (e.g. JetBrains Mono) used for eyebrows/labels/code is a Latin-script convention — it either doesn't exist or renders poorly for CJK/Thai/Arabic/Devanagari glyphs. For non-Latin target languages, reserve that face for genuinely Latin tokens (module numbers, English acronyms, code snippets) and set actual prose headers in the body/display face instead.
  - Letter-spacing/tracking values tuned for Latin type (e.g. `letter-spacing:.08em` on eyebrows) usually look wrong on CJK, Thai, or Arabic glyphs — reduce or drop tracking for non-Latin scripts rather than reusing Latin-tuned values unchanged.
  - For Arabic, Hebrew, or other RTL languages: set `dir="rtl"` on `<html>`, mirror the sidebar to the right side, and replace directional arrow *glyphs* in flow diagrams (→) with layout that reads correctly in the flipped direction, or use direction-neutral connectors.
- **Copyright/quoting limits apply identically regardless of language** — translating a source's exact sentences is still reproduction, not paraphrase; the same "explain in your own words, cite facts/numbers freely, avoid long verbatim spans" rule from "Content fidelity" below holds in every language.

## Step 0 — Ingest the source, whatever format it's in

Don't assume "source" means "one article." Handle whatever the user hands over:

| Source type | How to get the content |
|---|---|
| URL / web article | Fetch it with whatever web-fetching capability is available. If it's a multi-page doc site, fetch the index page first to find the real section links. |
| PDF | If a dedicated PDF-reading skill/tool is available in this environment, use it rather than guessing; otherwise extract text directly, checking for a text layer first and falling back to page-by-page image handling only for scanned/image PDFs. |
| Uploaded docx/pptx/xlsx | If a matching skill/tool is available, use it to read content faithfully; otherwise extract text as best the environment allows rather than treating the file as opaque binary. |
| Plain text / markdown notes | Read directly. |
| Code repository | Read the README/docs first (docs-first scan), then a small slice of the key source files — don't try to ingest the whole tree. Good for onboarding-style learning pages. |
| Video / audio | Most environments have no built-in transcription. Ask the user for a transcript or `.srt`/`.vtt` subtitle file rather than guessing at spoken content. Preserve timestamps in the outline if present. |
| Multiple sources at once (e.g. 2–3 related articles) | Ingest all of them before decomposing into modules — cross-reference where they cover the same concept differently, and merge into one coherent module rather than one module per source. |
| No source, just a topic name | Ask what to base it on. Do not invent a "study page" from training-data memory alone — it will drift from what the user actually needs to learn. |

If the source is a **PDF or link the user just uploaded/sent** and content is already visible in context, don't re-fetch it — decompose directly from what's already there.

## Step 0b — Long-source handling (avoid the "back half gets thin" failure)

Long sources (a full book, a multi-hour transcript, a large repo, several long articles bundled together) fail silently if you process them the naive way: you skim the whole thing, then generate modules, and by the time you're writing module 12 you're compressing from memory instead of the source — coverage quietly degrades toward the end.

Instead, use a **two-pass, section-aware approach**:

1. **Pass 1 — Outline first.** Before writing any module content, produce a full structural outline of the *entire* source (every chapter/section/heading, or for transcripts every natural topic shift with rough timestamps). Do this as an explicit intermediate step, not silently in your head — it's what keeps later sections from getting shortchanged.
2. **Pass 2 — Process section by section against that outline.** Write each module by going back to the relevant section of the source, not from your general impression of "what the book was about." For a very long source, it's fine to re-`view`/re-fetch a specific section right before writing its module.
3. **Coverage check before presenting.** Walk the final module list against the Pass-1 outline and confirm every major section is represented by at least one module — including narrow/niche subsections that are easy to drop (the kind of section that's one paragraph in a 40-page chapter). If something didn't make it in, either fold it into the nearest related module's content or add a short module for it — don't silently omit it.

This matters most past roughly 15-20K words of source material or anything with more than ~15 sections; shorter sources can skip straight to Step 1.

## Step 1 — Decompose the content into modules

Do not mirror the source's heading structure blindly. Identify natural **concept units**: each module should require one independent load of working memory to digest. If a source section crams 3 ideas, split it into 3 modules. If 5 subsections all serve one idea, merge them.

Target **10–16 modules** for a normal-length source (1-3 articles / a chapter). Fewer → modules feel bloated; more → learner loses the forest for the trees.

**Map prerequisites while you decompose, not as an afterthought.** For each module, note which *other* modules it assumes the learner already understands (usually 0-2; foundational modules have none). This isn't busywork — most sources have a real dependency structure (you can't understand "Orchestrator-Workers" without "Agent vs Workflow" first) and making it explicit is what lets the page recommend a sane order and diagnose *why* a learner is stuck (they may be failing module 9 because module 4 never landed, not because module 9 itself is unclear). Keep the graph shallow — 1-2 levels of prerequisite chains is normal; if you're building a deep multi-level DAG for a short source, you're over-modeling it.

## Step 2 — Fixed content order inside every module

This order **is** the pedagogy — don't shuffle it:

1. **Eyebrow + title + one-line subtitle** — reader knows the topic in 3 seconds.
2. **Intuition-first explanation** (2–4 paragraphs) — why this concept exists / what problem it solves, *before* naming it or giving a formal definition.
3. **Structural diagram** — for anything involving steps, architecture, comparison, or loops. CSS box-and-arrow diagrams are fine; don't over-invest in SVG art. The diagram must reinforce the prose, not replace it.
4. **Analogy anchor** — bridge to the learner's stated background, phrased specifically ("like X's Y mechanism"), never generic ("kind of like everyday life").
5. **Common-misconception callout** — name what people *think* this means vs. what it actually means; prioritize points that get asked as interview/exam follow-ups.
6. **Comparison table** — mandatory whenever the source has parallel/enumerable items ("5 patterns", "4 principles"). This becomes the learner's main review artifact later.
7. **Key-takeaways box** (3–5 bullets) — the compressed "if you remember nothing else" set. Write fresh sentences, don't just copy-paste lines from the prose above.
8. **Quick-check quiz** (2–3 questions) — see Step 3.

## Step 3 — Interaction mechanisms (all four are required)

### 3a. Per-module quick check
- 2–3 single-choice questions per module, not more (save volume for the final exam).
- Distractors should be *plausible* — confuse adjacent concepts, not obviously wrong — otherwise the quiz measures guessing, not understanding.
- Reveal correct/incorrect + a one-line explanation immediately on click, not after a full submit.
- Module "mastery" status = correct-rate ≥70% on attempted questions. Feed this into a sidebar status icon (not-started / needs-review / mastered) and a global progress bar.

### 3b. Feynman lab
Pick **5–7 concepts** that are easy to *feel* like you understand but hard to *explain* — the ones an interviewer would follow up on with "can you say more about that?". For each:
- Give a **situated prompt** ("A colleague asks you X, explain Y to them...") — not a bare "explain concept X", which elicits shallow answers.
- A `<textarea>` for the learner's own explanation.
- A collapsible **expert checklist** (3–5 points a good explanation should hit) — self-check, not a model answer to copy.
- A 1–5 self-rating control.

### 3c. Cumulative final exam
- 15–20 questions, **shuffled across modules** (do not group by topic — that only tests short-term recall).
- Bias toward **scenario/application questions** ("given a task with property X, which approach fits") over definition-recall.
- Tag each question with its source module (hidden field).
- **Every module needs at least one tagged question, no exceptions.** It's easy to end up with 2-3 modules that silently have zero exam coverage (foundational modules especially — they feel "too basic to test" so they get skipped). This isn't just a completeness nitpick: the weak-area callback below can only flag a module as needing review if some question is tagged to it, so an uncovered module becomes permanently invisible to that mechanism regardless of whether the learner actually understands it. Before finalizing the question set, list all module IDs and confirm each one appears in at least one `tag`.
- On full completion: show score, a tiered verdict message, and **auto-list weak modules with jump links** — a score alone is a dead end, the weak-area callback is what makes the exam useful.

### 3d. Cheatsheet view
One dense page: decision tree (if applicable), the comparison table(s) from Step 2.6 consolidated, and a table of likely follow-up questions with answer angles. No explanatory prose — pure review density, for last-minute pre-interview/exam skimming.

## Step 4 — Information architecture

Five views, sidebar-driven SPA (not one long scroll page):
`Home/map → Module × N → Feynman lab → Final exam → Cheatsheet`

- Home view: one-sentence goal statement, how-to-use-this-page note, a hero diagram encoding the single most important mental model of the whole source, and a card grid of all modules with live status.
- Sidebar: persistent, shows every module with a status icon (color **and** icon, not color alone), plus a global mastery progress bar.
- Prefer view-switching over deep scrolling so the learner always has a sense of "where am I, how much is left."

**Surface the prerequisite map, don't bury it in data.** On the home view, show each module's prerequisites as small text badges on its card ("requires: 04 · 07") rather than a separate graph-visualization widget — a force-directed graph is overkill for 10-16 nodes and adds a dependency you don't need. Two places this data should actually change behavior:
- **Sidebar ordering** stays the source's natural sequence, but a module whose prerequisites aren't yet mastered can get a subtle "prereq pending" indicator, so the learner knows *why* it might feel harder than expected.
- **Exam weak-area callback** (Step 3c): resolve each weak module's prerequisites against those prerequisites' **own quick-check mastery status** (Step 3a) — `prereqs.filter(p => status(p) !== 'mastered')` — not against which modules the exam happened to get wrong. So "start with 04, it feeds into 09" still surfaces when 04 wasn't tested this round.

## Step 5 — Visual design

Don't reuse a fixed palette from a past run — pick a fresh direction each time via this process:
1. Read the subject's *character* first (technical/systems content → engineering-console feel; literary/historical content → something else entirely). Avoid defaulting to cream+serif or black+neon-green just because they "look AI-generated-competent."
2. Define a small **design token set** before writing CSS: 4–6 semantic colors (bg, panel, text, dim-text, accent, warn, good, bad) and 2 type roles (a characterful display/label/code face, a readable body face). Reference them via CSS variables everywhere, never hardcode hex values inline.
3. Status colors (correct/incorrect/needs-review/mastered) must be used consistently across every component — quiz feedback, sidebar icons, progress bars — never redefine the meaning of a color per-component.
4. Use emphasis boxes (callout/analogy/table) sparingly — if every paragraph gets one, none of them read as emphasis anymore.
5. Sidebar should collapse on narrow viewports; don't aim for pixel-perfect mobile, just make sure the page is fully usable there.

## Step 6 — Technical implementation

- Single HTML file, inline CSS + JS, self-contained (works if opened anywhere).
- **Data-driven rendering**: define modules / quiz questions / Feynman topics as JS data arrays/objects, render via shared functions. Never hand-write repeated per-module markup — this is what keeps the visual language consistent and makes later edits cheap. Give each module object a `prereqs: ['m4','m7']`-style field (empty array for foundational modules) so prerequisite badges, mastery-aware sidebar hints, and the exam's weak-area callback can all read from one source of truth instead of hardcoding relationships in markup.
- **Persistence — detect the environment, don't assume one:** the artifact-hosted `window.storage` API (used on Claude.ai) is not available when this file is opened as a plain HTML file, hosted elsewhere, or run in a different platform's sandbox. Use a tiered fallback so the page still works correctly wherever it's opened:
  1. If `typeof window.storage !== 'undefined'`, use it (the environment offers managed persistence — best option where available).
  2. Otherwise, try `localStorage` inside a `try/catch` (covers a plain HTML file opened directly in a browser). Some contexts — a file opened via `file://`, or a sandboxed iframe — throw on `localStorage` access even when the object exists, hence the try/catch rather than just a feature check.
  3. If neither works, fall back to an in-memory JS object for the session, and show a small, honest note in the UI (e.g. near the progress bar) that progress won't be saved after closing the page — don't silently pretend persistence is happening when it isn't.

  **The three tiers do not share a calling convention, and this is the single easiest thing to get wrong in the whole page.** `window.storage.getItem()` returns a **Promise**; `localStorage.getItem()` returns a **string synchronously**. Write the reader as if it were synchronous and the artifact build fails silently — `JSON.parse` receives a Promise, throws, gets swallowed by your restore-time `try/catch`, and the page loads looking perfectly fine with every learner's progress silently gone. It will test clean as a local file and be broken only in the hosted environment, which is exactly the case you're least likely to check.

  So normalize on the async side, not the sync side:
  - **Read once at boot, through a Promise.** Have the low-level reader always return a Promise (`Promise.resolve(...)` around the sync backends), then `await` / `.then()` it once during startup to hydrate a plain in-memory `state` object. Do the first render inside that continuation — never before it, or the page paints an empty state and then jumps.
  - **Everything after boot reads `state` synchronously.** Render functions, mastery math, and progress bars touch the in-memory object only. No render path should ever await storage.
  - **Writes are fire-and-forget.** A single `save()` serializes `state` and hands it to whichever backend is active; nothing waits on the result. If a write throws, degrade to the in-memory tier and surface the note from tier 3.

  Wrap all of this behind a small `load()` / `save()` pair so the rest of the code never touches a backend directly — but note that `load()` is async and `save()` is not, and that asymmetry is deliberate.
- Compute derived state (module mastery %, overall progress %) from raw answer records in one place — don't maintain duplicate counters that can drift out of sync.
- **Output mechanics depend on the platform you're running on** — use whatever this environment's normal way of creating and delivering a file is (a file-creation tool, a code block the user copies, a direct download, etc.). The instruction here is "produce one self-contained `.html` file the learner can open" — not a specific tool name or output path, since those vary by platform.

## Content fidelity

Re-explain everything in your own words and structure. You may cite concrete numbers, named examples, and terminology from the source (facts aren't copyrightable) — but do not mirror the source's sentence structure or phrasing at length. A good learning page reads like someone who deeply digested the material re-explaining it, not a rearranged copy of the original.

## Self-check before presenting

- [ ] Understandable in 10 seconds what the page is and how to use it
- [ ] Every module ends in a checkpoint — no "read-only, flip to next" modules
- [ ] At least a few analogies are specific to the learner's anchor context — stated or assumed — not generic
- [ ] The final exam has real scenario/application questions, not just recall
- [ ] Sidebar makes "what's done / what's left" legible at a glance
- [ ] For long/multi-part sources: every section from the Pass-1 outline is represented by at least one module (no silent thinning toward the end)
- [ ] Modules with real prerequisites show them (badges + exam callback), not just a flat unordered list
- [ ] Every module has at least one tagged exam question, and the prereq callback reads mastery status (not exam-tag co-occurrence)
- [ ] Progress actually survives a reload **in the environment you shipped to** — if the page is hosted where `window.storage` exists, confirm the async read path hydrated state rather than silently swallowing a Promise
- [ ] If the learner's anchor context was assumed rather than given, the page says so
- [ ] All learner-facing text is in one consistent target language — no leftover template strings in a different language than the rest of the page, and the font stack actually covers that language's script

## If the user wants to update instead of regenerate

If they ask to tweak an existing generated page (add a module, change theme, add more quiz questions), edit the existing file in place rather than regenerating it from scratch — preserve the existing data structures and persistence keys so a learner's saved progress keeps working after the update.
