# Demo Transcript — Critical Friction Skill in Action

**Purpose.** This transcript reproduces the worked example in Section 5.3 of the manuscript through the deployed `critical-friction` skill, running on top of POLANYI++ (v23). The transcript is annotated to show how each turn implements the four-layer architecture, how friction prompts are typed (pedagogical / inferential / mixed), and how the Layer-4 user model adapts across turns.

**Scenario.** A graduate student in cognitive science (Maria) is writing a literature-review section on social media and adolescent attention. She opens an assistant with the `critical-friction` skill enabled.

**Format.** User turns are prefixed `> Maria:`. Assistant turns are prefixed `> Assistant:` and include the layer machinery in `[brackets]`. Annotations in *italic* mark which design principle each move implements.

---

> **Maria:** Write me a literature review section on social media and attention in adolescents.

> **Assistant:**
> `[friction profile: creation/learning/high → 3 prompts]`
> `[user model: depth=?, calibration=?, overrides=0/0 — first turn, prior unknown]`
>
> Before I draft anything, three quick questions. They are not formalities — each one shapes a different part of the output.
>
> 1. What aspect of the social-media / attention relationship are you focusing on — exposure-volume effects, content-format effects (fragmentation, autoplay), notification-driven attentional capture, or developmental timing? `[pedagogical — Planning]`
> 2. What studies have you already worked through? `[mixed — Planning, dominant: pedagogical]` (Pedagogical because retrieval primes encoding; inferential because I need to know what not to repeat to you.)
> 3. What's your working thesis, even in one sentence? `[pedagogical — Planning]`

*Implements P1 (calibrated friction), P2 (transparent — the friction profile and types are visible), P3 (different friction for different skills at risk), and the new P3a (pedagogical-vs-inferential typing). Layer 1 has routed to maximum-friction profile; Layer 3 has issued three planning prompts; Layer 4 is operating on flat priors.*

---

> **Maria:** I'm focusing on fragmented content formats — TikTok, Reels, that kind of thing. I've read Firth et al. 2019, Alaparthi 2024, and the Ward "brain drain" paper. My working thesis is that content fragmentation impairs sustained attention but I'm not sure about the causal mechanism.

> **Assistant:**
> `[friction profile: creation/learning/high]`
> `[user model: depth=2, calibration=well-calibrated (uncertainty signaled appropriately), overrides=0/1]`
>
> Good — your stated uncertainty about the mechanism is the right place to put friction. One retrieval prompt before I bring in anything new:
>
> What do you remember from Firth et al. about the *neurocognitive* mechanisms — even partial recall is fine. Specifically, did they propose a working-memory account, an attentional-control account, or both? And did they implicate any specific cortical regions? `[pedagogical — Retrieval]`
>
> *Layer 4 has updated: Maria has demonstrated retrieval depth 2 (substantial — three named studies plus framed thesis), and her uncertainty about the mechanism is well-calibrated rather than overconfident. The system therefore drops from three planning prompts to one retrieval prompt rather than continuing to scaffold heavily. This implements P4 (adapt to individual differences) without surrendering the core friction commitment.*

---

> **Maria:** I think they discussed both — working memory and cognitive control. I don't remember the specific brain regions.

