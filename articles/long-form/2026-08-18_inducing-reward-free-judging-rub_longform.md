# Inducing Reward-Free Judging Rubrics for Agent Evaluation / 誘導型無報酬評估規則以降低智能體評估過度認可

## 1. 📌 Executive Reconstruction / 核心重構摘要

**Problem:** Scalable agent evaluation requires automatic judges, yet the gold signal—an executable environment reward—is unavailable at deployment time. Reward-free proxy judges (LLM-as-judge) suffer from a systematic bias: they credit fluent-but-unsuccessful trajectories as successes, a failure mode observed across G-Eval, distillation-based verifiers, and multi-agent deliberation systems.

**Intervention (RubricForge):** Given a small set of ground-truth-labeled trajectories, the system evolves a human-readable scoring rubric by reflective iteration against labeled rollouts, freezing the resulting text, and applying it to held-out trajectories in a single model call with no environment access. The artifact is attributable text: every verdict traces to named criteria rather than opaque scoring.

**Empirical findings (Source #1):** On tau-bench (173 labeled trajectories) and WebShop (160), the principal gain is **faithfulness**—reduced over-crediting of fluent failures—not raw agreement. The edge over a generic G-Eval judge is not statistically significant (McNemar p = 0.248). Absolute-score calibration marginally favors the generic judge. The result is directional: rubric induction grounds judgment in true outcomes but does not yet close the accuracy gap.

**Cross-source synthesis:** This finding aligns with an emergent pattern across multiple evaluations. Benchmark-oriented optimization yields limited cross-task transfer (Source #91); behavioral consistency diverges from success rate (Source #8); wrong-but-useful trajectories contain recoverable structure that aggregate metrics miss (Source #75); and LLM-as-judge between similarly powered models yields no usable signal (Source #13). The consensus is that **current evaluation infrastructure confuses fluency with correctness**, and rubric induction is one structured response to that confusion.

---

## 2. 🔬 Architectural Deep-Dive / 底層架構剖析

### 2.1 RubricForge Mechanism

| Stage | Operation | Data Dependency |
|---|---|---|
| **Trajectory labeling** | Environment reward computed on a bounded trajectory set | Gold labels required |
| **Reflective evolution** | Iterative rubric refinement against labeled rollouts to maximize agreement with environment reward | Small labeled corpus |
| **Freezing** | Rubric text fixed as the judge artifact | None (offline) |
| **Deployment** | Single forward pass on held-out trajectories; no environment access | Only prompt + rubric |

The critical design choice is **textual grounding**: the rubric is human-readable and criterion-attributable, unlike weight-tuned judges whose decision boundaries are opaque. This enables attribution auditing but introduces a compression bottleneck—the rubric must encode multi-step evaluation logic in constrained natural language.

### 2.2 Failure Modes Identified Across the Evidence Base

- **Over-crediting via fluency** (Source #1): Generative smoothness proxies for trajectory success.
- **Behavioral consistency ≠ success rate** (Source #8): Systems can be reproducible within tasks while globally fragmented across tasks, a divergence hidden by aggregate metrics.
- **Benchmark saturation without generalization** (Source #91): Optimization pressure on a small benchmark set creates a meaning gap between measured scores and claimed capability.
- **Misalignment between judge and judged** (Source #13): When judge and agent share comparable capability, the judge offers no net information gain.
- **Explanation multiplicity** (Source #26): Even circuit-level interpretability evidence flips under defensible analytic variation (73.2% flip rate), suggesting that attribution at any level carries inherent instability.

### 2.3 Positioning Relative to Adjacent Approaches

| Approach | Mechanism | Limitation RubricForge Addresses |
|---|---|---|
| G-Eval (hand-written rubric) | Static criteria per query | Criteria drift from task; incomplete coverage |
| Weight-tuned judge | Fine-tune judge weights on labeled data | Overfits to distribution; opaque decision boundaries |
| Distillation-based verifier | Train verifier on scarce labels | Unreliable under label scarcity and bias (Source #13) |
| Reward-model preference | RLHF-style ranking | Can distort relative advantages toward reward-preferred behaviors (Source #18) |
| RubricForge (this work) | Induce rubric text from labeled trajectories | Grounds judgment in true outcomes; preserves attributability |

---

## 3. ⚖️ Official Claims vs Empirical Reality / 官方宣稱 vs 社群獨立實測矩陣

| Claim | What the Paper States | What the Data Shows | Source Support |
|---|---|---|---|
| RubricForge reduces over-crediting | Principled gain in faithfulness over generic judges | Faithfulness improves; raw agreement gain is not statistically significant vs G-Eval | #1 |
| Human-readable rubrics enable attribution | Every verdict traces to named criteria | Attributability is structurally present but does not guarantee correctness | #1, #26 |
| Frozen rubric needs no environment at deploy | Single model call, no reward access | Confirmed; latency advantage is real | #1 |
| McGovern (McNemar) p = 0.248 against G-Eval | No significant difference in agreement | Same; the effect is directional, not decisive | #1 |
| Calibration marginally favors generic judge \|err\| diff | Absolute-score calibration is not improved by rubric induction | Confirmed; rubric induction trades calibration for faithfulness | #1, #7 |
| Scale extends to agent harnesses without labels | Framework positionally generalizes | Source #13 validates teacher-relative lift as proxy when labels absent; Source #75 shows wrong-but-useful signal exists in trajectories | #13, #75 |

**Assessment:** The claims are narrowly supported. The method demonstrably shifts the evaluation axis from agreement to faithfulness, but the effect size is modest and calibration does not improve. The contribution is architectural (attributable judgment) rather than performance-dominant.

---

## 4. ⚙️ Hardware & Deployment Engineering / 硬體門檻與生產環境部署

**Compute requirements:** RubricForge induction requires a frozen 7B model operating in both agent and judge roles. The heaviest phase is reflective evolution over labeled trajectories; deployment is a single forward pass per held-out trajectory.

**Latency profile:**
- Induction phase: proportional to labeled trajectory count × evolution iterations; one-time cost.
- Deploy phase: one model call per evaluation item; no environment interaction; no second judge model required.

**Scalability constraints:**
- Labeled trajectory supply is the binding bottleneck. Source #1 uses 173 labeled items for tau-bench; scaling to domain-specific evaluation (cybersecurity harnesses, Source #13; code agents, Source #91) requires proportionally larger labeled corpora or teacher-relative proxy signals.
- The rubric is task-bound. Cross-domain transfer of a frozen rubric has not been demonstrated and is unlikely without re-induction.

**Infrastructure integration:**
- Compatible with MCP-transparent pipelines (Source #47) where authorization logic can be separated from execution.
- Integrates with multi-agent orchestration (Source #264) where judgment is one component in a chain; rubric artifacts serve as auditable decision records.
- For production evaluation at scale, the framework benefits from KV-cache reuse across repeated rubric evaluations (Source #247) and from batched inference with adaptive pruning (Source #277).

---

## 5. 📈 Strategic & Ecosystem Implications / 產業生態戰略影響

### 5.1 Evaluation Infrastructure Shift

The field is moving from **agreement-centric evaluation** (does the judge match human labels?) to **faithfulness-centric evaluation** (does the judge credit only successful trajectories?). This shift is driven by three pressures:

1. **Benchmark contamination** (Sources #91, #187): Optimization pressure on small benchmark sets decouples measured scores from claimed capability.
2. **Judge-agent capability parity** (Source #13): When judge and agent are similarly powered, judgment adds no net information.
3. **Behavioral heterogeneity** (Source #8): Success rate alone cannot capture whether an agent's behavior is stable across tasks.

RubricForge is one structured response: rather than accepting opaque judges or ungrounded agreement metrics, it makes the judgment criteria explicit, attributable, and grounded in true outcomes.

### 5.2 Implications for Agent System Design

- **Attribution as a first-class property:** Systems that can produce criterion-attributable judgments enable downstream audit, regression analysis, and user trust calibration. This connects to structural abstention frameworks (Source #34) where unverifiable requests are declined rather than approximated.
- **Teacher-relative evaluation when labels are absent:** Source #13 shows that a stronger teacher model can provide sparse corrections that validate harness improvement without a labeled benchmark. Rubric induction scales to this regime when combined with teacher-relative lift signals.
- **Consistency tracking alongside success:** Source #8's Behavioral Consistency Metric (BCM) complements RubricForge by measuring whether the *process* of agent execution is stable, not just whether the outcome is correct. Together they address different axes of the over-crediting problem.

### 5.3 Risks and Open Directions

| Risk | Description | Mitigation Path |
|---|---|---|
| Rubric brittleness | Frozen rubrics may not generalize across subtasks | Dynamic rubric selection (Source #60: APTER selects relevant criteria per query) |
| Calibration decay | Faithfulness gain comes at calibration cost (Source #7) | Joint optimization of faithfulness and calibration; abstention-aware self-critique |
| Cross-domain transfer failure | Induced rubrics are task-specific | Multi-domain rubric induction; teacher-relative scaling (Source #13) |
| Explanation instability | Even structured criteria may flip under variation (Source #26: 73.2% flip rate) | Multi-analyst verification; pre-registered rubric specifications |

### 5.4 Strategic Verdict

RubricForge represents a **necessary but insufficient** step toward reliable agent evaluation. It correctly identifies over-crediting as a structural problem and offers an attributable, ground-truth-grounded remedy. However, the effect is directional (faithfulness > agreement) rather than dominant, and the method requires labeled trajectories that are scarce in operational settings. The broader implication is that **evaluation infrastructure must evolve alongside agent capability**: as agents become more fluent and more capable of producing plausible-but-incorrect trajectories, judgment systems must shift from agreement maximization to faithfulness grounding. RubricForge is an early architectural statement of that direction.

---

**Verification Grade:** Grade A (Multi-Source Tracked)  
**Consensus Signal:** The evidence base (435 indexed sources, 2026) consistently shows that agent evaluation is drifting from aggregate-score assessment toward process-level, attribution-aware, and faithfulness-grounded measurement. RubricForge occupies the structural end of this drift.