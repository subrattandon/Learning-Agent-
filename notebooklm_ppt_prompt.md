# 📊 NotebookLM PPT Prompt — Autonomous Learning Agent Project

> **Use kaise karein:** Ye poora content copy karo → NotebookLM me naya notebook banao → apne project files upload karo (README.md, presentation.md, project_explanation.md, src files) → phir neeche wala **MAIN PROMPT** paste karo. NotebookLM jab PPT banaye, uske baad "Add a detailed speaker note for every slide" wala follow-up prompt bhi use karna.

---

## 📋 PROJECT CONTEXT (NotebookLM ko ye do)

NotebookLM me ye files upload karo:
1. `README.md`
2. `presentation.md`
3. `project_explanation.md`
4. `src/deep_research_from_scratch/learning_project.py` (main graph)
5. `src/deep_research_from_scratch/utils.py`
6. `CLAUDE.md`

---

## 🎯 MAIN PROMPT (Ye paste karo)

```
You are helping me prepare a college final-year project presentation (Project Demonstration / Viva).

PROJECT: "Designing an Autonomous Learning Agent with Checkpoint Verification and Feynman Pedagogy"

Based ONLY on the uploaded project documents, create a professional, visually well-structured slide deck (Google Slides style) with EXACTLY 16-18 slides. Use simple English with occasional Hindi/Hinglish explanations where natural, keep every slide concise (max 5-6 bullets, 12-18 words per bullet), and make it suitable for a 10-12 minute presentation to professors.

SLIDE STRUCTURE:

Slide 1 — Title Slide
- Project title, tagline: "An AI tutor that researches, teaches, tests, and repairs understanding using Feynman Technique"
- Team/Group name placeholder, college placeholder, date

Slide 2 — The Problem
- Self-learning has no accountability (no one verifies if you actually understood)
- Information overload: YouTube/Google give 50 links but no personalized path
- No adaptive teaching — everyone gets the same explanation
- Complex jargon scares beginners
- Every new topic = fresh research, no memory of past work

Slide 3 — Our Solution (One-line pitch + overview)
- One-line pitch: An autonomous AI agent that researches a topic, converts the report into checkpoints, teaches, quizzes you, and if you fail, re-teaches using Feynman Technique — until you pass
- Key words: Research → Teach → Test → Verify → Repair → Repeat

Slide 4 — Real-World Impact / Applications
- EdTech platforms (automated personal tutors like Byju's/Coursera-style)
- Corporate training & onboarding
- Exam preparation systems
- Anyone doing self-study without a teacher

Slide 5 — High-Level Architecture (Diagram)
- 3 phases: SCOPE → RESEARCH → LEARN
- Draw as horizontal flow with arrows
- Mention: single LangGraph workflow integrates all three phases

Slide 6 — Complete Workflow (Part 1: Scope + Research)
- clarify_with_user: asks clarifying question if request is vague
- check_data_exists: keyword-overlap cache check on saved reports (skips research if a relevant 250+ word report exists)
- If no data: write_research_brief → prepare_mcp_research → MCP research subgraph (ReAct loop with think_tool) → supervisor subgraph (parallel researchers via asyncio.gather, max 6 iterations) → final report generation → saved to files/ as .md

Slide 7 — Complete Workflow (Part 2: Learning Loop)
- generate_structure: breaks report into 3 checkpoints (name + objective)
- create_content: ~100-word study material + exactly 3 questions per checkpoint
- teach_checkpoint: displays material + questions
- administer_quiz: graph PAUSES via interrupt() — human-in-the-loop
- evaluate_submission: LLM grading, pass mark 70/100
- Conditional router: PASS → next checkpoint / FAIL → Feynman re-teaching / all done → course_completed

Slide 8 — Node-by-Node Deep Dive Table
- Table with columns: Node | Purpose | Key Technology
- Rows: clarify_with_user (Structured output), check_data_exists (Keyword cache), mcp_research_subgraph (MCP + ReAct loop), supervisor_subgraph (Multi-agent parallel), generate_structure (Pydantic schema), administer_quiz (interrupt/resume), evaluate_submission (Rubric 70/100), simplified_teaching (Feynman Technique)

Slide 9 — Checkpoint Object Lifecycle (State flow)
- Visual chain: {name, objective} → +study_material, +quiz_questions → +user_answers → +score, +passed, +feedback → +simplified_material (if failed)
- Explain State = shared memory (TypedDict), every node reads & updates it

Slide 10 — Feature 1: Human-in-the-Loop Quiz (interrupt/resume)
- interrupt() legally pauses the graph
- State persisted by LangGraph checkpointer
- Resume via graph.invoke(Command(resume=answers))
- App restart does not lose progress

Slide 11 — Feature 2: Feynman Technique Automation (USP)
- What is Feynman Technique: explain like to a 10-year-old, no jargon, use analogies
- Input: student's WRONG answers + feedback + original material
- Output: simplified_material replaces original on retry
- Insight: where you cannot simplify = where you don't understand (quiz detects these gaps)

Slide 13 — Feature 3: Smart Memory / Caching
- Reports saved as .md in files/ folder
- check_data_exists uses keyword overlap (4+ letter words, stopwords removed)
- Sufficiency gate: 250+ words AND (overlap ≥2 & ratio ≥50% OR ratio ≥70%)
- Proof: re-asking same topic shows "Found existing report — skipping research"
- This makes the agent self-improving over time

Slide 14 — Technology Stack (Grid/Table)
- LangGraph: state-machine agent workflows (nodes, edges, conditional routing)
- LangChain: LLM orchestration
- Pydantic: structured output — stops hallucination, fixed schema (score:int, passed:bool)
- Tavily API: AI-friendly web search
- MCP (Model Context Protocol): standardized tool access (filesystem server via npx)
- Ollama/LM Studio: local LLM (privacy, free, offline) + Google Gemini fallback for structured output
- LangGraph Studio: visual debugging UI
- Docker Compose: one-command deployment
- Jupyter Notebooks: tutorials that auto-generate src/ code via %%writefile
- uv: fast Python package manager

Slide 15 — LangGraph Concepts Used (for viva)
- State (TypedDict) + Reducers (operator.add, add_messages)
- Nodes (functions), Edges (fixed), Conditional Edges (router)
- Command(goto=..., update=...) — routing + state update together
- interrupt() + Command(resume=) — human-in-the-loop
- Subgraphs (mcp_research, supervisor as child graphs)
- Structured Output with Pydantic

Slide 16 — Current Limitations (Honest section)
- Local LLM marking inconsistency
- Fixed exactly 3 questions, one-shot 70-mark check (ratta possible)
- No long-term student profile / progress memory
- Keyword-based file matching weak (synonyms like "AI" vs "Artificial Intelligence" miss)
- Single topic, English only, limited crash/retry in research
- No automated tests/CI

Slide 17 — Future Scope
- Student progress dashboard with analytics graphs
- Spaced repetition (1-3-7-15 day revision quizzes)
- Adaptive difficulty (90+ score → harder questions)
- Hindi/Hinglish teaching mode
- Question bank with MCQ/coding/audio, Bloom's taxonomy levels
- RAG with vector embeddings instead of keyword matching
- Web app UI (Streamlit/Next.js), multi-user classroom mode
- LangSmith full tracing + certification on completion

Slide 18 — Future Corrections (Technical Improvements)
- Robust LLM-as-judge: multiple evaluators, temperature=0, fixed answer keys for MCQ
- Embedding-based semantic file matching (RAG)
- Real DB (Postgres/SQLite) for student profiles + thread-based resume
- Exponential backoff retries + graceful degradation to cached reports
- Unit tests + Pytest + GitHub Actions CI
- Explicit score_out_of:100 field instead of score<=10 hack
- Hybrid LLM routing: local for simple tasks, cloud for critical evaluation

Slide 19 — Conclusion / Closing
- Recap in one line: Research → Teach → Test → Verify → Feynman Repair — all in ONE autonomous LangGraph workflow
- Human-in-the-loop + structured output + local-first + memory = reliable autonomous learning
- "Thank you — Questions?"

DESIGN REQUIREMENTS:
- Clean academic style, consistent colors (blue/white theme), clear headings
- Use arrows, tables, and simple diagrams wherever possible instead of plain text
- Each slide needs a short bold TITLE (max 8 words)
- Add 3-4 line SPEAKER NOTES for every slide explaining the content in simple Hinglish (Roman Hindi) so I can speak naturally during the presentation
- Do NOT put code snippets on slides — only concepts
- Include the project's key numbers: 3 checkpoints, 3 questions each, 70/100 pass mark, ~100-word material, max 6 supervisor iterations, 250-word cache gate
```

