# Roadmap — AI Tutor

Each phase produces something a student can use for a week. The exit test is a
learning-behaviour test, not a feature count.

---

## Phase 0 — Skeleton

- Auth, student profile, guardian consent flow for minors
- Curriculum model: board, grade, subject, topic, concept, prerequisite links
- One subject seeded fully (suggestion: Mathematics, Grade 9) with a concept graph
  and 100 verified items
- Session and turn storage

**Exit test:** a student signs up, picks a board and subject, and sees the concept
map for their grade.

---

## Phase 1 — Diagnostic

- Adaptive placement per topic (mastery estimate, not a score)
- Prerequisite-aware report ("you are stuck on X because Y is weak")
- Item selection by estimated mastery

**Exit test:** placement on a synthetic student with a known gap correctly names
the prerequisite gap, not the surface topic.

---

## Phase 2 — Socratic tutoring

- State machine: probe → hint ladder → check → consolidate
- Structured session state and prompt builder
- One concept at a time; first response is a question
- Transcript review tool for the teaching team

**Exit test:** across 20 sessions, no session opens with a lecture, and every
hint follows the ladder in order.

---

## Phase 3 — Verified practice

- SymPy step checker; first-divergent-step localisation
- Item generator constrained by concept templates, verified before entry
- Formats: numeric, steps, MCQ, explain-it-back with rubric
- Mock-paper mode

**Exit test:** 500 generated items pass verification; a scripted set of wrong
steps is caught at the correct line every time.

---

## Phase 4 — Spaced repetition

- FSRS-style scheduler per concept and item
- Daily plan with time caps
- Hint-adjusted grading
- Burnout guard

**Exit test:** a two-week simulated student shows reviews returning at
increasing intervals; a failed concept re-enters its prerequisite.

---

## Phase 5 — Bilingual

- Urdu and Roman Urdu concept content for the seeded subject
- Per-turn language detection and consistent glossary
- Voice mode (optional)

**Exit test:** the same concept is taught in all three registers, and a student
switching mid-sentence is followed, not corrected.

---

## Phase 6 — Teacher dashboard

- Class mastery heatmap, misconception clusters
- Assign concepts and papers; completion and re-attempt views
- Evidence-backed flagging for human conversation

**Exit test:** a teacher identifies the class's top misconception in under two
minutes and assigns a 10-minute practice set.

---

## Phase 7 — WhatsApp channel and parent report

- WhatsApp session flow on the same state machine
- Weekly parent summary with no invented grades or comparisons
- Opt-in/opt-out for every message

**Exit test:** a student runs a full week primarily from WhatsApp; the parent
receives one accurate weekly summary.

---

## Not yet (deliberate)

- Replacing teachers or producing school grades
- Public leaderboards (motivating for some, harmful for many)
- Multi-subject breadth before one subject is genuinely good
- Voice-first product before text is strong
- Training on student conversations for model improvement — requires consent and
  governance; the default is no

---

## Definition of done

1. Learning behaviour tested, not just UI: does the student reason more?
2. Maths never unverified; every step checked deterministically.
3. No minor's data leaves the deployment without consent.
4. Curriculum content versioned; corrections never rewrite history.
5. Documentation updated with the learning-design rationale, not only the code.
