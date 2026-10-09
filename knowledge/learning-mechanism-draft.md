# Learning Mechanism — DRAFT

Status: **draft, still in discussion** (started 2026-10-08).
Goal: one reusable learning flow that agent-builder can install into any new agent.
First user: the trader agent in `../ws-agent-001` (not built yet — on hold until this design is done).

Note: we call agents **he/his**, not "it".

---

## 1. What "learning" means here

The agent is an LLM. His model weights never change, and each session starts fresh.
So learning can NOT mean "the model gets smarter inside".

**Definition:**
> Learning = building up knowledge in the workspace, from the agent's own experience
> and checked against evidence, that measurably improves his future decisions.

All three must be true:

1. **Written down outside the model.** Only files he reads next session count.
2. **Backed by evidence.** A test result, a practice log, or a cited source. Reading is not knowing.
3. **Changes behavior in a measurable way.** If metrics do not improve, he is only collecting notes.

Three kinds of knowledge:

| Kind | Example | Where |
|---|---|---|
| Knowing (facts, concepts) | "Spreads widen at market open" | `knowledge/concepts/` |
| Knowing what works (tested rules) | "Strategy X: PF 1.4 over 200 trades, fails in sideways markets" | `strategies/` (domain folder) |
| Knowing himself (calibration) | "When I say 70% sure, I am right 52% of the time" | `knowledge/lessons.md` |

How we prove he is learning: track a few numbers over time (prediction accuracy, calibration, practice results).

---

## 2. The flow (6 steps, a loop)

```
STUDY → HYPOTHESIZE → TEST → PRACTICE → REFLECT → CONSOLIDATE → (repeat)
                       ▲        ▲
                       └────────┴── the only parts that change per agent
```

| Step | Generic (same for every agent) | Trader agent | Example: support-reply agent |
|---|---|---|---|
| Study | Read sources, write concept notes with sources | Indicators, risk | Product docs, policies |
| Hypothesize | Turn a concept into a testable rule | "RSI<30 → long" | "Refund question → template B" |
| **Test** | Check the rule against past data | Backtest | Replay old tickets |
| **Practice** | Act safely; log the decision BEFORE the outcome | Paper trade | Draft replies a human approves |
| Reflect | Compare prediction with result, write the lesson | same | same |
| Consolidate | Merge, cap, archive, update the index | same | same |

**Graduation gates:** the agent moves to the next level (e.g. paper → live) only when he meets measurable targets AND a human approves.
The gates live in `AGENTS.md` and only the human edits them.

---

## 3. Token rules (use knowledge only when needed)

Pattern: **progressive disclosure** — always load a small map, open details only when the task needs them.

| Tier | What | When loaded | Budget |
|---|---|---|---|
| 0 — always | `AGENTS.md` + `knowledge/INDEX.md` | Every session | < ~2–3k tokens |
| 1 — on demand | One concept / rule / lesson file | Only when INDEX points to it | < ~1.5k tokens per file |
| 2 — search only | Raw journals, old tests, data | Never read whole; grep or scripts | Unlimited |

Rules:

1. At session start, read only `AGENTS.md` and `knowledge/INDEX.md`.
2. INDEX has one line per file with a "read when…" hint.
3. One topic per file. Split a file when it passes ~1.5k tokens.
4. Distill: raw logs → weekly review → short lessons.
5. `lessons.md` has a hard cap (e.g. 15). A new lesson replaces a weaker one.
6. Never read raw data. Run a script that returns a short summary.
7. Heavy research goes to a subagent. Only his summary comes back.
8. Start with index + grep. Add vector search only if this clearly stops working.
9. Tier 0 stays the same each session, so prompt caching makes it cheap.

---

## 4. Triggers (how to make him learn)

### Entry point: one inbox

Everything to learn goes into `learning/inbox.md` first.
Capturing an item is fast. Processing it happens later, in a learning session.

```markdown
| id | added      | by    | type    | item                                   | status  |
|----|------------|-------|---------|----------------------------------------|---------|
| 7  | 2026-10-08 | human | source  | https://…/article                      | new     |
| 8  | 2026-10-08 | human | idea    | "volume spikes before breakouts matter" | new     |
| 9  | 2026-10-09 | agent | mistake | lost trade #42, ignored news           | studied |
```

Status path: `new → studied → hypothesis → tested → practiced → lesson | rejected`.

### Four triggers

1. **Human command** (trigger phrases in `AGENTS.md`; optional Claude Code skills `/learn`, `/reflect`, `/consolidate`):

   | You say | He does |
   |---|---|
   | `learn: <url / text / idea>` | Add to inbox, confirm |
   | `study now` | Process inbox items through the flow |
   | `reflect` | Review recent results, write lessons |
   | `consolidate` | Merge, cap, archive, update INDEX |
   | `what did you learn?` | Show recent lessons and metric changes |

2. **Events during work** (automatic): human correction (most valuable), wrong prediction, surprise, outcome becomes known → add to inbox or run reflect.
3. **Schedule** (automatic): daily = process inbox + reflect; weekly = consolidate + check graduation gates.
4. **Session hooks** (safety net): start = load INDEX and show new inbox count; end = ask "anything worth adding to the inbox?".

```
 human "learn: …" ─┐
 correction/miss  ─┼──►  inbox.md ──► study → hypothesize → test → practice → reflect ──► knowledge/
 session-end hook ─┘         ▲                                                                │
                      daily/weekly schedule ─────────────── consolidate ◄─────────────────────┘
```

---

## 5. Planned files (not created yet)

```
ws-agent-000/
├── knowledge/
│   └── learning-mechanism.md        # final version of this draft
└── templates/
    └── learning-module/             # copied by agent-builder into a new agent
        ├── LEARNING.md              # step-by-step protocol + trigger table
        ├── learning/inbox.md        # inbox format
        ├── knowledge/INDEX.md       # empty index + format example
        ├── knowledge/lessons.md     # capped list + format example
        ├── journal/_template.md     # decision-before-outcome log
        └── domain-slots.md          # per-agent questions: what is TEST? PRACTICE? gates?
```

Plus one new rule for agent-builder:
"If the new agent needs to learn, install the learning module, fill in the domain slots with the user, and link `LEARNING.md` from `AGENTS.md`."

---

## 6. Open questions

1. Where to save: `knowledge/` + `templates/learning-module/` in this repo (suggested), or somewhere else?
2. Should every new agent get the learning module, or only agents marked "needs learning"?
3. Who updates the mechanism: only the human, or may agent-builder propose changes?
4. Add, remove, or rename any of the 6 steps?
5. `learn: X` → only add to inbox (suggested), and `learn now: X` → study right away?

## 7. Trader agent (on hold)

Questions to answer later, when we build `ws-agent-001`:
market (crypto / stocks / forex / IDX), data source, trading style, and manual vs scheduled runs.
Safety: paper trading only until a human approves live; risk limits set by the human in `AGENTS.md`.