---

## 🔄 FOLLOW-UP PROMPTS (PPT banne ke baad use karne wale)

### Prompt 1 — Speaker Notes (Hinglish)
```
For every slide you just created, add detailed speaker notes in easy Hinglish (Roman Hindi) — 5-7 lines each — as if I am speaking to professors. Include a natural transition line at the end of each slide like "Aage badhte hain..." Also add a 30-second opening script on slide 1 and a closing "thank you + questions" script on the last slide.
```

### Prompt 2 — Q&A Preparation
```
Create a separate document (not slides) with 15 likely viva questions about this project — technical (LangGraph, interrupt, structured output, MCP, Feynman), comparison (vs ChatGPT), and project-specific — each with a 3-4 line easy Hinglish answer.
```

### Prompt 3 — Diagram Slides
```
Redesign slides 5, 7 and 9 as full visual diagram slides: Slide 5 = 3-phase architecture flow, Slide 7 = complete graph workflow with arrows (START → clarify → cache check → research path OR learning path → evaluate → Feynman loop → END), Slide 9 = checkpoint lifecycle chain. Use boxes and arrows, minimal text.
```

### Prompt 4 — 2-Minute Version
```
Create a condensed 6-slide version of this deck for a 2-minute pitch: Problem, Solution, How it works (one flow), Key Tech, Demo highlight (Feynman moment), Future scope.
```

---

## 💡 PRO TIPS (NotebookLM ke liye)

1. **Pehle files upload karo, phir prompt do** — tabhi grounded (sahi) content aayega
2. Agar NotebookLM ka prompt chhota lage toh **"Create a detailed 18-slide deck"** likho pehli line me
3. PPT banne ke baad **"Make slide 7 more visual with a flow diagram"** jaise specific follow-ups do
4. Google Slides export ka option check karna — NotebookLM slides usme ja sakti hain
5. Demo wala screenshot slide chahiye toh alse likhna: *"Add a slide showing a sample conversation: user asks topic → quiz appears → wrong answer → Feynman simplified explanation appears"*
