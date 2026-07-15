# I2DL Exam Q&A — TUM IN2346

A closed-book revision deck for **Introduction to Deep Learning (IN2346)** at TU München.
Every problem from 12 past exams + the official course summary, classified by
**chapter → knowledge point**, sorted by **exam frequency**, and answered from the
**official solutions**. Interactive, searchable, mobile-friendly, formulas in KaTeX.

🔗 **Live:** https://c0nsTantin77.github.io/i2dl-exam-qa/

## Contents
**492 questions** across 7 chapters and 43 knowledge points (every point also has an AI practice question):
Machine Learning Basics (69) · Neural Networks (93) · Convolutions (63) · Optimization (126) ·
Popular Architectures (65) · RNNs & Transformers (56) · Appendix: Matrix Calculus (20).

## Features
- **Site-wide search** + per-question **concept tags** to review a theme across chapters.
- **Interactive multiple-choice, short-answer** (matching words light up green, including common singular/plural forms) **& number-checked calculation questions**.
- **Review / Test modes**: read answers immediately or hide them for active recall.
- **🃏 Anki-style flashcards (SM-2)**: study a deck (due / wrong-book / all), rate each card *Again / Hard / Good / Easy*, and let spaced repetition schedule it; MC, short-answer and calculation cards are interactive, and you can jot a **note without leaving the card** (shared with that question's own note).
- **Study tracking**: mark **Reviewed**, keep a **wrong book**, write per-question **notes (Markdown + `$math$`, live preview)**, and work a **spaced-repetition** review page grouped by chapter with progress rings — optionally **synced** across devices via Google sign-in.
- Selected questions carry the original **exam figures**.
- **Frequency-first** ordering (🔥 badges); every question **source-tagged** like `SS22 3.1`.
- KaTeX formulas, collapsible answers, mobile-optimized.

## Recent updates
- **2026-07-02** — 🃏 Flashcard polish: **notes inside a card** (shared with that question's note), **quick-jump pills** (Flashcards / Due today / Wrong book) on the Review page, a clearer **"How review works"** explainer, and a **"Review due"** button that no longer creeps the schedule forward on repeat taps.
- **2026-06-30** — 🃏 **Anki-style flashcards** on the Review page: rate each card *Again / Hard / Good / Easy* and let **SM-2** spaced repetition schedule it.
- **2026-06-30** — 🖼️ Questions can show **exam figures**; reviewed/due questions are **colour-coded** green/red; and there's a guarded **Reset progress** button.
- **2026-06-22** — 🧹 Feedback cleanup: removed **20 duplicate questions** (512 → **492**) and migrated reviewed / wrong-book / notes / spaced-repetition data from removed question IDs to the kept copies.
- **2026-06-22** — 🎯 New study flow: chapter pages now have **Review / Test** modes, short-answer self-checks highlight overlapping answer words in green, and **21 calculation questions** accept typed numeric answers.
- **2026-06-22** — ✍️ Better answer matching: lightweight singular/plural matching now catches pairs like `parameter`/`parameters`, `weight`/`weights`, and `gradient`/`gradients` without adding client-side NLP dependencies.
- **2026-06-09** — 🔍 Search now **highlights your match**: the home-page search and each chapter's filter box light up the words you typed (bold + soft highlight) in the question & answer; formulas stay intact.
- **2026-06-09** — ⚡ Under-the-hood glow-up: the whole site was **rebuilt with Astro + Vue**. Formulas now render **instantly** (no load-time flicker), pages are lighter and faster, and all your old bookmarks & links still work. 🚀
- **2026-06-08** — Home-page polish: collapsible **Recent updates / How to use** panels, more **Popular tags**, the in-page **contents menu** restored on exam & tag pages, and tighter mobile spacing.
- **2026-06-08** — Home page: toggle **By chapter / By exam** (each exam card shows your review progress), a refreshed Apple-flavoured hero, **one-tap Share** (copies a ready-to-send message), and a **Feedback & issues** card.
- **2026-06-08** — Chapter navigation revamp: the contents menu is now a top-bar **"you are here" pill** (shows your current section, opens a floating popover that's **full-width on mobile**); assets are cache-busted so updates appear immediately.
- **2026-06-07** — ✨ **You're never studying alone!** A live counter at the top of every page shows how many fellow students are grinding through I2DL right now — look up, someone's in the trenches with you. 💪
- **2026-06-07** — Chapter-page readability: **numbered topics** (4.2) **& questions** (4.2.1), an **always-on left contents panel** that highlights your current section as you scroll, and tidier multi-part **answer/summary line breaks**.
- **2026-06-07** — Mined SS20/SS21/SS22/SS24: +87 questions (now **512**); **all 12 exams covered, overall 90%** (SS21 100%, SS22 98%, SS20 91%).
- **2026-06-07** — Mined WS22/WS21/WS26/WS23/SS23/Mock: +144 questions; overall exam coverage **41%→70%** (WS22 95%, SS23 93%, WS26 89%).
- **2026-06-06** — Full WS24 (Winter 2024) endterm mined: +41 questions, WS24 coverage 4%→98%, overall 31%→41%.
- **2026-06-06** — Sign in with Google to sync progress (reviewed / wrong book / notes) across devices.
- **2026-06-06** — Study tools: mark Reviewed, wrong book, per-question notes, spaced-repetition review, progress dashboard.
- **2026-06-06** — Browse by exam paper (`SS23 6.1` jumps to a question), concept-tag pages, AI multiple-choice.
- **2026-06-04** — One night, I couldn't sleep, so I decided to build something useful for myself and all TUM students. That was the beginning of an amazing story!😊

## Milestone

## Develop
Built with **[Astro](https://astro.build) + Vue islands**. Content stays in `data/*.json`
(validated by a Zod schema); formulas are rendered to HTML **at build time** with KaTeX, so
pages ship no client-side math JS. Interactive bits (search, tag/exam/review browsing) are Vue
islands; per-question quiz/study controls and the progress dashboard are small ES-module scripts.

```bash
npm install
npm run dev        # local dev server (hot reload)
npm run build      # → dist/  (static site)
npm run preview    # serve the built dist/ locally
npm run check      # astro check (type-check .astro/.ts)
```

> On **Windows PowerShell** the `npm` shim is `npm.ps1`, which can be blocked by the
> execution policy — use **`npm.cmd run …`** instead (e.g. `npm.cmd run preview`).

Content validators still run on the JSON source:

```bash
python tools/check_sources.py    # verify every SSyy x.y citation against the exam text
python tools/check_math.py       # validate KaTeX delimiters / macros
python tools/coverage_report.py  # per-exam coverage → COVERAGE.md
```

### Repository layout
```
data/                 chapter JSON (content source of truth) + manifest + coverage
src/
  content.config.ts   content collection + Zod schema
  lib/                qid · md (markdown+KaTeX) · content/search · exams · paths · config
  lib/client/         store · quiz · study · progress · cloud · presence · contents · …
  components/         Layout · Topbar · SiteFooter · QuestionCard · KnowledgePoint
  components/islands/ GlobalSearch · TagBrowser · ExamBrowser · ReviewPage (Vue)
  pages/              index · chapters/[id] · tags · exams · review · search-index.json
public/               static assets (tum.svg, .nojekyll)
tools/                content mining + validation scripts (Python)
```

### Deploy
Pushing to `main` runs `.github/workflows/deploy.yml` (build + publish to GitHub Pages).
**One-time:** set **Settings → Pages → Source = GitHub Actions** (was "main / root"). `dist/`,
`node_modules/`, and `.astro/` are git-ignored — the repo holds source only.

---
*For personal revision. Answers follow the official I2DL sample solutions; AI-generated items are marked.*
