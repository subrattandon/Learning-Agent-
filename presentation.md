# 🎯 PROJECT: Autonomous Learning Agent
### (with Checkpoint Verification + Feynman Pedagogy)

**Repo ka naam:** *Designing an Autonomous Learning Agent with Checkpoint Verification and Feynman Pedagogy — Group 2*

---

## 1️⃣ Ek-line Pitch (Presentation ki shuruaat isi se karna)

> *"Bhai ye ek AI agent hai jo pehle khud internet pe research karke ek detailed report banata hai, phir us report ko padhke aapko padhata hai — checkpoint by checkpoint, quiz leta hai, fail ho gaye toh Feynman technique se bilkul simple bhasha me dobara samjhata hai, aur tab tak nahi chhodta jab tak tum pass nahi ho jaate."*

**Problem ye thi:** YouTube/Google pe content bahut hai, but *personalized* learning nahi hota. Koi guarantee nahi ki tum samajh gaye ya nahi.

**Solution:** Ek agent jo **Research → Teach → Test → Remediate (Feynman) → Repeat** ka loop chalata hai.

---

## 2️⃣ Tech Stack

| Layer | Technology |
|---|---|
| Agent Framework | **LangGraph** (state-machine based agent workflows) |
| LLM Orchestration | **LangChain** |
| LLM | **Local LLM** (Ollama / LM Studio) + **Google Gemini fallback** |
| Web Search | **Tavily API** |
| Tool Protocol | **MCP (Model Context Protocol)** — filesystem server via `npx` |
| Structure/Validation | **Pydantic** structured output |
| UI/Debug | **LangGraph Studio** |
| Deploy | **Docker Compose** / `langgraph dev` server |

Special baat: **local-first** design — API key optional hai, structured output na aaye toh Gemini pe fallback ho jaata hai (`utils.py:get_structured_output_model`).

---

## 3️⃣ Project ka Structure

```
Learning-Agent/
├── notebooks/          ← SOURCE OF TRUTH (yahi edit hota hai)
│   ├── 1_scoping.ipynb            → user se clarify + brief banana
│   ├── 2_research_agent.ipynb     → Tavily se research agent
│   ├── 3_research_agent_mcp.ipynb → MCP tools integrate kiye
│   ├── 4_research_supervisor.ipynb→ multi-agent supervisor
│   ├── 5_full_agent.ipynb         → end-to-end system
│   └── 6_checkpoint_agent.ipynb   → ★ Learning + Quiz + Feynman (hamara core)
│
├── src/deep_research_from_scratch/  ← GENERATED (notebook ka %%writefile output)
│   ├── learning_project.py   ← ★ MAIN final graph (sab kuch integrated)
│   ├── learning_agent.py     ← standalone learning agent
│   ├── research_agent_scope.py, multi_agent_supervisor.py, prompts.py, utils.py...
│   └── files/                ← research reports save hoti hain (.md) = memory/cache
│
├── langgraph.json   ← kaunsa graph deploy hoga
├── Dockerfile / docker-compose.yml
└── .env              ← API keys
```

**Important point:** notebooks me `%%writefile` magic command hai — notebook chalao toh `src/` wala Python code apne aap generate ho jaata hai. Isliye `src/` files haath se edit nahi karte. Ye "interactive tutorial → production code" ka pattern hai.

---

## 4️⃣ THE BIG PICTURE — 3 Phases

```
┌─────────────────────────────────────────────────────────┐
│  PHASE 1: SCOPE     →  user se baat karke topic set karo │
│  PHASE 2: RESEARCH  →  internet pe deep research + report│
│  PHASE 3: LEARN     →  checkpoints, quiz, Feynman repair │
└─────────────────────────────────────────────────────────┘
```

---

## 5️⃣ DEEP DIVE — Final Graph ka Flow (`learning_project.py`)

Ye main graph hai (`langgraph.json` me `"learning_project"`). Flow:

