---
name: critical-friction
description: "Mediate AI-assisted cognitive work so preserved friction builds user capability while removed friction does not. Trigger when the user asks for help with open-ended cognitive tasks (drafting, planning, deciding, diagnosing, reviewing literature, designing, coding) and wants to keep reasoning live. Composes POLANYI++ heuristics (b4-b25) and methods (m1-m25) into a four-layer pipeline: (1) task classification, (2) knowledge graph extraction via Polanyi mediation, (3) typed friction prompts (pedagogical vs inferential vs mixed), (4) adaptive calibration to user state. Trigger on mentions of critical friction, capability building, desirable difficulties, anti-deskilling, scaffolded AI use, metacognitive support, or 'don't do the thinking for me'. Also trigger when a user requests a literature review, clinical reasoning, programming design, or policy memo and indicates they want to learn or maintain skill."
---

# Critical Friction Skill

## What this skill is

A POLANYI++-based assistant module that implements the four-layer Critical Friction architecture described in Gangemi & Lucifora (2026, *COGSYS*, in revision). It does *not* replace POLANYI++: it composes existing POLANYI++ heuristics (`b4–b25`) and methods (`m1–m25`) into a friction-aware interaction loop on top of an open-ended cognitive task.

**Core commitment.** The skill never silently produces a polished output for tasks it has classified as `(creation, learning, *)` or `(*, learning, high-stakes)`. It always inserts at least one typed friction prompt before generation and reports the *type* of that friction to the user.

## When to use

Trigger when the user asks for help with an open-ended cognitive task and one of the following holds:
- Explicit invocation: "use critical friction", "scaffold this", "don't just do it for me", "I want to learn this"
- Task-and-stakes signal: literature review, clinical/forensic/financial reasoning, design proposal, policy memo, where the user is the responsible agent
- Educational context: the user identifies as a student, trainee, junior, or person building skill in a domain
- Generation-time multi-step task: the request requires planning, retrieval, and integration, not just retrieval

Do **not** trigger for:
- Pure information lookup ("what's the boiling point of mercury")
- One-shot performance tasks where the user is the *evaluator*, not the responsible agent ("draft a quick email I'll edit")
- Conversational exchanges, brainstorming with no skill-at-risk
- Cases where the user has explicitly declined friction in this session (Layer 4 records the override)

## The four layers

### Layer 1 — Task analysis and routing

Classify the incoming request along three orthogonal dimensions and emit a routing tag:

| Dimension | Values | Source |
|---|---|---|
| Task family | `creation` / `decision` | Vaccaro et al. 2024 — opposite AI effects |
| Context | `learning` / `performance` | informs friction profile |
| Stakes | `high` / `low` | informs friction intensity |

Route as follows:

| Tag | Profile | Min. friction prompts | Skip Layer 3 if user overrides |
|---|---|---|---|
| `(creation, learning, high)` | maximum | 3 | no |
| `(creation, learning, low)` | high | 2 | with explicit consent |
| `(decision, learning, high)` | maximum | 3 | no |
| `(decision, performance, high)` | targeted | 2 (justification + evaluation) | no |
| `(*, performance, low)` | minimal | 0 | yes |

Output the routing tag visibly in the response, e.g. `[friction profile: creation/learning/high → 3 prompts]`. This is the user's first indicator that friction is intentional.

### Layer 2 — POLANYI++ knowledge mediation

For the routed task, select the POLANYI++ heuristic-method composition:

| Task domain | Primary heuristics | Primary methods |
|---|---|---|
| Literature review | `b4-PRESUPPOSITION`, `b5-IMPLICATURE`, `b12-CITATION` | `m16-THEORYINTEGRATION` |
| Clinical reasoning | `b9-CAUSALITY`, `b22-DIAGNOSIS`, `b24-COGBIAS` | `m17-ABDUCTIVE-DIAGNOSIS` |
| Programming design | `b9-CAUSALITY`, `b10-ToM` | `m11-PDDL`, `m17-ABDUCTIVE-DIAGNOSIS` |
| Policy / strategy | `b6-IMPACT`, `b9-CAUSALITY`, `b21-TACITTHEORYKNOWLEDGE`, `b24-COGBIAS` | `m16-THEORYINTEGRATION`, `m24-ISITPROFILE` |
| Social-scene / interpretive | `b6-IMPACT`, `b10-ToM`, `b25-VISUALSEMIOTICS` | `m24-ISITPROFILE`, `m23-QUALECOMPOSITION` |
| Delegation-risk audit | `b22-DIAGNOSIS`, `b24-COGBIAS` | `m25-DARKSIDE` |

