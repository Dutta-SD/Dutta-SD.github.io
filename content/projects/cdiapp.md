---
title: "Plant Disease Triage"
date: 2022-09-01
lastmod: 2026-05-31
summary: "A vision-LLM triage tool for plant disease — schema-validated outputs, no chemical dosage advice, escalation to local extension officers. Originally a Smart India Hackathon 2022 finalist; currently being rebuilt as a calibrated, evaluable system."
tags: ["machine-learning", "vision-llm", "responsible-ai", "chainlit", "in-progress"]
weight: 1
---

A chat tool that takes a leaf photo and returns a structured triage report — likely disease, confidence, visible symptoms, severity, and next steps. **It does not prescribe treatments.** Hallucinated agrochemical dosages are a physical-world harm vector, so the system identifies likely problems and routes uncertain cases to a local extension officer (KVK) instead of recommending pesticides.

Originally built for **Smart India Hackathon 2022** as a CNN-plus-LLM remedy app and shortlisted for the national finals. Currently being rebuilt as a calibrated, evaluable system suitable for a real pilot.

**Live:** [sdutta28-crop-disease-diagnosis.hf.space](https://sdutta28-crop-disease-diagnosis.hf.space) · **Code:** [GitHub](https://github.com/Dutta-SD/CropDiseasePredictionApp)

### Current state — Phase 1 (shipped May 2026)

The pre-rebuild app was an ~80 LOC prompt wrapper around a free hosted vision LLM. Phase 1 takes it from "wrapper" to "defensible":

- **Triage reframe.** Stripped all chemical, dose, and brand-name recommendations from the system prompt. Every response carries a permanent disclaimer footer pointing the user to their local KVK or a qualified agronomist.
- **Pydantic schema contract.** LLM responses are parsed against a discriminated union (`PlantDiagnosis | NotAPlant`) with confidence and severity enums. The parser is tolerant of fenced-JSON wrappers and prose preambles. On a schema violation, the system retries once with a corrective message that includes the assistant's bad reply and the validation error; on a second failure, a typed `SchemaViolationError` surfaces and the chat handler renders a clean error instead of garbage.
- **Server-side rendering.** Diagnoses are rendered to markdown server-side, which lets the disclaimer footer ride on every response without depending on prompt obedience.
- **UI polish.** Replaced brittle Material-UI class-hash CSS with Chainlit's actual shadcn/Tailwind theme variables, custom green theme, custom logo, and four leaf-themed starter prompts.
- **Lint and format hygiene.** Added Ruff (replaces isort with the same author's tool) covering pycodestyle, pyflakes, isort, bugbear, pyupgrade, and simplify rule sets. Auto-fixes applied across the codebase. CI-ready.

### What's next — Phase 2: own a model, measure it honestly

The current app still depends on a hosted vision LLM. The signature differentiator is bringing in a model I've trained myself, paired with an evaluation harness that doesn't lie.

- **DINOv2 linear-probe classifier.** Train a linear head on cached DINOv2-ViT-S/14 features over a unified 20–25-class taxonomy reconciled across PlantVillage, PlantDoc, and Cassava Leaf Disease. Export to ONNX, serve CPU-side in the same Hugging Face Space. Picked over a full fine-tune because the taxonomy reconciliation work — class merges, label-provenance notes, cleanlab-surfaced label errors — is the boring 60% of vision work and the most credible part to talk about.
- **Five-slice eval harness.** Score the classifier and the LLM separately on five slices (in-distribution holdout, in-the-wild, mobile photos, OOD non-leaf, and adversarial bad photos I take myself). Report top-1, macro-F1 with **per-class binomial 95% CIs**, ECE with reliability diagrams, and OOD AUROC. **Never average across slices** — aggregate accuracy on PlantVillage is a lie.
- **Confidence router with abstention.** Two-stage pipeline: classifier first, with post-hoc temperature scaling. High confidence → classifier label, LLM only formats the explanation. Low confidence → abstain ("uncertain — please retake or escalate to KVK"). Mid-band → LLM with classifier's top-3 as context.
- **ADRs.** Real, dated architecture decision records written as decisions are made — for the linear-probe choice, the abstention design, the triage framing, etc.

### What's next — Phase 3: real users and closed-loop evaluation

A polished demo is not impact. Phase 3 is about putting it in front of real farmers and feeding their feedback back into the eval set.

- **KVK partnership + WhatsApp pilot.** Partner with a Krishi Vigyan Kendra to deploy the tool over Twilio's WhatsApp channel to ~8 smallholder farmers. The system escalates uncertain cases directly to the KVK's existing number — the tool is a triage layer in front of human expertise, not a replacement for it.
- **Closed-loop telemetry.** Thumbs-up / thumbs-down on every response, persisted to a private Hugging Face Dataset. A weekly GitHub Action pulls thumbs-down samples, replays them through the current and a candidate prompt, and posts a structured diff. The eval set grows from real failures, not synthetic ones.
- **Public retrospective.** A 1500–2000 word write-up after the pilot covering what shipped, the in-the-wild eval gap, and the one decision I'd make differently.

### Why triage, not treatment

The very first time I tested an earlier version of this on a real photo from a relative who farms in West Bengal, the model confidently named a disease the leaf did not have and recommended a fungicide dose. That moment is what turned this from "leaf classifier app" into a triage tool. A system that returns "uncertain — please escalate" is more useful and more honest than one that hallucinates a chemical name with 30% confidence.

### Stack

Chainlit · Python · OpenRouter (vision LLM) · Pydantic · Docker on Hugging Face Spaces · Ruff · *(planned: DINOv2, ONNX Runtime, scikit-learn, torchmetrics, netcal, cleanlab, Twilio WhatsApp)*