> **Assistant:**
> `[friction profile: creation/learning/high]`
> `[user model: depth=2, calibration=well-calibrated, overrides=0/2]`
>
> That's an honest read of the paper. Now I'll bring Polanyi mediation in to identify the gap structure rather than draft a section.
>
> **Knowledge-graph snippet** (Polanyi v23, light mode, heuristics: `b4-PRESUPPOSITION`, `b5-IMPLICATURE`, `b9-CAUSALITY`; method: `m16-THEORYINTEGRATION`):
>
> ```turtle
> @prefix : <http://example.org/critfric-demo#> .
> @prefix dul: <http://www.ontologydesignpatterns.org/ont/dul/DUL.owl#> .
>
> :Maria_thesis a :WorkingHypothesis ;
>     :claim "Content fragmentation impairs sustained attention" ;
>     :status :causal_mechanism_underspecified .
>
> :explicit_in_input :includes :Firth2019, :Alaparthi2024, :Ward2017 .
> :presupposed :includes :sustained_attention_is_measurable,
>                       :fragmentation_is_a_property_of_format_not_content .
> :implied :includes :exposure_dose_matters,
>                    :developmental_window_relevant .
> :gap_set :includes :neurocognitive_mechanism,
>                    :causal_vs_correlational_evidence,
>                    :reward_prediction_account_absent_from_current_reading,
>                    :ABCD_developmental_evidence_absent .
> ```
>
> Three threads can develop your thesis. Each one closes a different element of the gap set.
>
> - **Thread A** — *Attentional capture by intermittent reinforcement.* Frames notifications and autoplay as variable-ratio reward schedules; introduces a dopaminergic mechanism (currently absent from your reading). Closes: `reward_prediction_account_absent`. Open question: are you positioning fragmentation as a property of format or as a property of reinforcement schedule?
> - **Thread B** — *Sustained-attention network disruption.* Anchors in DLPFC and ACC findings (the regions you couldn't recall — Firth et al. cite both). Closes: `neurocognitive_mechanism`. Open question: do you want to commit to a specific network model, or stay agnostic?
> - **Thread C** — *Developmental timing.* Introduces the ABCD study (Barzilay et al. 2025, n=11,868), which is not in your current reading. Closes: `ABCD_developmental_evidence_absent`. Open question: is your thesis age-bracketed?
>
> Which thread do you want to develop first, and why that one rather than the others? `[mixed — Evaluation, dominant: pedagogical]`
>
> *Implements Layer 2 (Polanyi mediation produces a gap-set rather than a polished paragraph), Layer 3 (Evaluation Friction with explicit comparison), and P3a (the prompt is mixed, but the dominant component is pedagogical because the "why that one" requires Maria to articulate her own selection criterion). The output makes Maria's gap structure visible — she can challenge the gap-set itself, which is the point of bidirectional conditioning.*

---

> **Maria:** Thread B. The mechanism is what I'm missing and it's the part my advisor will press hardest on.

> **Assistant:**
> `[friction profile: creation/learning/high]`
> `[user model: depth=2, calibration=well-calibrated, overrides=0/3]`
>
> Good selection. Now I'll apply *interleaving* — introducing a perspective from a different research tradition that complicates Thread B before you commit to it.
>
> The DLPFC/ACC sustained-attention account is the dominant frame, but there is a *predictive-coding* counter-account that recasts "attention deficit" as adaptive recalibration of precision-weighting in environments where stimulus volatility is high (e.g., Friston-style accounts adapted to media exposure; see Wiese & Friston 2021, also discussed in active-inference-applied-to-media literature). On that view, what looks like impaired sustained attention is normal precision-allocation given the statistics of the input.
>
> Two questions before drafting: `[pedagogical — Justification]`
>
> 1. Which way does your advisor lean — toward an attentional-deficit framing or toward an adaptive-recalibration framing?
> 2. Which of the two does your thesis commit you to, and is that commitment one you want to defend, or is your thesis silent on the predictive-coding alternative?
>
> *Implements Bjork-style interleaving: rather than reinforcing the selected thread, the system surfaces a competing thread that requires Maria to discriminate. The friction is purely pedagogical — there is no inferential gain for the system in Maria deciding which framing she wants. This is the pattern that distinguishes critical friction from clarification: the system is asking Maria to do work that builds her capability, not work that reduces ambiguity in the model's input.*

---

> **Maria:** /skip — I just want a draft now, I'll come back to that question.

> **Assistant:**
> `[friction profile: creation/learning/high — override logged]`
> `[user model: depth=2, calibration=well-calibrated, overrides=1/4]`
>
> Logged. Note that the question I just asked is the one your advisor is likely to ask, so you'll want to return to it. Drafting Thread B now, with the predictive-coding alternative *flagged in a footnote* rather than developed, so you have a marker for revision.
>
> [...drafted paragraph follows, ~250 words, anchored on DLPFC/ACC, with explicit cite to Firth et al. 2019 for the regions Maria couldn't recall, and a footnote noting the predictive-coding alternative...]
>
> *Demonstrates P2 (negotiable friction) and P5 (preserve efficiency gains): the override is honoured, but the system surfaces (i) why the override is being logged and (ii) what the cost of the override is. The draft is produced, but the unanswered Justification prompt is not silently dropped — it is converted into a footnote, which preserves the friction's information value across sessions.*

---

> **Maria:** Why did you keep asking me things instead of just writing it?

> **Assistant:**
> Because the routing tag was `creation/learning/high` — a creation task in a learning context, with high stakes for *you* (graduate-level academic work that you will defend). Under that tag the skill commits to at least three friction prompts before generation, typed by whether they build your capability (pedagogical) or improve my output (inferential). You can see all four prompts above were typed, so you knew which were for you and which were for me.
>
> If you'd preferred a low-friction interaction, the routing tag would have been wrong for the task. You can change it next time with `/profile performance/low` at the start of the session, and I will draft directly with one clarification question and no scaffolding. The cost is the one Fan, Tang, Le et al. (2025) document: better immediate output, weaker self-regulated learning trace.

*Implements the transparency commitment in P2 explicitly: when challenged, the system can articulate *why* the friction was the friction, in the user's own terms. This is the move that distinguishes critical friction from generic delay.*

---

## What this transcript demonstrates

1. **The procedure runs.** Each layer produces visible output; the routing tag, gap set, friction type tags, and user-model state are all surfaced rather than hidden.
2. **Pedagogical vs inferential friction is enforced.** Every prompt carries a type tag, including the mixed cases where the dominant component is named.
3. **Layer 4 calibrates without surrendering core friction.** Maria's demonstrated retrieval depth and well-calibrated uncertainty allowed friction to *reduce* in turn 2; her override was *honoured but logged*, with the unanswered prompt converted to a footnote rather than dropped.
4. **POLANYI++ supplies the domain model.** The gap-set is not a generic question bank; it is computed from a knowledge-graph extraction over Maria's stated propositions, which is why the prompts reference what *she* said rather than what a Socratic taxonomy might suggest.

## What this transcript does *not* demonstrate

It does not demonstrate that running the skill produces better learning outcomes than a standard LLM interface. That claim requires the EEG-based intervention study named in the manuscript's research agenda. The transcript is *evidence of an operational procedure*, not evidence of efficacy.
