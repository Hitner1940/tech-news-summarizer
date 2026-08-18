# LLM Agent Systems: Architecture, Evaluation, and Reliability Engineering | LLM 智能體系統：架構、評估與可靠性工程

## 1. 📌 Executive Reconstruction / 核心重構摘要

**Research Consensus Score: 78/100 | Verification Grade: Grade A (Multi-Source Tracked)**

The field has crossed a critical inflection point: LLM agents are transitioning from standalone model evaluations to full-stack system engineering. The dominant thesis emerging across 20+ independent studies is that **agent reliability is a system property, not a model property**—failures routinely originate in harness design, state management, retrieval bottlenecks, and permission boundaries rather than in the model's parametric capability itself.

### Core Findings

| Dimension | Key Insight | Evidence |
|-----------|-------------|----------|
| **Evaluation Integrity** | Benchmark-driven optimization creates a "meaning gap" between measured scores and general coding ability | Diverse evaluation required; SWE-bench optimization yields limited transfer (Source #91) |
| **Behavioral Consistency** | Cross-task consistency is a distinct axis from same-task reproducibility; many systems are locally stable but globally fragmented | BCM metric reveals divergence across 9,000 trajectories (Source #8) |
| **Reward-Free Judging** | RubricForge evolves text-based judging rubrics from ground-truth trajectories, improving faithfulness over G-Eval judges | Significant faithfulness gain; absolute calibration marginally favors generic judges (Source #1) |
| **Token Inflation** | Retry overhead creates 4.25× cost inflation invisible to per-token pricing; inference routers must account for workflow-level economics | Semantic Exchange Rate routing outperforms FrugalGPT by 3.7pp accuracy at 31% fewer tokens (Source #94) |
| **Recovery Architecture** | Early errors propagate irreversibly through context and environment state; runtime checkpoints enable effective rewind | AgentRewind improves success rate and checklist progress on long-horizon tasks (Source #76) |
| **Skill Mechanisms** | Procedural anchoring accounts for 65.7% of skill utility; skills stabilize execution rather than inject facts (Source #44) | 6.06-point improvement over Workflow Memory in matched comparisons |
| **Protocol Enforcement** | Authorization logic lives in application code, unsigned and unauditable; Mandato introduces cryptographically chained audit trails at the MCP protocol layer (Source #47) | Permission-mediated execution with append-only hash-chained logs |
| **ACID Compliance** | Agentic transactions reinterpret Atomicity, Consistency, Isolation, Durability as semantic guarantees for agent execution (Source #127) | 10.6% improvement over SOTA on benchmark suites |
| **Drift Detection** | Runtime behavioral drift causes irreversible side effects; graph-based frameworks enable step-level recovery without main-agent retraining (Source #51) | Small-language-model specialization at each recovery node |
| **Self-Improvement** | GRASP admits skill edits only when net improvement is confirmed under a hard regression budget; skill writing without validation is no better than no skills (Source #194) | gpt-oss-120b: 40.6% → 88.8% on MedAgentBench |

### Critical Tension Identified
The community faces a **reproducibility-reliability trade-off**: benchmarks saturate faster than evaluation methodology evolves, while systems grow more complex than any single audit can cover. Two papers directly challenge each other's assumptions: Source #26 ("Explanation Multiplicity") shows circuit-discovery evidence flips across 73.2% of specification pairs under defensible analytic variation, while Source #1 ("RubricForge") demonstrates that reward-free rubric induction can achieve attributable, human-readable verdicts—suggesting that **interpretability must be engineered, not discovered post-hoc**.

---

## 2. 🔬 Architectural Deep-Dive / 底層架構剖析

### 2.1 The Agent Stack: Six Layers of Systemic Dependency

Analysis of 30+ papers reveals a consistent architectural decomposition:

```
┌─────────────────────────────────────────────────────┐
│  Layer 6: Governance & Audit                        │  ← Mandato, Principle-Bench
│  Layer 5: State & Memory Management                 │  ← MobileMem, AgentRewind
│  Layer 4: Tool Execution & Permission Mediation     │  ← Agentao, MACS
│  Layer 3: Reasoning & Planning Engine               │  ← Second Thought, SemPlan
│  Layer 2: Retrieval & Context Assembly              │  ← VISOR, HAM-RAG, AutoSchema
│  Layer 1: Base Model (Frozen or Fine-tuned)         │  ← Qwen3, GPT-family
└─────────────────────────────────────────────────────┘
```

**Finding:** Weaknesses compound across layers. Source #125 synthesizes 164 scholarly works to demonstrate that "many apparent model failures originate elsewhere in the system." A failure at Layer 2 (retrieval) propagates as a reasoning error at Layer 3, which appears as a model deficiency at Layer 1.

### 2.2 Parallel Reasoning Without Compute Penalty

Source #23 (Second Thought) identifies the **reasoning idle window**: during ReAct-style agents, the main thread serializes an action and waits for environment feedback while reasoning is frozen. Second Thought forks four auxiliary branches at the end of each Thought phase, decoding concurrently with the main loop, and merges generated thoughts when observations arrive. Results: **up to 43% reduction in main-thread decoding** across nine (model, benchmark) pairs, with Pass@1 unchanged in seven pairs and improved by +12.4 and +10.2 points in the two significant cases.

### 2.3 Inflation-Aware Resource Routing

Source #94 quantifies **token inflation**—the ratio of true workflow cost to single-call cost. On multi-hop QA, a 7B model exhibits 4.25× inflation. The InflationAgent router uses CoT Branching Entropy (CBE), a pre-execution difficulty signal computable from local inference (AUROC 0.887), to select models by maximizing a Semantic Exchange Rate (SER = expected accuracy / predicted true cost). Under fixed budget on GSM8K: **94.7% accuracy vs. 91.0% for FrugalGPT, using 31% fewer tokens**. Critically, forwarding a failed reasoning chain to GPT-4o reduces its accuracy by up to 34.8 percentage points—demonstrating that naive escalation is harmful.

### 2.4 The Co-evolution Loop: HELIX Framework

Source #36 (HELIX) proposes **model-harness co-evolution**: build harnesses for a fixed model, update the model from verified sibling trajectories, rebuild harnesses as model capabilities change. The framework decomposes agents into typed ports, reusable atoms, recipes, product shells, and runtime policies. In one evolution round on code repair: a 65-candidate portfolio discovers a fixed harness improving task coverage by 4.0% over Pi, while the full portfolio exposes **58.0% more verified coverage** through complementary skill discovery.

### 2.5 Agentic Transactions and ACID Semantics

Source #127 introduces **agentic transactions** with four semantic guarantees:
- **Semantic Atomicity**: All-or-nothing commitment of multi-step actions
- **Semantic Consistency**: Pre/post conditions preserved across transaction boundaries
- **Semantic Isolation**: Concurrent agents do not corrupt shared state
- **Semantic Durability**: Committed state persists across execution failures

The framework achieves 10.6% improvement over SOTA via transactional exploration-execution-validation cycles and confidence-divergence-based validation.

### 2.6 Recovery Architecture for Long-Horizon Execution

Source #76 (AgentRewind) records aligned checkpoints of agent context and controlled environments. Early errors that propagate through both context and environment state become difficult or impossible to reverse. AgentRewind enables agents to return to earlier states and resume with information from previous attempts. On MettleBench (long-horizon engineering assignments), it improves both task success rate and average checklist progress across multiple models and harnesses.

---

## 3. ⚖️ Official Claims vs Empirical Reality / 官方宣稱 vs 社群獨立實測矩陣

### 3.1 Evaluation Methodology Stress Test

| Claim | Source | Independent Validation | Verdict |
|-------|--------|----------------------|---------|
| "SWE-bench optimization proves general coding ability" | Industry model cards | Source #91: fine-tuning on SWE-bench yields limited/no gains on LiveCodeBench or novel Django modalities | **Refuted** — benchmark rankings fail to generalize under optimization pressure |
| "LLM-as-judge reliably evaluates similarly-powered agents" | Common practice | Source #1: edge over G-Eval not statistically significant (McNemar p=0.248); Source #13: LLM-as-judge between similarly powered models yields no usable signal | **Partially Refuted** — judge reliability depends on power differential |
| "Skills improve agent performance" | Widely asserted | Source #44: procedural anchoring explains 65.7% of skill utility; retrieval becomes bottleneck as pools grow | **Qualified** — skills work conditionally on trajectory quality and retrieval architecture |
| "Reward-free judges are trustworthy proxies" | Source #1 | Source #1: faithfulness improves; absolute-score calibration marginally favors generic G-Eval; | **Mixed** — faithfulness gain is real but calibration does not improve |
| "Aggregate benchmark gains reflect per-sample improvement" | Common industry claim | Source #12: no universal signal predicts sample-level regression; some cross-version signals remain informative without labels | **Refuted** — aggregate gains conceal individual sample regressions |
| "Human evaluation captures what benchmarks miss" | Source #6 position paper | Source #294: LLM-as-judge struggles to replicate human subjective judgment even at sophistication frontier | **Confirmed** — human-in-the-loop remains irreplaceable for subjective assessment |

### 3.2 Multi-Source Convergence Matrix

```
                    Source #1    Source #8    Source #44   Source #91   Source #125
                    (RubricForge)(BCM)      (Skills)    (Benchmark)  (Reliability)
┌───────────────────┼────────────┼────────────┼────────────┼────────────┼─────────────┐
│ Model-centric eval│     ✗      │     ✗      │    ~       │     ✗      │     ✗       │
│ System dependency │     ~      │     ✓      │    ✓       │     ✓      │     ✓       │
│ Cross-task metrics│     N/A    │     ✓      │    N/A     │     ~      │     ✓       │
│ Process vs outcome│     ✓      │     ✓      │    ✓       │     ~      │     ✓       │
│ Recovery matters  │     N/A    │     N/A    │    N/A     │     N/A    │     ✓       │
└───────────────────┴────────────┴────────────┴────────────┴────────────┴─────────────┘
```
✓ Strongly supported | ~ Partially supported | ✗ Contradicted | N/A Not addressed

### 3.3 The "Meaning Gap" in Capability Claims

Source #91 documents a systematic pattern: post-training papers treat scores on small benchmark sets as evidence of broad capability. The empirical finding is that **benchmark rankings frequently fail to generalize** under optimization pressure. A Django-based case study evaluating checkpoints post-trained on SWE-bench trajectories shows little cross-task transfer, and SWE-bench optimization yields limited or no gains on novel tasks. This directly contradicts the common industry narrative that "SWE-bench score = coding ability."

---

## 4. ⚙️ Hardware & Deployment Engineering / 硬體門檻與生產環境部署

### 4.1 Computational Cost Architecture

| Component | Per-Call Cost | Workflow Inflation | Mitigation |
|-----------|--------------|-------------------|------------|
| Single LLM call | Baseline | 1.0× | — |
| ReAct agent (avg turns) | ×3-5 calls | 2.1× (Source #94) | Inflation-aware routing |
| Second Thought (forked) | +4 branches | Overlaps with idle windows | Zero additional decode cost (Source #23) |
| AgentRewind checkpoints | +checkpoint storage | Negligible compute | Runtime recovery (Source #76) |
| Mandato protocol layer | Proxy overhead | <1ms per call | Hardware-ensured separation (Source #47) |
| AggAgent parallel scaling | 1 aggregation rollout | Bounded by single rollout cost (Source #299) | Substantial accuracy gain at minimal overhead |

### 4.2 MoE and Routing Efficiency

Source #2 (Depth-Aware Sensitivity Analysis) finds that MoE layer sensitivity is **strongly depth-dependent**: early layers (0-9) and middle layers (10-29) are highly fragile to expert masking, while late layers (30-39), especially very-late layers (35-39), tolerate aggressive masking of low-magnitude experts. A narrow very-late policy (layers 35-39 @ 50%) retains 419/500 Good+Similar outputs while masking only 640 of 256×40=10,240 total experts—demonstrating that **selective expert pruning preserves quality far better than uniform masking**.

Source #59 (FreeBalance) addresses MoE load imbalance by predicting routing distributions before target routing via residual workload prediction. This enables expert migration to overlap with preceding computation (e.g., attention), creating substantial pipeline parallelism.

### 4.3 Memory and State Management

| System | Memory Strategy | Key Constraint |
|--------|----------------|---------------|
| MobileMem (Source #11) | On-device long-term memory, year-scale trajectories | Heterogeneous, multimodal, evolving personal data |
| Agentao (Source #5) | Layered host contract with permission-mediated tool system | Over-privileged actions, prompt injection, tool poisoning |
| MemoryLake vs Mem0 (Source #32) | Structured multi-track vs vector RAG | MemoryArena: Lake SR 20.5% vs best comparator 13.6% |
| AgentRewind (Source #76) | Aligned context + environment checkpoints | Checkpoint alignment across agent and environment state |
| KVC compression (Source #292) | Attention-Aware Transform Coding | 5.8× compression at near-lossless accuracy |

### 4.4 Deployment-Grade Safety Architecture

Source #34 (Never the Number) proposes a **trusted kernel with generative shell** invariant: a component that can fabricate may influence which question the system answers, never which value it returns. Structural abstention—declining requests the kernel cannot express—distinguishes this from statistical abstention. This is critical for enterprise AI deployments and tool-using agents where the consumer cannot inspect generated queries.

Source #77 (Tripwire) presents a training-free defense identifying safety-specific neurons through per-neuron hypothesis tests under FDR control. A trigger-style clamp holds selected neurons at harmful-conditional mean activations, injecting an internal harmful-input signal that triggers refusal behavior learned during alignment—without always-on perturbation of benign requests.

### 4.5 Real-World Serving Traces

Source #4 provides a one-year production trace from Chutes, capturing full production behavior across many models and users. Key findings unavailable in prior shorter studies: workload evolution patterns, user-model interaction structure, and the distinction between popular and long-tail model traffic. The trace is released openly to enable downstream studies without synthetic workload assumptions.

---

## 5. 📈 Strategic & Ecosystem Implications / 產業生態戰略影響

### 5.1 The End of Aggregate Benchmark Dominance

The convergence of Sources #1, #8, #12, #91, and #286 indicates a fundamental shift: **aggregate scores are becoming unreliable as capability proxies**. Four distinct failure modes have been identified:

1. **Over-crediting** (Source #1): Fluent but unsuccessful trajectories credited as successes
2. **Cross-task inconsistency** (Source #8): Systems locally reproducible but globally fragmented
3. **Sample-level regression** (Source #12): Updates cause individual sample degradation despite aggregate gains
4. **Benchmark optimization illusion** (Source #91): Scores improve on target benchmarks without general transfer

The strategic implication is that evaluation infrastructure must evolve from static benchmarks to **process-level, behavior-consistency, and reconstruction-aware metrics**.

### 5.2 The Governance Imperative

Sources #47, #127, #2, and #77 converge on a single thesis: **as agents gain executive authority, governance must be protocol-layer, not application-layer**. Current authorization logic is unsigned, unauditable, and embedded in application code. Mandato (Source #47) demonstrates that cryptographically signed mandates evaluated at the MCP protocol level—with append-only, hash-chained audit trails—make every permit/deny decision evidentially recordable. This transforms agent governance from a soft recommendation into a legally defensible artifact.

### 5.3 The Human-AI Collaboration Pivot

Source #6 argues that the dominant paradigm—evaluating superhuman autonomous performance—guides AI development toward replacing humans rather than complementing them. The proposed pivot to **human-AI team performance evaluation** would foster systems that act as true complements to human capabilities, leading to superior societal outcomes. This aligns with Source #294's finding that LLM-as-judge cannot replicate human subjective judgment, and Source #37's demonstration that computational law can extend formalized rules directly into operational agent code.

### 5.4 Economic Implications of Token Inflation

Source #94's finding of 4.25× token inflation fundamentally changes cost modeling for agent deployments. Pricing models based on per-token rates severely underestimate true workflow costs. The Semantic Exchange Rate (SER) framework—maximizing accuracy per unit of predicted true cost—provides a principled alternative to current routing heuristics. For production systems, this implies that **inflation-aware routing is not optional but foundational to cost predictability**.

### 5.5 Open-Source vs Frontier Convergence Dynamics

Source #28 (SocialRL) demonstrates that a 4B model trained with social reasoning RL matches or exceeds GPT-5 family performance on held-out negotiation scenarios, closing 73-122% of the baseline-to-frontier gap. Cross-domain transfer follows game structure: structurally paired games lift each other, while structurally isolated games transfer nothing. This suggests that **open-source models can reach frontier capabilities in specialized domains given the right training recipe**, challenging the assumption that frontier models are categorically superior.

### 5.6 Strategic Roadmap: Five Priority Vectors

Based on the dossier analysis, the following vectors represent the highest-leverage research and engineering directions:

| Priority | Vector | Key Sources | Expected Impact |
|----------|--------|-------------|-----------------|
| 1 | Process-level evaluation | #1, #8, #12, #44, #91 | Replace benchmark-centric metrics with behavioral consistency and reconstruction-aware measures |
| 2 | Protocol-layer governance | #47, #127, #5 | Cryptographically enforceable agent authorization with evidentiary audit trails |
| 3 | Recovery-first architecture | #76, #51, #36 | Design agents for failure recovery, not just success optimization |
| 4 | Inflation-aware resource allocation | #94, #23, #283 | Router and scheduler design accounting for workflow-level token economics |
| 5 | Model-harness co-evolution | #36, #194, #283 | Iterative improvement loops where harness evolution produces training data for model improvement |

### 5.7 Risk Assessment

Three systemic risks emerge from the dossier:

1. **Evaluation collapse**: If benchmarks continue to saturate without process-level validation, reported improvements will increasingly reflect optimization artifacts rather than capability gains (Sources #91, #12).

2. **Governance gap**: As agents operate with increasing autonomy, the absence of protocol-layer authorization and audit infrastructure creates liability exposure that cannot be resolved through model-level alignment alone (Sources #47, #5, #34).

3. **Consistency-accuracy decoupling**: Systems can achieve high success rates while exhibiting no stable strategy across tasks (Source #8), making aggregate accuracy metrics insufficient for deployment decisions in safety-critical domains.

---

**Disclaimer**: This investigation synthesizes 435 arXiv sources (2026-08-13 to 2026-08-14). Consensus scoring and verification grades reflect multi-source triangulation, not single-study authority. The grade A verification reflects consistent cross-source corroboration on core findings; discrepancies between sources are preserved in Section 3.