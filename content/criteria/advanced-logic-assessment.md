---
name: Advanced Logic and Assessment Capabilities
type: criteria
tags: [criteria, branching, assessment, logic, simulation, adaptive-learning]
---

# Advanced Logic and Assessment Capabilities

## Definition

Advanced Logic and Assessment covers a tool's ability to **build complex branching scenarios, implement variable-driven adaptive learning, create sophisticated assessment question types, and enable simulation-style interactions**. This includes trigger/action systems, question banks, remediation branching, conditional paths, and integration with LMS gradebook/tracking.

**Why it matters**: Simple click-through courses have minimal learning impact. Evidence-based instructional design requires branched scenarios, realistic practice environments, adaptive assessments, and feedback loops. Teams building compliance simulations, sales training, or clinical decision-making courses need deep logic capabilities.

---

## Rankings

| Tool | Score | Notes |
|------|-------|-------|
| [[tools/articulate-360\|Articulate 360]] | ⭐⭐⭐⭐⭐ (5) | Storyline triggers/variables; unlimited branching; question banks |
| [[tools/lectora\|Lectora]] | ⭐⭐⭐⭐⭐ (5) | Scripting/action system; JS access; accessibility-compliant assessments |
| [[tools/dominknow\|dominKnow]] | ⭐⭐⭐⭐ (4) | Complex branching; software simulations; test branching |
| [[tools/ispring-suite\|iSpring Suite]] | ⭐⭐⭐⭐ (4) | 14+ question types; randomization; question banks; TalkMaster scenarios |
| [[tools/mindsmith\|Mindsmith]] | ⭐⭐⭐ (3) | Conditional logic; AI short-answer grading; basic branching |
| [[tools/parta-io\|Parta.io]] | ⭐⭐⭐ (3) | Standard assessment types; no advanced branching documented |
| [[tools/easygenerator\|Easygenerator]] | ⭐⭐⭐ (3) | Standard types; basic branching; adequate for knowledge checks |
| [[tools/elucidat\|Elucidat]] | ⭐⭐⭐ (3) | Standard types; template-driven interactions; limited custom logic |
| [[tools/gomo-learning\|Gomo Learning]] | ⭐⭐⭐ (3) | Standard quizzes; basic branching; xAPI tracking is strong |

---

## Key Insight

> **Articulate Storyline 360 and Lectora are in a separate tier** for advanced logic. Storyline's trigger/variable system is Turing-complete in practice — any interaction imaginable can be built. Lectora's action/condition scripting system plus JavaScript access achieves the same result with a different paradigm. Both are designed for experienced instructional designers; neither is approachable for casual authors.
>
> **Mindsmith's AI short-answer grading is a unique innovation** in assessment: the LLM evaluates open-ended responses against rubrics, enabling authentic assessment types that rule-based logic cannot support.

---

## Logic System Comparison

| Tool | Logic System | Variables | JS Access | Branching Depth |
|------|-------------|-----------|-----------|-----------------|
| Articulate Storyline | Trigger/Variable/Condition | ✅ | ✅ | Unlimited |
| Lectora | Action/Condition/Script | ✅ | ✅ | Unlimited |
| dominKnow | Branching + Sim engine | ✅ | Limited | Deep |
| iSpring Suite | Quiz branching + TalkMaster | Limited | ❌ | Moderate |
| Mindsmith | Conditional logic + AI grading | Limited | ❌ | Basic |
| Others | Template-based | ❌ | ❌ | Basic |

---

## Assessment Question Types

| Type | Articulate | iSpring | Lectora | dominKnow | Others |
|------|-----------|---------|---------|-----------|--------|
| Multiple choice | ✅ | ✅ | ✅ | ✅ | ✅ |
| Multiple select | ✅ | ✅ | ✅ | ✅ | ✅ |
| Drag and drop | ✅ | ✅ | ✅ | ✅ | Partial |
| Hotspot | ✅ | ✅ | ✅ | ✅ | Partial |
| Short answer (AI graded) | ❌ | ❌ | ❌ | ❌ | Mindsmith only |
| Free text | ✅ | ✅ | ✅ | ✅ | Partial |
| Matching | ✅ | ✅ | ✅ | ✅ | Partial |
| Sequence/ordering | ✅ | ✅ | ✅ | ✅ | Partial |
| Question banks | ✅ | ✅ | ✅ | ✅ | Limited |
| Adaptive remediation | ✅ | ✅ | ✅ | ✅ | ❌ |

---

## Tool Commentary

### Articulate 360 (Storyline)
The trigger/variable system is the most flexible in this comparison. Variables can track any learner action, drive conditional content display, and create fully adaptive learning experiences. Storyline is the tool of choice for complex simulation-based learning. Question banks with randomization, result slides, and SCORM tracking are mature and reliable.

### Lectora
Action/condition scripting provides a similar depth to Storyline but with a different UI paradigm. The addition of JavaScript access makes Lectora extensible beyond what its native features provide. Particularly strong for accessibility-compliant assessments with full keyboard navigation support.

### dominKnow
Combines branching scenario design with software simulation capture (screen-based walkthroughs). Strong for IT training, software adoption, and procedure-based learning that requires showing learners exactly how to perform tasks in a system.

### iSpring Suite (Max)
TalkMaster (Suite Max) enables branching dialogue/roleplay scenarios. 14+ quiz question types with robust randomization and scoring. A strong assessment platform for PPT-familiar authors who don't need full trigger-based logic.

### Mindsmith
AI short-answer grading is genuinely innovative — no other tool in this comparison uses LLM evaluation for assessment. Conditional logic handles basic branching. The ceiling for complex logic is lower than the top-tier tools.