```
START
  │
  ▼
clarify_with_user ──(agar info kam hai)──► END  [user se sawaal poochta hai]
  │ (info sufficient)
  ▼
check_data_exists ──────────────────────┐
  │                                      │
  ├─ data mila (relevant report) ───────┤
  │      ▼                              │
  │   load_report ──────────────────┐   │
  │                                 │   │
  └─ data nahi ──► write_research_brief │
                      ▼                │
               prepare_mcp_research    │
                      ▼                │
               mcp_research_subgraph   │   ← ReAct loop (tools call karta hai)
                      ▼                │
               supervisor_subgraph     │   ← parallel researcher agents
                      ▼                │
               final_report_generation ┤
                      ▼                │
               save_report_to_file ──► END  (report files/ me save = cache)
                                         │
                      └──────────────────┘
                                 ▼
                        generate_structure   ← report ko 3 checkpoints me toda
                                 ▼
                         create_content      ← har checkpoint ka padhai material + 3 sawaal
                                 ▼
                         teach_checkpoint    ← UI pe material dikhata hai
                                 ▼
                         administer_quiz     ← ⏸️ INTERRUPT! graph ruk jaata hai
                                 ▼               (jawab aane pe Command(resume=) se chalta hai)
                         evaluate_submission ← LLM marking (pass mark 70/100)
                                 ▼
                    ┌──── CONDITIONAL ROUTER (decide_next_step) ────┐
                    │                    │                          │
              FAIL (<70)            PASS, aur                saare checkpoint
                    │              topics baaki                 complete
                    ▼                    │                          │
          simplified_teaching            ▼                          ▼
          (Feynman Technique)      create_content             course_completed → END
                    │              (agla topic)                  
                    └──────────► administer_quiz  (RETRY LOOP)
```

---

## 6️⃣ Har Node ka Deep Explanation

### 🔹 `clarify_with_user` — Scoping
- LLM ke paas structured schema `ClarifyWithUser` hai: `{need_clarification: bool, question: str, verification: str}`.
- Agar user ne "Mujhe AI ke baare me batao" jaisa vague bola hai → `need_clarification = true` → agent ek sawaal pooch kar graph **END** kar deta hai. Phir user ke next message se graph dobara chalta hai (**thread memory** ke wajah se).
- Agar info sufficient hai → `write_research_brief` pe jaata hai — poore conversation se ek **structured research brief** bana deta hai.
- **Ye `Command(goto=..., update=...)` use karta hai** — matlab ek hi return me routing + state update dono.

### 🔹 `check_data_exists` — Intelligence/Cache Layer ⭐ (unique feature)
- `files/` folder me saari purani reports (.md) padhta hai.
- **Keyword overlap** nikalta hai topic ke aur har file ke beech (`_keywords()` — 4+ letter words, stopwords hata kar).
- **Sufficiency gate:** file kam se kam **250 words** ki ho **aur** keyword match strong ho (`overlap ≥2 aur ratio ≥50%` ya `ratio ≥70%`).
- Match mila → **poora research phase skip**, seedha `load_report` → `generate_structure`. *Yaani agent apni purani research ko memory ki tarah reuse karta hai — baar baar internet search nahi karta.*

### 🔹 Research Path (jab data nahi hai)
1. **`prepare_mcp_research`** — brief ko researcher ki state me daalta hai.
2. **`mcp_research_subgraph`** — ye ek **ReAct loop** hai:
   - `mcp_llm_call` → LLM tool maangta hai
   - `mcp_tool_node` → tools chalata hai (**MCP filesystem server** + `think_tool`)
   - tool calls khatam → `mcp_compress_research` → saara raw content ek **compressed note** me badalta hai (context window bachane ke liye)
   - `think_tool` ka matlab: har search ke baad LLM ko *sochne ka forced pause* milta hai — "kya mila? kya missing hai? aur search karun ya likh dun?" — ye quality badhata hai.
3. **`supervisor_subgraph`** — **Multi-agent pattern**: ek *supervisor* agent baaki *researcher* agents ko kaam deta hai (`ConductResearch` tool), aur sab parallel chalte hain (`asyncio.gather()`). Ek `ResearchComplete` tool hai jisse supervisor bolta hai "kaam ho gaya". Max 6 iterations ka cap hai.
4. **`final_report_generation`** — sab notes jod kar ek detailed markdown report.
5. **`save_report_to_file`** — `report_<uuid>.md` naam se save. **Ye hi aage `check_data_exists` ko feed karta hai = self-improving memory.**

