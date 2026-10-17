# Architecture — AI Tutor

Companion to the [README](../README.md). Focused on the parts that decide whether
the tutor actually teaches: session state, the hint ladder, answer verification,
and the scheduler.

---

## 1. Domain model

```
student ── enrols ──► programme (board, grade, subject)
                        └─► syllabus ──► topic ──► concept ──► prerequisite ──► concept
concept ──► item (question, answer, difficulty, format)
student × concept ──► mastery (estimate, confidence, last_seen, next_review)
session ──► turn (role, content, hint_level, concept, verification)
item_attempt ──► outcome, working, time, hints_used
```

The **concept graph** is the backbone: assessment, scheduling, and teaching all
read from it. A concept has a short canonical explanation, worked examples, common
misconceptions, and links to the syllabus outcome it satisfies.

---

## 2. Tutoring state machine

```
            ┌──────────┐
   start ──►│  probe   │◄──────────────┐
            └────┬─────┘               │
                 │ student responds    │
            ┌────▼─────┐  correct   ┌──┴──────────┐
            │  check   ├───────────►│ consolidate │
            └────┬─────┘            └─────────────┘
                 │ incorrect / unclear
            ┌────▼─────┐
            │   hint   │ level 1 → 2 → 3 → 4
            └────┬─────┘
                 │ still stuck at level 4
            ┌────▼──────────┐
            │ teach fully,  │
            │ then re-probe │
            └───────────────┘
```

Hint ladder:

| Level | What the tutor does | Effect on mastery credit |
|---|---|---|
| 1 | Points at the relevant idea ("what happens to the power when you divide?") | small |
| 2 | Narrows to the exact rule, no application | moderate |
| 3 | Worked example on a *different* problem, same rule | large |
| 4 | Full explanation of this problem | item counts as taught, not mastered |

Rule: never jump more than one level at a time. A student who solves after level 1
demonstrated more than one who needed level 4 — the mastery update reflects that.

---

## 3. Session state (stored, not inferred)

```json
{
  "session_id": "s_...",
  "student_id": "st_...",
  "concept_id": "c_linear_rearrange",
  "misconception_hypothesis": "sign_error_when_moving_terms",
  "hint_level": 1,
  "hints_used": 2,
  "worked_examples_shown": 0,
  "items_attempted": ["i_1043", "i_1091"],
  "language": "roman_urdu",
  "updated_at": "2026-10-17T12:00:00Z"
}
```

Prompts are built from this object plus the retrieved concept chunk. That is what
keeps the tutor coherent across turns, devices, and the WhatsApp/web split.

---

## 4. Answer verification

The model never decides whether maths is correct.

```
student step ─► normalise (parse, unit check)
             ─► compare with SymPy (symbolic equality, not string equality)
             ─► if equal: correct
             ─► if not: locate the first divergent step, compare
                        "what changed between line 2 and line 3?"
             ─► generate a hint about that specific transition
```

- Numeric answers: tolerance-based equality.
- Algebraic answers: `simplify(expr - expected) == 0`.
- Word answers: rubric grading by the model, with required ideas listed explicitly
  and a "cannot assess" path when the answer is off-rubric.
- Question generation: every generated item runs through SymPy before entering
  the bank — solution exists, unique integer/nice-form answer where required,
  difficulty within bounds.

---

## 5. Scheduler (FSRS-style)

Per `(student, concept)` and per `(student, item)`:

- Retrievability estimate decays with elapsed time and stability.
- On review: success raises stability; failure lowers it and re-enters the
  prerequisite concept.
- Daily plan = due reviews first (cap at N minutes), then the next concept on the
  syllabus whose prerequisites are all ≥ threshold, then free practice.
- Hints adjust the grade used for scheduling — a heavily hinted success is not the
  same evidence as an unaided one.

Burnout guard: if a student's accuracy drops across three sessions, the plan
reduces new material and increases review — silently, without a "you're falling
behind" message.

---

## 6. Curriculum retrieval

- Each concept chunk: canonical explanation, 2 worked examples, common
  misconceptions, syllabus outcome code.
- Embedded with a multilingual model so Urdu and English queries retrieve the same
  concept.
- The tutor prompt includes the concept chunk and is instructed to teach *from*
  it; if the question is outside the syllabus, it says so and routes to a human
  or defers.
- Content is versioned; a correction to a concept creates a new version, and past
  sessions keep the version they used.

---

## 7. Bilingual behaviour

- Detects the language of the student's turn; responds in it.
- Vocabulary glossary ensures a term is translated consistently.
- Numerals and symbols stay conventional in both languages.
- Roman Urdu is treated as its own register, not transliterated Urdu.

---

## 8. Safety and integrity

- Minors: guardian consent required at signup; no ads; no public profiles.
- Data minimisation: no unnecessary personal data; transcripts encrypted.
- Homework integrity: a graded-question detector routes to "teach the method"
  mode and refuses the final answer.
- Emotional safety: if a student expresses distress, the tutor stops teaching,
  responds supportively, and surfaces a human/helpline path configured by the
  operator.
- Content review: generated items and explanations are sampled and reviewed by a
  teacher before reaching the bank.

---

## 9. Failure modes

| Failure | Answer |
|---|---|
| Model gives wrong maths | SymPy check gates every step; wrong steps never pass |
| Model drifts off curriculum | Retrieval-bound prompts; out-of-syllabus questions are named |
| Hint ladder bypassed | State machine enforces one level at a time |
| Student guesses to game hints | Hint use lowers mastery credit; guessing is visible in timing |
| Review list grows unmanageably | Daily cap; scheduler prioritises by decay risk |
| Student switches language mid-session | Language detected per turn; state carries the preference |
