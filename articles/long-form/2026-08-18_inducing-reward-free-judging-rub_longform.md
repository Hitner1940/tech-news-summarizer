# Inducing Reward-Free Judging Rubrics that Reduce Over-Crediting in Agent Evaluation | 誘導型無報酬判定規程によるエージェント評価の過表彰削減

## 1. 📌 Executive Reconstruction / 核心重構摘要

### 1.1 Problem Core / 問題核心

Scale evaluation of language-model agents increasingly depends on a **second model as automatic judge** (LLM-as-judge), because the gold signal—executable-environment reward—is expensive, slow, or unavailable at deployment time. Such a judge is a **reward-free proxy**, whose trustworthiness has hitherto depended on one of two brittle approaches:

| Approach | Mechanism | Failure Mode |
|----------|-----------|--------------|
| Hand-written rubrics (G-Eval) | Static scoring criteria | Credits fluent-but-failed trajectories |
| Fine-tuned judges | Weight-space adaptation | Fragile to distribution shift; no interpretability |

The central failure is **over-crediting**: rewarding surface-level fluency rather than outcome-verified success. As Source #166 reports, frontier judges lose up to 47 accuracy points under adversarial keyword-stuffing ("compliance theatre"), and Source #8 documents that over 60% of agent failure modes reflect process-level instability invisible to endpoint-only metrics.

### 1.2 RubricForge Contribution / 貢献

RubricForge (arXiv:2608.13564) inverts the rubric-induction direction:

1. **Start from ground-truth-labeled trajectories** (outcomes already verified against environment reward)
2. **Evolving reflective induction** against those labels to maximize agreement with true outcomes
3. **Freeze the artifact** → one-call judgment with no environment access
4. **Artifact is human-readable text** → every verdict is attributable to named criteria

**Empirical anchor:** On tau-bench (173 labeled trajectories from 220 rollouts) and WebShop (160), using a single frozen 7B as both agent and judge:

- The principal gain is **faithfulness**, not raw agreement
- McNemar p = 0.248 vs. G-Eval → edge **not statistically significant**
- Absolute-score calibration **marginally favors generic G-Eval** (|err| diff < threshold)

This is a **qualified result**: the method structurally improves interpretability and attribution without delivering clear accuracy gains on these benchmarks.

### 1.3 Why This Matters / 意義

The agent-evaluation pipeline is experiencing what Source #26 terms the **explanation multiplicity crisis**: discovered circuits/flaws flip across 73.2% of specification pairs even under defensible analytic variation. A rubric-induction framework that anchors judgments in named, traceable criteria rather than opaque weight-space preferences is arguably more valuable as a **governance artifact** than as an accuracy booster. Source #166 similarly argues that any deployment-grade LLM-judge must report per-prediction counterfactual attributions; RubricForge's human-readable output satisfies this constraint by construction.

---

## 2. 🔬 Architectural Deep-Dive / 底層架構剖析

### 2.1 RubricForge Architecture / アーキテクチャ

```
┌─────────────────────────────────────────────────┐
│                 Training Phase                  │
├─────────────────────────────────────────────────┤
│  Labeled Trajectories (outcome-verified)        │
│         ↓                                       │
│  Reflective Evolution Loop                      │
│   • Generate candidate rubric                   │
│   • Score trajectories under rubric             │
│   • Compare to environment-reward labels        │
│   • Select rubric maximizing agreement          │
│         ↓                                       │
│  Frozen Rubric Artifact (text)                  │
└─────────────────────────────────────────────────┘
              ↓↓↓ inference ↓↓↓
┌─────────────────────────────────────────────────┐
│              Deployment Phase                   │
├─────────────────────────────────────────────────┤
│  Unlabeled Trajectory                           │
│         ↓                                       │
│  One-shot Judge Call (7B model)                 │
│   • Feed trajectory + frozen rubric             │
│   • Output: verdict + named-criteria            │
│     justification                               │
└─────────────────────────────────────────────────┘
```

### 2.2 Key Design Tensions / 設計上の緊張関係

| Dimension | Tension | RubricForge Position |
|-----------|---------|---------------------|
| **Grounding** | Reward-free judge vs. reward-dependent training | Trains on verified outcomes, deploys reward-free |
| **Attribution** | Statistical agreement vs. named-criteria traceability | Names explicit scoring dimensions |
| **Calibration** | Faithfulness vs. absolute-score accuracy | Optimizes faithfulness; marginal calibration trade-off |
| **Transfer** | Benchmark-specific induction vs. general rubric | Single frozen rubric applied across tasks |