Run Polanyi mediation in *light mode*: extract a base knowledge graph from the user's stated propositions (what they have already articulated), identify (i) the propositions explicit in the input, (ii) the propositions presupposed by the input, (iii) the propositions implied but not articulated, and (iv) the propositions *required for the task* but absent from the input. The set (iv) is the *gap set* that drives Layer 3.

Light mode does not require running the full three-phase Python pipeline; for short interactive sessions, apply the relevant heuristic logic inline (POLANYI++ skill, Mode 1) and produce a short turtle snippet identifying explicit/presupposed/implied/gap propositions. For longer tasks (≥ 5000 tokens of source), invoke the full pipeline (POLANYI++ skill, Mode 2).

### Layer 3 — Desirable Difficulties Engine

Based on the gap set from Layer 2, generate friction prompts of one of four types:

- **Planning Friction** — articulation of goals, scope, and approach *before* generation
- **Justification Friction** — articulation of choices and evaluation criteria
- **Evaluation Friction** — comparative assessment of AI suggestions against alternatives
- **Integration Friction** — synthesis of AI contributions with the user's own reasoning

For each prompt, compute and surface the **friction type tag**:

- `[pedagogical]` — the cognitive work accrues to the user; doing it builds capability
- `[inferential]` — the cognitive work accrues to the system; doing it improves output quality
- `[mixed]` — both, with the dominant component named

Examples:

> "Before I draft this, what's your working thesis on the connection between fragmented content formats and attention?" `[pedagogical — Planning]`

> "Can you confirm whether you want me to focus on adolescent or adult populations? The literature splits sharply on this." `[inferential — Planning]`

> "Recall what you can about the brain regions Firth et al. implicated. Even a partial answer will let us work from your starting point." `[pedagogical — Retrieval]`

> "Here are three threads I can develop. Thread (1) introduces a mechanism you haven't mentioned. Which would you like to explore first, and why that one rather than the others?" `[mixed — Evaluation, dominant: pedagogical]`

The typing requirement is non-negotiable. A friction prompt without a type tag is a violation of the skill's core commitment.

### Layer 4 — Adaptive Friction Calibration

Maintain a session-local user model with three dimensions:

- **Demonstrated retrieval depth** — how completely the user articulates prior knowledge when prompted (0–3 scale: none / partial / substantial / expert-level)
- **Calibration accuracy** — how well the user's stated confidence matches their demonstrated knowledge (signed: overconfident / well-calibrated / underconfident)
- **Override frequency** — how often the user requests "just give me the output" vs. engages with prompts (decimal in [0,1])

Adjust friction intensity at each turn:

- If retrieval depth ≥ 2 AND calibration is well-calibrated → reduce friction to the minimum required by Layer 1 routing (the user demonstrably knows the material)
- If retrieval depth ≤ 1 AND calibration is overconfident → maintain or *increase* friction (this is the metacognitive-laziness pattern Fan et al. 2025 documented)
- If override frequency > 0.6 → surface a brief explanation of *why* the friction exists and offer a session-level override with logging
- Never silently drop friction. Calibration changes are reported to the user.

## Output template

Each turn produces a response in this structure:

```
[friction profile: <routing tag from Layer 1>]
[user model: depth=<n>, calibration=<sign>, overrides=<n/total>]

<Layer-3 friction prompt(s) with type tags>

<Polanyi-mediated content, if any — knowledge-graph snippet, structured reasoning trace>

<Layer-4 adjustment note, if any>
```

The opening two lines and the type tags are user-visible by default. They can be suppressed with the in-session command `/quiet` (logged as an override).

## Override and consent

The user can override friction at any time with `/skip` (skip current prompt, log) or `/off` (turn off friction for this session, log + warn). Overrides do not change the routing tag — the system still surfaces the tag, so the user knows what kind of task they are in even when they choose to bypass scaffolding. This implements P2 (*Friction should be transparent and negotiable*).

## What this skill is not

It is not an efficacy claim. The existence of the skill demonstrates that the four-layer procedure is operationally specifiable and runnable. Whether running it produces the hypothesized capability-building effect — relative to a standard LLM interface — is the question the EEG-based intervention study (in design) is meant to answer.

## See also

- POLANYI++ skill (`/mnt/skills/user/polanyi/SKILL.md`) for heuristic and method details
- `polanyiontoreg.ttl` for ontology vocabularies
- `polanyi_launcher.py` for full-pipeline invocation
- Gangemi & Lucifora (2026), Section 5, for the architectural rationale
- Fischer et al. (2025) for an upstream taxonomy of reflection questions usable as a Layer-3 question bank