### 🔹 `generate_structure` — Checkpoints banana
- LLM ko `CheckpointResponse` schema diya (structured output) → report se **3-4 checkpoints** (name + objective) extract karwata hai, code me top **3** tak limit kar diye (focused session ke liye).
- Har checkpoint ke saare default fields initialize hote hain (`score=0, passed=False, ...`).

### 🔹 `create_content` — Padhai material banana
- Har checkpoint ke liye: **~100 words ka study material** + **exactly 3 questions** (`CheckpointContent` schema).
- **Lazy generation** — sirf current checkpoint ka content banta hai; agar pehle se bana hai toh LLM call skip.
- **Try/except fallback** — LLM fail ho jaaye toh objective se fallback content + default 3 questions bana deta hai (system kabhi crash nahi hota).

### 🔹 `teach_checkpoint` — Presentation
- Study material + questions ko ek nicely formatted `AIMessage` me render karta hai (📘 Checkpoint 1, 📖 Study Material, ❓ Questions).

### 🔹 `administer_quiz` — ⏸️ Human-in-the-Loop ⭐ (bahut important viva topic)
```python
user_answers = interrupt(user_view)   # GRAPH YAHIN RUK JAATA HAI
```
- `interrupt()` call hote hi **graph pause** ho jaata hai aur UI ko `{title, material, questions}` bhejta hai.
- User jawab bhejta hai → client `graph.invoke(Command(resume=answers))` chalata hai → graph **wahi se** aage chalta hai.
- **LangGraph ka checkpointer** poora state (checkpoints, index, messages) persist karta hai — isliye tab bhi chalta hai tab app band ho gayi ho.
- Retry round me material badal deta hai: `simplified_material` ho toh wahi, warna original.

### 🔹 `evaluate_submission` — Verification
- LLM ko rubric: *"Pass mark is 70 out of 100"* → `EvaluationResult` `{score, feedback, passed}`.
- **Score normalization hack:** model kabhi 10-point scale pe de deta hai, toh code check karta hai `0 < score <= 10` toh `×10` kar deta hai (defensive coding).
- Pass → index++ (agla topic). Fail → index same (usi topic pe retry).

### 🔹 Router: `decide_next_step` — Conditional Edge
```python
if idx >= len(checkpoints): return "course_completed"            # sab ho gaya
if answers hai and passed == False: return "simplified_teaching" # fail → repair
return "create_content"                                          # pass → next topic
```

### 🔹 `simplified_teaching` — 🧠 FEYNMAN PEDAGOGY ⭐ (project ka USP)
- Prompt me saara context bhejta hai: topic, original explanation, questions, **student ke galat jawab**, feedback.
- Rules: *jargon bilkul nahi, 10-year-old ko samjha jaisa, analogies + real-world examples, chhote tukdon me, aur focus sirf us concept pe jisme student fail hua.*
- Output → `simplified_material` → wapas `administer_quiz` → **retry loop**.
- **Ye Feynman Technique ka automation hai:** "kisi cheez ko itna simplify karo ki 10 saal ka baccha samajh jaaye" — kyunki jahan tum simplify nahi kar pa rahe, wahi tumhe nahi aata (yahi gaps quiz ne pakde).

---

## 7️⃣ Core LangGraph Concepts (ye samajh gaya toh viva me koi nahi hila paayega)

