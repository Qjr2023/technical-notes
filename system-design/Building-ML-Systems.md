# From Client Meeting to Architecture: A Framework for Building ML Systems Under Real-World Constraints

I recently built a machine learning detection system with an industry partner in healthcare. I expected the hardest part to be training a model. It wasn't. The hardest part was figuring out what to build.

Academic ML projects start with a dataset and a metric. Industry projects start with a conversation. That conversation is full of business language, compliance jargon, and assumptions that nobody writes down. Translating all of that into a system design is a skill that no coursework prepared me for.

This post shares the framework I developed: from understanding what an industry partner actually needs, to making design decisions, to navigating the tradeoffs that show up once you start building.

## Step 1: Separate hard constraints from soft preferences

In our first few meetings, the partner rarely talked about models or algorithms. They talked about workflows, compliance rules, and user experience. The word "accuracy" almost never came up.

I learned to sort everything I heard into two buckets.

**Hard constraints** come from regulations, legal obligations, or fundamental business needs. In a healthcare environment, data privacy rules don't have a "best effort" option. You comply or you don't ship. These constraints are architectural. They eliminate entire categories of solutions before you start designing.

**Soft preferences** are things the partner cares about but can negotiate on. Detection thresholds, UI behavior, the exact scope of file types to cover. These are design parameters you can revisit as the project evolves.

The distinction matters because hard constraints should be locked in early. They narrow the solution space, and that narrowing is actually a gift. Fewer choices means faster, more confident decisions downstream.

One habit I developed: after every meeting, I wrote down not just what was said, but what was implied. A single sentence like "we can't send files outside our environment" cascades into model selection, infrastructure design, and even which open-source libraries you can use. If you don't trace these implications early, they'll surprise you during implementation.

## Step 2: Document every fork, not just every decision

Once constraints are clear, design decisions follow more naturally than you'd expect. But the decisions themselves aren't the hard part. The hard part is remembering *why* you made them three weeks later.

I found it useful to document every significant choice as a "fork." At each fork, I wrote down three things:

1. What were the options?
2. What eliminated the alternatives?
3. What tradeoffs does the chosen path introduce?

This "why not" record serves two purposes. First, it protects you from second-guessing. When someone asks "why didn't you use approach X?", you have a clear answer grounded in constraints, not preferences. Second, it becomes invaluable when you need to explain your architecture to stakeholders or new team members. People want to see that you considered alternatives, not just that you picked the popular option.

Here is a concrete example. Suppose your partner's environment requires all data processing to stay on-premise. That single constraint eliminates any solution that depends on cloud-hosted APIs for analysis, even if those APIs offer superior detection rates. The fork note would read: *"Cloud-based analysis API eliminated due to data locality requirement. Local library selected for equivalent feature coverage within the compliance boundary."*

This takes five minutes per decision. Over the course of a project, it builds into a design rationale document that explains the entire architecture.

### Example fork: where does shared logic live?

One fork that seems minor but has outsized impact: when the same logic appears in multiple stages of your pipeline (feature transformations used in both training and inference), do you copy it into each script, or do you maintain a single shared module?

Copying feels faster. But training notebooks and inference scripts evolve independently. Small inconsistencies in preprocessing silently degrade real-world performance even when test metrics look fine. I've seen this firsthand.

The fork note: *"Shared module selected over duplicated logic. Adds an import dependency but guarantees training-inference consistency. Drift between the two paths is the most common silent failure mode in ML pipelines."*

## Step 3: Pick the right metric for your deployment context

Every detection system has multiple success criteria. Choosing which one to optimize for is itself a design decision, and it's one that many projects get wrong by defaulting to accuracy.

### In detection systems, false positive rate matters more than accuracy

A system that correctly classifies 99% of samples sounds great until you think about what the other 1% means in production.

A false negative (a threat slipping through) is the kind of risk the system was built to mitigate. A false positive (a legitimate file getting blocked) actively damages the system's credibility. Users whose workflows get interrupted will file complaints, lose trust, and push to have the system disabled.

The best detection system is the one that stays turned on. Tuning for low FPR, even at the cost of a few points of recall, often produces a system that users actually trust and keep enabled. The right way to pick your primary metric is to think through the business impact of each failure mode. In most detection systems, a false positive disrupts real users in real time, while a false negative represents a probabilistic risk that other layers of defense can still catch. That asymmetry should drive your optimization target.

### Latency shapes your architecture more than you expect

Our detection system sat in the file upload path. Every uploaded file had to pass through the pipeline before it could proceed. This created a hard latency budget. If analysis takes too long, the upload experience feels broken.

This constraint directly shaped model selection. Powerful but expensive techniques like deep learning inference, NLP-based content analysis, or multi-stage ensemble stacks become impractical when your budget is measured in seconds. You might get marginally better detection with a heavier pipeline, but if users stare at a spinner for 30 seconds per upload, you've failed a different requirement.

The tradeoff we made: sacrifice detection depth for detection speed. We chose lightweight models that completed inference in a fraction of a second. The more expensive analysis techniques were scoped as future enhancements. One realistic pattern is a staged approach: the fast model gates the upload in real time, while a deeper scan runs in the background and can flag or quarantine files retroactively if something suspicious surfaces. This keeps the user experience snappy without giving up on thorough detection entirely.

This isn't cutting corners. It's recognizing that a system has multiple success criteria and designing an architecture that addresses them in layers rather than trying to do everything in one synchronous pass.

### Document your known limitations

During development, I found edge cases where certain input formats produced feature distributions different from training data, leading to incorrect predictions. The instinct is to fix every edge case before shipping. The reality is that no project has unlimited time, and chasing every tail case delays the things that matter more.

What you can do is document the limitation, explain why it occurs, and propose a path forward. In our case, the issue was tied to how different versions of a file format encode their internal structure. A file created with one version produced different raw features than a file from another version, even though both were legitimate. The fix wasn't retraining. It was adjusting the decision boundary for those patterns and flagging them for review.

A well-documented limitation signals engineering maturity, not failure. Partners and reviewers respect teams that know the boundaries of their system.

## Takeaways

**Constraints accelerate decisions.** They feel limiting, but they eliminate options. Embrace them early.

**Record the forks.** The "why not" is as valuable as the "why." It shows your reasoning, prevents backtracking, and gives future developers context they'd otherwise have to guess at.

**Optimize for your deployment context.** In detection systems, FPR often matters more than accuracy. In upload pipelines, latency matters more than model complexity. Think through the business impact of each failure mode before picking your primary metric.

**Protect the must-haves. Cut the nice-to-haves.** When timelines tighten, scope reduction should come from soft preferences, never from hard constraints.

**Ship what works. Document what doesn't.** A deployed system that reliably handles 90% of cases is worth more than a prototype that theoretically handles 100%.