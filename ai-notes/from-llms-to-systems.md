# From "Using LLMs" to "Building AI Systems": What I Finally Understood

When I first started learning about large language models (LLMs), my mental model was simple:

*If the model is strong enough, it should handle everything.*

That assumption led me to a series of confusions—especially when people talked about "sovereign AI," "auditable systems," or what I now think of as the second layer of AI systems.

This article is my attempt to clarify what I misunderstood, and how I now differentiate between model-centric and system-centric approaches to AI—and why the difference matters far more than I initially realized.

---

## 1. The Question That Started It All

My journey into this topic didn't begin with system architecture. It began with a simple question:

*Which countries have their own LLMs?*

The answer surprised me—not because of who was building models, but because of how differently countries approached the problem. What emerged was roughly three tiers:

**Tier 1: The foundation model builders.** The US and China are racing to build the most capable general-purpose models—GPT, Claude, Gemini, DeepSeek, Qwen. The focus here is raw capability: reasoning, generation, multimodal understanding. This is the AGI race.

**Tier 2: The sovereign system builders.** Countries like France (Mistral), Germany, Switzerland, and India aren't primarily trying to out-scale GPT. Instead, they're building AI ecosystems with national strategic goals: data sovereignty, regulatory compliance, and systems they can audit and control. France's Mistral, for example, is notable not just as a model but as part of a broader European push for AI independence.

**Tier 3: The cultural and regional model builders.** Countries like Japan, South Korea, Portugal, and parts of Latin America are developing LLMs focused on serving their specific languages and cultural contexts. These aren't competing with GPT on general benchmarks—they're solving a different problem: ensuring their languages and communities aren't left behind by English-centric AI. Their value lies in *local fit*, not global scale.

What caught my attention was Tier 2. These countries seemed to be asking a fundamentally different question. While Tier 1 asks *"How do we make the smartest model?"*, Tier 2 asks *"How do we build systems we can trust, control, and audit?"*

That distinction—between building smarter models and building trustworthy systems—turned out to be the key insight that reshaped how I think about AI engineering.

---

## 2. My Initial Assumption: LLM = The System

When I first worked with LLMs, I thought an AI system looked like this:

```
input → LLM → output
```

This is how most demos and APIs work. You give a prompt, and the model gives you an answer. It feels powerful, even magical.

So I assumed:

- The LLM does the reasoning
- The LLM makes decisions
- The LLM explains itself

And if it can already explain its reasoning, then why do we need anything else?

---

## 3. The First Realization: Explanation ≠ Auditability

One of my biggest misunderstandings was this:

*"If an LLM can explain its reasoning, isn't that already auditable?"*

The answer is: no.

What I didn't realize is that the "reasoning" from an LLM is generated text—it is not a record of what actually happened internally. The explanation may change with different prompts, may not reflect the true decision path, and cannot be reliably verified or reproduced. This is often called *post-hoc rationalization*: the model generates a plausible explanation after the fact.

A common counterargument is: *"What about Chain-of-Thought (CoT) prompting? Doesn't that make reasoning transparent?"* CoT does improve output quality by encouraging step-by-step generation. But here's the crucial distinction: CoT is still a generated token sequence. It is not an execution log. You cannot independently verify that the model "followed" those steps the way you can trace a function call stack in traditional software. The "reasoning" in CoT is what the model predicts reasoning *should look like*, not a faithful record of its internal computation.

Compare this to a traditional ML model like a gradient-boosted decision tree. When you compute feature importance from such a model, values like split gains and information gain are derived directly from the model's structure. Even post-hoc explanation methods like SHAP, while computed externally, are grounded in a mathematically rigorous framework—not generated text. That's a fundamentally different kind of transparency.

So even if the answer sounds reasonable, it is not traceable, verifiable, or reliable for compliance.

---

## 4. What "Auditable" Actually Means

I used to think "auditable" meant "well-explained." Now I understand it's much stricter.

**Auditable = the system can be traced, reproduced, and verified by others.**

An auditable system must include:

**Traceability:** What was the input? Which model version was used? What steps were executed?

**Reproducibility:** Can we run the same process again and get the same result?

**Verifiability:** Can a third party check whether the decision followed the rules?

This is not something an LLM can provide on its own.

---

## 5. The Core Shift: Model-Centric → System-Centric Thinking

This is the most important conceptual upgrade I had.

**Before (Model-Centric):**

```
input → model → output
```

**After (System-Centric):**

```
input
 → data validation
 → model(s)
 → rules / logic
 → post-processing
 → logging / tracing
 → output
```

The key difference: **the model is no longer "the system."** It becomes just one component inside a system.

This maps directly back to the national AI strategies I described earlier. Tier 1 countries are investing in the model—making the "brain" as powerful as possible. Tier 2 countries are investing in the system—building the infrastructure, governance, and audit layers that make AI deployable in regulated environments. Both are necessary, but they reflect fundamentally different design philosophies.

---