| Concept | Tumhare project me kahan |
|---|---|
| **State (TypedDict)** | `LearningProjectState` — shared memory jisme `report`, `checkpoints`, `current_checkpoint_index`, `notes`, `messages`... sab rehte hain. Har node state padhta hai aur updated dict return karta hai. |
| **Reducer** | `Annotated[List[str], operator.add]` = values append hoti hain replace nahi; `add_messages` = chat history badhti hai. |
| **Node** | Sirf ek Python function: `state → partial state update`. |
| **Edge** | `add_edge(a, b)` = fixed flow; `add_conditional_edges` = router function se dynamic decision. |
| **Command** | `Command(goto=..., update=...)` — routing + state update ek saath. |
| **Interrupt / Resume** | `interrupt()` + `Command(resume=)` = human-in-the-loop quiz. |
| **Checkpointer/Persistence** | Thread ID ke saath state save — conversation aage badh sakta hai. |
| **Subgraph** | `mcp_research_subgraph` aur `supervisor_subgraph` apne aap me chhote graphs hain jo main graph ke nodes hain. |
| **Structured Output** | Pydantic `.with_structured_output(schema)` — hallucination rokne ke liye LLM ko fixed format me hi bolna padta hai (`need_clarification`, `score`, `passed`...). |

**Checkpoint object ka lifecycle (state ka flow):**
```
{name, objective, ""}                    ← generate_structure
   → +study_material, +quiz_questions    ← create_content
   → +user_answers                       ← administer_quiz (interrupt se)
   → +score, +passed, +feedback          ← evaluate_submission
   → +simplified_material (agar fail)    ← simplified_teaching
```

---

## 8️⃣ Demo Flow (presentation me aise chalana)

1. `docker-compose up -d` (ya `langgraph dev`)
2. LangGraph Studio kholo: `https://smith.langchain.com/studio/?baseUrl=http://localhost:8000`
3. Graph select: **`learning_project`**
4. Message bhejo: *"Mujhe Transformer architecture samajhna hai"*
5. Dikhao: clarify → (data nahi hai) → research → report → **3 checkpoints ka roadmap** → Padhai + 3 sawaal
6. **Jaan bujh ke 1-2 jawab galat do** → next message aayega: *"💡 Let's Simplify This (Feynman Technique)"* → ye money-shot hai 🎯
7. Sahi jawab do → ✅ PASSED → agla checkpoint → end me 🎓 Complete
8. **Ek baar firse same topic bhejo** → dikhana ki *"📚 Found existing comprehensive report... Skipping research"* — caching ka proof

---

## 9️⃣ Likely Viva Questions + Ready Answers

**Q: ChatGPT aur ye agent me farq kya hai?**
> ChatGPT sirf baat karta hai. Ye agent ek **workflow/state-machine** hai — research karta hai, tools call karta hai, report save karta hai, quiz leta hai, marking karta hai, aur fail hone pe teaching strategy khud badalta hai. Har step ka state persist hota hai.

**Q: Hallucination kaise rokte ho?**
> **Structured output (Pydantic)** se — LLM free text nahi, fixed schema me hi jawab de sakta hai (boolean `passed`, int `score`). Plus research me Tavily se real sources, aur 70/100 ki objective rubric.

**Q: Feynman technique kya hai?**
> Concept ko itna simplify karo ki koi bhi layman samajh jaaye — analogies, simple bhasha, jargon hatana. Hum ise **automate** karte hain: quiz ke wrong answers + feedback de kar LLM ko simplified explanation generate karwate hain.

**Q: Context window ka problem?**
> Do tarah se handle: (1) `compress_research` se raw research → chhota note, (2) content generation me report truncate (`report[:3500]`, `[:2000]`).

**Q: Human-in-the-loop kaise?**
> `interrupt()` — graph execution ko legally pause karta hai, state checkpointer me save, `Command(resume=answers)` se aage.

**Q: Local LLM use kar rahe ho?**
> Haan — Ollama/LM Studio se local model, aur structured output support na ho toh Gemini pe automatic fallback (`get_structured_output_model`).

**Q: Agla step/future scope?**
> Zyada checkpoints, spaced repetition, multi-turn conversation memory, LangSmith pe evaluation dashboard, student progress analytics, audio/visual explanations.

---

## 🔟 30-Second Closing Line

> *"Toh basically — research agent se lekar teaching agent tak ka jo journey hai, humne use ek single LangGraph workflow me integrate kiya hai, jahan human-in-the-loop quiz, automated verification, Feynman-based remediation, aur file-based memory — ye chaaro milke ek genuinely autonomous learning system banate hain."*