### 2.3 Comparison with Related Frameworks / 関連手法比較

| Framework | Rubric Induction | Attribution | Calibration | Out-of-Domain |
|-----------|-----------------|-------------|-------------|---------------|
| G-Eval | Manual | Named criteria | Poor | N/A |
| Fine-tuned Judge | Weight update | Opaque | Variable | Fragile |
| **RubricForge** | **Reflective evolution** | **Named criteria** | **Marginal** | **Unknown** |
| ASCERTAIN (Source #123) | Specification-driven | Test-case traceable | Audit-linked | Documented |
| Principle-Bench (Source #166) | Pre-registered rubric | Per-exemplar counterfactual | Calibrated cascade | Adversarial-tested |

Source #123 (ASSERT) demonstrates that **reported evaluation rates shift substantially** with dialogue setup, simulated user, judge model, and evidence bar—reinforcing that RubricForge's claim to "faithfulness" must be understood as **process traceability**, not absolute correctness.

### 2.4 Connection to Broader Evaluation Science / 評価科学との接点

Three convergent findings from the dossier contextualize RubricForge:

1. **BCM (Source #8):** Cross-task behavioral consistency is a distinct axis from success rate. RubricForge measures **verdict faithfulness**; BCM measures **trajectory consistency**. Together they form a two-dimensional evaluation: *what* was decided and *how consistently* the decision process behaved.

2. **RepBench (Source #249):** Capability representations extracted from benchmarks show interior clustering optima but low agreement with human taxonomy. RubricForge's named criteria partially address this gap by providing **human-interpretable dimensionality** rather than latent-space abstraction.

3. **TANGLE (Source #33):** Evaluating agents under genuinely unresolvable memory conflicts reveals that models recognize conflicts more reliably than they calibrate confidence. RubricForge's named criteria provide a **structured format** for expressing uncertainty at the criterion level, though calibration remains an open challenge.

---

## 3. ⚖️ Official Claims vs Empirical Reality / 公式宣称 vs 社群獨立實測矩陣

### 3.1 Claim Verification Matrix / 主張検証行列

| Claim | Source Evidence | Independent Signal | Status |
|-------|----------------|-------------------|--------|
| Reduces over-crediting | Abstract states "tend to credit fluent but unsuccessful trajectories" | Source #158: Legal RAG hallucinations range from <10% to ~50%; false-premise questions produce high hallucination rates | **Partially Verified** — structural mechanism present but magnitude unquantified |
| Faithfulness > raw agreement | "principal gain is faithfulness rather than raw agreement" | Source #166: No single judge dominates all four axes (accuracy, paraphrase robustness, adversarial robustness, calibration) | **Consistent** — aligns with multi-axis evaluation consensus |
| Not statistically significant edge over G-Eval | McNemar p = 0.248 | Source #26: 73.2% flip rate across defensible analytic variations suggests evaluation instability is common | **Confirmed** — result is robustly null on accuracy |
| Absolute-score calibration marginally favors generic judge | "|err| diff..." (truncated) | Source #7: Stable miscalibration in LLMs — confident errors can be locally stable, not merely fragile | **Plausible** — calibration remains an open problem per Source #7 |
| Human-readable, attributable verdicts | "optimized artifact is human-readable text" | Source #166: Ceca requires exact per-exemplar counterfactual attributions | **Structurally satisfied** — attribution format enabled; quality untested |

### 3.2 The Over-Crediting Mechanism / 過表彰メカニズム

Source #158 provides an independent characterization of the over-crediting phenomenon in legal RAG:

> "Hallucinations remain pervasive, ranging from less than 10% of responses for the best-performing systems to nearly half in the worst case... false-premise questions produce high hallucination rates."

This confirms that **verbal fluency can masquerade as correctness** across domains. RubricForge's named-criteria approach directly attacks this by requiring each verdict to reference explicit dimensions rather than holistic fluency judgments.

### 3.3 The Calibration Wall / 較正の壁

Source #7 identifies **stable miscalibration**: "some high-confidence errors may be stable and miscalibrated rather than simply fragile." This finding implies that even a perfectly induced rubric cannot guarantee calibration—because the judge model may **systematically over-commit** to certain reasoning patterns regardless of rubric specification. The calibration gap is therefore **architectural**, not merely procedural.

### 3.4 Cross-Domain Validity Gaps / 横断的妥当性ギャップ

| Domain | Over-Crediting Risk | RubricForge Applicability |
|--------|--------------------|--------------------------|
| Code generation (Source #91) | Benchmark optimization creates meaning gap between scores and general ability | **High risk** — fluency ≈ correctness in code is a known trap |
| Legal RAG (Source #158) | False-premise questions amplify hallucination | **Moderate** — structured criteria may reject false premises |
| Agent evaluation (Source #8) | Success rate ≠ behavioral consistency | **Directly applicable** — rubric targets process, not just outcome |
| Medical compliance (Source #15) | Binary judgments insufficient; graded scores needed | **Low fit** — domain requires probabilistic reasoning, not rubric pass/fail |

---

## 4. ⚙️ Hardware & Deployment Engineering / 硬體門檻與生產環境部署

### 4.1 Compute Profile / 計算構成

RubricForge's deployment profile is intentionally lightweight:

| Phase | Compute Cost | Hardware Requirement | Latency |
|-------|-------------|---------------------|---------|
| Rubric induction | Reflective evolution loop (unspecified iterations) | GPU (single 7B training) | Hours (one-time) |
| Judgment (deployment) | One forward pass | Single 7B model | ~100-300ms (estimated) |
| Total per trajectory | O(1) model calls | No environment access | Minimal |

This stands in contrast to:
- **Source #13** (teacher-relative harness evaluation): Requires a stronger teacher model for sparse corrections
- **Source #76** (AgentRewind): Requires checkpoint recording and environment state capture
- **Source #127** (Agentic ACID): Requires transactional exploration-execution-validation cycles

RubricForge's **zero-environment-deployment** advantage is its primary engineering contribution.

### 4.2 Integration Patterns / 統合パターン

```
┌────────────────────────────────────────────────────────────┐
│            RubricForge Integration Options                 │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Option A: Standalone Judge                                │
│  ┌─────────┐    ┌──────────┐    ┌──────────┐              │
│  │  Agent   │───→│RubricForge│───→│ Verdict  │              │
│  │ Trajectory│   │  (7B)    │    │ + Criteria│              │
│  └─────────┘    └──────────┘    └──────────┘              │
│                                                            │
│  Option B: Hybrid (Judge + Environment)                    │
│  ┌─────────┐    ┌──────────┐    ┌──────────┐              │
│  │  Agent   │───→│RubricForge│───→│Env Reward│              │
│  │ Trajectory│   │  (7B)    │    │ (fallback)│              │
│  └─────────┘    └──────────┘    └──────────┘              │
│            ↑          ↕ verification loop                  │
│         Conflict resolution                             │
│                                                            │
│  Option C: Cascade (with ASSERT/SPEC)                      │
│  ┌─────────┐    ┌──────────┐    ┌──────────┐              │
│  │  Agent   │───→│RubricForge│───→│ASSERT    │              │
│  │ Trajectory│   │  (7B)    │    │ Audit    │              │
│  └─────────┘    └──────────┘    └──────────┘              │
│            ↑          ↕ trace linkage                     │
│         Spec-driven measurement                          │
└────────────────────────────────────────────────────────────┘
```

### 4.3 Failure Modes in Production / 本番 failure 事例

Based on cross-source analysis, three production failure modes are anticipated:

1. **Rubric Drift**: As Source #26 demonstrates (73.2% flip rate under defensible variation), even well-specified rubrics are unstable under analytic variation. In production, rubric drift would compound.

2. **Calibration Collapse**: Source #7 shows that confident errors can be **locally stable**. A frozen rubric cannot self-correct stable miscalibration without re-induction.

3. **Criterion Myopia**: Source #14 (SemPlan) finds that answer correctness in enterprise data queries remains low (22-25%) even with structured planning. Rubric-induced criteria may optimize for named dimensions while missing **unforeseen failure modes**.

### 4.4 Recommended Hardening / 推奨硬化策

| Vulnerability | Mitigation | Source Support |
|--------------|------------|----------------|
| Rubric instability | Periodic re-induction from fresh labeled trajectories | Source #26 (modification instability) |
| Calibration failure | Deploy alongside abstention mechanism (Source #34 structural abstention) | Source #7 (stable miscalibration) |
| Criterion myopia | Layer with behavioral consistency metric (BCM) | Source #8 (consistency ≠ success) |
| Evaluation fragility | Tie to specification-driven measurement (ASSERT) | Source #123 (audit traceability) |

---

## 5. 📈 Strategic & Ecosystem Implications / 産業生態戦略影響

### 5.1 The Evaluation Stack Reconfiguration / 評価スタック再構成

RubricForge occupies a **structural niche** in the emerging evaluation stack:

```
┌─────────────────────────────────────────────┐
│  Strategic Layer: Principle-Based Regulation│  ← Source #166 (Principle-Bench)
│  (four-axis judge evaluation)               │
├─────────────────────────────────────────────┤
│  Specification Layer: ASSERT/ASCERTAIN      │  ← Source #123
│  (traceable measurement pipelines)          │
├─────────────────────────────────────────────┤
│  Rubric Layer: RubricForge                  │  ← THIS PAPER
│  (induced, attributable criteria)           │
├─────────────────────────────────────────────┤
│  Consistency Layer: BCM                     │  ← Source #8
│  (cross-task behavioral stability)          │
├─────────────────────────────────────────────┤
│  Process Layer: AgentRewind/TANGLE          │  ← Sources #76, #33
│  (recovery, conflict navigation)            │
└─────────────────────────────────────────────┘
```

The strategic value of RubricForge is **interoperability**: its human-readable artifact can feed upward into specification layers (ASSERT) and downward into consistency layers (BCM), creating a **traceable evaluation chain** rather than an isolated accuracy metric.

### 5.2 Industry Implications / 産業への影響

| Stakeholder | Impact | Recommendation |
|-------------|--------|---------------|
| **Model Developers** | Need eval artifacts that survive adversarial stress (Source #166) | Adopt RubricForge as part of model card reporting |
| **Regulators** | Principle-based regulation requires auditable judgment traces (Source #166, #37) | Mandate rubric-attribution as deployment prerequisite |
| **Enterprise Deployers** | Need to distinguish over-crediting from genuine capability (Source #158) | Layer RubricForge with structural abstention (Source #34) |
| **Benchmark Researchers** | Current benchmarks saturate; need renewable evaluation (Source #187) | Use RubricForge to create ongoing rubric evolution |

### 5.3 Research Trajectory Implications / 研究軌道への影響

Three trajectories emerge from cross-dossier synthesis:

1. **The Calibration Problem**: Source #7, #166, and RubricForge collectively demonstrate that **accuracy is necessary but insufficient**. Future work must address stable miscalibration through either architectural change (Source #34's trusted-kernel pattern) or continuous rubric updating.

2. **The Multi-Axis Evaluation Mandate**: Source #166's four-axis framework (accuracy, paraphrase robustness, adversarial robustness, calibration) should be the **default evaluation contract** for any judge system. RubricForge currently satisfies only the accuracy and attribution axes.

3. **The Process-Outcome Distinction**: Source #8's finding that consistency and success rate are **separable axes** suggests that RubricForge's focus on verdict faithfulness (a process measure) is strategically sound, even if raw agreement does not improve.

### 5.4 Open Questions / 未解決課題

| Question | Why It Matters | Required Investigation |
|----------|---------------|----------------------|
| What is the **rubric half-life**? | Source #26 shows 73.2% instability under analytic variation | Longitudinal tracking of rubric-derived verdicts |
| Can rubrics be **compositionally combined**? | Source #8 shows cross-task consistency is distinct | Multi-rubric fusion experiments |
| Does rubric induction **transfer across model families**? | Source #13 shows LLM-judge is no stronger than evaluated agent | Cross-family induction benchmarks |
| How do rubrics interact with **agentic self-improvement**? | Source #36 (HELIX) couples harness evolution with model improvement | Closed-loop rubric-update studies |

---

## Summary / 総括

RubricForge delivers a **structurally sound but empirically modest** advance. Its primary contribution is not accuracy improvement—which is statistically indistinguishable from G-Eval—but **attribution traceability** in a landscape where evaluation artifacts face escalating regulatory and adversarial scrutiny (Sources #26, #123, #166). The method's design aligns with a broader industry shift from aggregate-score evaluation toward **multi-axis, specification-grounded, process-aware assessment**. Its deployment viability depends on addressing three unresolved tensions: (1) stable miscalibration (Source #7), (2) rubric instability under variation (Source #26), and (3) the process-outcome divergence documented by BCM (Source #8). Future work should treat RubricForge not as a standalone evaluator but as a **modular component** within a layered governance stack that couples induced rubrics with structural abstention, behavioral consistency metrics, and specification-driven audit trails.