## 6. Model-Centric vs. System-Centric (My Current Framework)

### Model-Centric Systems

These systems focus on capability.

Typical architecture:

```
user → LLM → answer
```

**Strengths:** Flexible, powerful, fast to build.

**Weaknesses:** Not controllable, not auditable, hard to deploy in regulated environments.

Note: I originally called this "AGI-oriented," but that's misleading. A simple ChatGPT API call is model-centric, but no one would call it an AGI system. The defining characteristic isn't ambition—it's that the model owns the entire decision pipeline.

### System-Centric Systems

These systems focus on control and reliability.

Core principle: *Don't let the model own the entire decision.*

```
user
 → LLM (parse input)
 → tools / database / ML models
 → rule engine
 → LLM (generate output)
 → logging system
```

**Key properties:** Every step is observable. Decisions are structured. Behavior is constrained.

---

## 7. Do System-Centric Systems Still Use LLMs?

This was another confusion I had. I wondered:

*"If everything is controlled, are LLMs even used?"*

The answer is: yes—but differently.

LLMs are typically used for *interface tasks*: parsing user input, extracting structured information, generating natural language responses. But not for final decision-making in critical systems, and not for unconstrained reasoning.

**Why this division?** Because it's about the cost of errors. If an LLM misparses a user query, the system can ask for clarification—the cost is a minor friction. But if an LLM makes an incorrect diagnostic assessment or wrongly flags a financial transaction, the consequences can be severe and irreversible. System-centric design routes high-stakes decisions through deterministic, auditable components precisely because the error tolerance is near zero.

A better way to say it: **LLMs are used for interface tasks, not authoritative decisions.**

---

## 8. What Actually Performs the "Analysis"?

In system-centric architectures, the "analysis" is distributed across multiple components: traditional ML models (classifiers, anomaly detectors), domain-specific models (computer vision, NLP extractors), rule-based systems (explicit business logic), and external tools (databases, APIs).

These components share critical properties: they are deterministic or bounded, versioned, and logged. Which makes them traceable, auditable, and safer to deploy.

---

## 9. A Concrete Example

Abstract principles are easier to grasp with a real scenario.

Consider an insurance claims system. A customer submits a claim, and the system must decide whether to approve or deny it.

**Model-centric approach:** The claim goes directly to an LLM. The model reads the claim, the policy, and outputs "Approved" or "Denied" with an explanation. This works in a demo. But when a denied customer asks "Why was I rejected?", the company cannot point to a specific rule. When a regulator audits the system, there's no decision log—just a prompt and a generated response. If you rerun the same claim with a slightly different prompt template, you might get a different decision.

**System-centric approach:** The claim is parsed (possibly by an LLM) into structured fields. Those fields are checked against policy rules by a deterministic engine. Edge cases are flagged for human review. Every step is logged with timestamps, model versions, and rule IDs. When the customer asks "Why?", the system can point to Rule 4.2.1. When the regulator audits, there's a complete decision trail.

The intelligence in both systems may be comparable. The difference is in accountability.

---

## 10. The Trade-Off Nobody Talks About

I want to be honest about something: system-centric design is not free.

It requires more upfront engineering. You need to define explicit rules, build logging infrastructure, version your models, and design the interaction between components. A model-centric prototype might take a weekend; a system-centric production system might take months.

There's also a latency cost. Routing through multiple components—validation, classification, rule checking, logging—adds processing time compared to a single LLM call.

And there's a flexibility cost. Structured systems are harder to modify when requirements change, because changes might ripple across multiple components.

So the real question isn't "which approach is better?" It's: **what are the consequences of being wrong?**

For a personal writing assistant, model-centric is perfectly fine. For a medical diagnostic aid, a financial fraud detector, or a government benefits system—where decisions affect people's lives and must withstand legal scrutiny—system-centric isn't optional. It's required.

---

## 11. Why This Matters in Regulated Domains

In domains like healthcare, finance, and government systems, you cannot simply say "the model thinks this is correct."

You must be able to answer: Why? Based on what data? Using which model version? Following which rules?

Without that, the system cannot pass compliance, cannot be audited, and cannot be trusted.

This is exactly why countries in Tier 2 of the global AI landscape are investing in sovereign AI infrastructure—not because they can't use American or Chinese models, but because their regulatory environments *demand* the kind of auditability that model-centric architectures cannot provide. The EU AI Act, for instance, explicitly requires risk assessment, transparency, and human oversight for high-risk AI applications. You can't satisfy those requirements with a black-box API call.

---

## 12. My Final Takeaway

The biggest shift for me was this:

**AI engineering is not just about building better models—it's about building better systems around models.**

And more specifically:

- Model-centric AI tries to make models smarter.
- System-centric AI tries to make systems trustworthy.

If I were to summarize my learning in one sentence:

**LLMs are powerful, but in real-world systems, they should be controlled components, not the sole decision-makers.**

This realization completely changed how I think about building AI projects—and what actually matters if I want to move from demos to production systems.
