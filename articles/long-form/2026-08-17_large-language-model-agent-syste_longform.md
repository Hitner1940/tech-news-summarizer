# Inducing Reward-Free Judging Rubrics & Agent Evaluation Integrity: A Bilingual Technical Investigation | 誘導獎勵免費評判標準與代理評估完整性：雙語技術調查

## 1. 📌 Executive Reconstruction / 核心重構摘要

**English:**
The arXiv literature from August 2026 reveals a structural pivot in AI evaluation and agent deployment. The central finding—captured by *RubricForge* (arXiv:2608.13564)—is that reward-free LLM-as-judge systems systematically over-credit fluent but unsuccessful trajectories, a bias rooted in surface-level linguistic coherence rather than outcome fidelity. Three convergent signals define the investigation: (1) evaluation metric fragmentation, where accuracy claims conceal calibration breakdowns (stable miscalibration per arXiv:2608.13591), behavioral inconsistency across tasks (BCM, arXiv:2608.13598), and version-instigated sample-level regression (arXiv:2608.13607); (2) emergent modular architectures within monolithic LLMs, demonstrated by circuit-level analyses showing brain-mirror specialization across language, reasoning, social, and physical domains (arXiv:2608.13667); and (3) the decoupling of agent capability from agent harness quality, where tool-use, memory, routing, and governance abstractions determine end-to-end reliability independently of base model proficiency (Agentao, arXiv:2608.13574; HELIX, arXiv:2608.13951). The consensus implication is that next-generation evaluation must shift from aggregate score optimization to process-level traceability, structural abstention, and verifiable reward alignment.

**繁體中文：**
2026年8月的arXiv文獻顯示，AI評估與代理部署正經歷結構性轉型。核心發現——由 *RubricForge*（arXiv:2608.13564）闡述——指出獎勵免費的LLM評判系統會系統性地過度誇獎流暢但失敗的軌跡，這種偏差源於表面語言連貫性而非結果忠實度。三個匯聚的信號定義了此次調查：(1) 評估指標碎片化，準確率聲稱掩蓋了校準崩潰（穩定性誤校準，見arXiv:2608.13591）、跨任務行為不一致（BCM，arXiv:2608.13598）以及版本引發的樣本級倒退（arXiv:2608.13607）；(2) 單體LLM內Emergent模組化架構，電路層級分析顯示大腦鏡像式專門化，涵蓋語言、推理、社會與物理領域（arXiv:2608.13667）；以及(3) 代理能力與代理執行框架品質的解耦，工具使用、記憶、路由與治理抽象層獨立於基礎模型熟練度決定端到端可靠性（Agentao，arXiv:2608.13574；HELIX，arXiv:2608.13951）。共識含義是，下一代評估必須從總分優化轉向流程級可追溯性、結構性拒絕與可驗證獎勵對齊。

---

## 2. 🔬 Architectural Deep-Dive / 底層架構剖析

### 2.1 Reward-Free Rubric Induction / 獎勵免費標準誘導

**English:**
RubricForge (arXiv:2608.13564) introduces reflective evolution: starting from ground-truth-labeled trajectories, the system evolves a judge rubric text that maximizes agreement with environment rewards, then freezes it for one-shot evaluation. On tau-bench (173 trajectories) and WebShop (160), the primary gain is *faithfulness*—verdicts are attributable to named criteria—rather than raw accuracy. The edge over G-Eval is not statistically significant (McNemar p = 0.248), but absolute-score calibration marginally favors the generic judge. This reveals a fundamental tension: induced rubrics improve interpretability without guaranteeing predictive superiority, suggesting that reward-free judging's bottleneck is not rubric quality but the absence of outcome-grounded supervision.

**繁體中文：**
RubricForge（arXiv:2608.13564）引入反思演化：從標註真實結果的軌跡出發，系統演化出與環境獎勵一致性最高的評判標準文本，然後凍結用於一次性評估。在 tau-bench（173個軌跡）與 WebShop（160個）上，主要增益是*忠實度*——裁決可歸因於明確標準——而非原始準確率。相比 G-Eval 的優勢未達統計顯著性（McNemar p = 0.248），但絕對分數校準 marginally 傾向通用評判。這揭示了一個根本張力：誘導標準提升可解釋性卻不保證預測優勢，暗示獎勵免費評判的瓶頸不在標準品質，而在缺乏結果錨定的監督。

### 2.2 Emergent Modular Cognitive Architecture /  Emergent模組化認知架構

**English:**
Source #3 (arXiv:2608.13667) presents circuit analyses across N=46 tasks spanning language, formal reasoning, social reasoning, and physical reasoning. The finding: LLMs develop modular organization mirroring the human brain—tasks drawing on the same cognitive network in humans recruit overlapping neurons in LLMs, while cross-domain tasks recruit distinct neurons. This convergence suggests modularity is a fundamental property of intelligent systems, not an evolutionary accident. Critically, this architecture emerges without explicit modular design, arising from scale and objective pressure alone.

**繁體中文：**
來源 #3（arXiv:2608.13667）展示了跨越語言、形式推理、社會推理與物理推理共46項任務的電路分析。發現：LLM發展出鏡像人類大腦的模組化組織——在人類中調用相同認知網絡的任務，在LLM中也會招募重疊的神經元；而跨領域任務則招募不同神經元。這種匯聚表明模組化是智能系統的根本屬性，而非演化偶然。關鍵在於，此架構無需顯式模組設計即出現，僅由規模與目標壓力產生。

### 2.3 Depth-Aware Expert Sensitivity in MoE / MoE深度感知專家敏感度

**English:**
arXiv:2608.13565 analyzes Qwen3.6-35B-A3B (40 MoE layers, 256 experts/layer, top-8 routing) via magnitude-based expert masking. Layer sensitivity is strongly depth-dependent: early layers (0–9) and middle layers (10–29) are fragile to masking, while late layers (30–39), especially very-late layers (35–39), tolerate aggressive masking of low-magnitude experts. At 300-prompt scale, flat masking retains only 150/300 Good+Similar outputs, whereas late-focused policies retain 249–255/300 while masking 640–1,145 experts. The narrow very-late policy (layers 35–39 @ 50%) achieves the strongest quality/masked-expert tradeoff (419/500 retained on held-out validation). This stratified tolerance implies compression strategies should be layer-adaptive, not uniform.

**繁體中文：**
arXiv:2608.13565 透過基於大小的專家遮罩分析 Qwen3.6-35B-A3B（40層MoE，每層256專家，top-8路由）。層級敏感度具有強烈深度依賴性：早期層（0–9）與中期層（10–29）對遮罩極度脆弱，而晚期層（30–39），特別是非常晚期層（35–39），能容忍對低 magnitude 專家的激進遮罩。在300提示規模下，全域遮罩僅保留150/300 Good+Similar輸出，而晚期聚焦策略保留249–255/300同時遮罩640–1,145個專家。狹義非常晚期策略（層35–39 @ 50%）在保留/遮罩專家權衡上表現最佳（驗證集保留419/500）。這種分層耐受性表明壓縮策略應為層級自适应，而非均勻。

### 2.4 Second-Think Parallel Reasoning / 第二思維並行推理

**English:**
arXiv:2608.13667 (Second Thought) identifies the "reasoning idle window" during ReAct-style agent execution—when the agent serializes an action and waits for environment observation, reasoning is frozen. The framework forks four auxiliary branches at each Thought conclusion, decoding concurrently with the main loop, then merging generated thoughts upon observation arrival. Across three agentic benchmarks and three reasoning LLMs, Second Thought lowers average turn count in all nine (model, benchmark) pairs and reduces main thread decoding by up to 43% (≈20% average) in six settings. Pass@1 shows no significant change in seven of nine pairs; the two significant differences are +12.4 and +10.2 points. Against a compute-matched control forcing equivalent budget onto the main thread, it attains strictly higher performance.

**繁體中文：**
arXiv:2608.13667（第二思維）識別了 ReAct 風格代理執行期間的「推理空窗期」——當代理序列化動作並等待環境觀察時，推理處於凍結狀態。該框架在每次思維結論時分支出四個輔助分支，與主循環並行解碼，並在觀察到達時合併生成的思維。 Across三項代理基準與三款推理LLM，第二思維在全部九組（模型，基準）對中降低平均回合數，並在六個設定中將主線程解碼減少高達43%（平均約20%）。Pass@1 在九組中的七組無顯著變化；兩組顯著差異為+12.4與+10.2點。相對於將等量計算預算強制投入主線程的控制組，其性能嚴格更優。

---

## 3. ⚖️ Official Claims vs Empirical Reality / 官方宣稱 vs 社群獨立實測矩陣

| Dimension / 維度 | Official Claim / 官方宣稱 | Empirical Finding / 實測發現 | Grade / 評級 |
|---|---|---|---|
| **Judge faithfulness** | RubricForge improves agent evaluation accuracy over G-Eval | No statistically significant edge (p=0.248); gain is in interpretability, not accuracy | ⚠️ Qualified / 有限制 |
| **LLM calibration** | High-confidence outputs are reliably correct | Stable miscalibration exists: confident wrong answers persist under perturbation (arXiv:2608.13591) | 🔴 Critical / 關鍵 |
| **Benchmark generalization** | SWE-bench/LiveCodeBench scores indicate general coding ability | Post-training on SWE-bench yields limited cross-task transfer; rankings fail to generalize (arXiv:2608.13566) | 🔴 Critical / 關鍵 |
| **Token efficiency of LSP** | Semantic retrieval via LSP saves tokens vs. lexical grep | LSP costs +6% to +118% tokens on symbol localization; saves tokens only for weakest models (arXiv:2608.13568) | ⚠️ Conditional / 條件性 |
| **Interpretability reproducibility** | Circuit discovery provides auditable explanations | 73.2% of specification pairs produce flip claims across defensible analytic variations (arXiv:2608.13754) | 🔴 Critical / 關鍵 |
| **Skill utility** | Skills improve agent performance over workflow memory | Skills work primarily via procedural anchoring (65.7% of cases), not knowledge injection (4.5%); retrieval becomes bottleneck as pools grow (arXiv:2608.14036) | 🟡 Nuanced / 微妙 |
| **Version upgrade benefit** | Newer models outperform predecessors on all samples | Sample-level regression is common; no universal signal predicts it (arXiv:2608.13607) | 🔴 Critical / 關鍵 |
| **Self-improvement reliability** | Continuous learning harnesses improve without retraining | GRPO rank allocation fails under RL vs. SFT; gradient landscape is fundamentally flatter (arXiv:2605.07366) | 🟡 Nuanced / 微妙 |

---

## 4. ⚙️ Hardware & Deployment Engineering / 硬體門檻與生產環境部署

**English:**
Deployment engineering emerges as the dominant reliability bottleneck. Five interdependent failure domains were identified across the dossier:

1. **Memory bandwidth and KV-cache pressure:** arXiv:2608.14191 proves that attention-aware transform coding achieves ~5.8× compression on Llama-3.1-8B and Qwen-2.5-7B at near-lossless accuracy, outperforming uniform quantization baselines that degrade in at least some settings. The key insight: quantization error must be measured through its attention-propagation impact, not cache reconstruction error.

2. **MoE routing imbalance:** arXiv:2608.13057 (TEMPO) demonstrates that expert time is neither linear in token count nor in activated-replica count. Below n*≈156–168 tokens, HBM weight streaming dominates (cost attaches to activated replicas, not tokens); above it, grouped GEMM rounds tokens to 128-tile M-tiles. A max-affine profile t=max(a+bG, c+βN) captures both regimes. Realistic decode batches hold hot experts in the linear regime and cold experts in the flat regime simultaneously; proxy dispatches differ by 1.4–1.6× in modeled block time (p95 up to 1.7×).

3. **Apple Silicon deployment fragility:** arXiv:2608.13987 documents five independent bugs preventing Nanbeige4.2-3B from running via Hugging Face transformers on MPS, including a silently-zeroed RoPE buffer. Even after bug fixes, the Looped Transformer's layer-reuse strategy doubles peak attention memory. A chunked-prefill strategy extends allowable context width by 2.7× on 32 GiB shared memory. Debugged, the model completes up to 30% of real agentic tasks (up from 0%).

4. **Batched inference pruning collapse:** arXiv:2608.14003 shows that training-free adaptive pruning methods severely degrade under batched inference because a single pruning mask must serve a batch, and threshold calibration on unaggregated activations no longer matches the aggregated distribution. The proposed solution—periodic top-k selection over aggregated importance scores plus activation memory—preserves speedup while stabilizing sparsity ratio.

5. **Token inflation in agentic workflows:** arXiv:2608.13571 defines token inflation as the ratio of true workflow cost to single-call cost. Systems like FrugalGPT route based on single-call cost, underestimating real cost by >2× on difficult tasks. InflationAgent measures inflation across model tiers, introduces CoT Branching Entropy (AUROC 0.887) as a pre-execution difficulty signal, and selects models by maximizing Semantic Exchange Rate. On GSM8K under fixed budget, it achieves 94.7% vs. 91.0% for FrugalGPT while using 31% fewer tokens.

**繁體中文：**
部署工程成為首要可靠性瓶頸。Across檔案識別出五個相互依賴的失效域：

1. **記憶體頻寬與KV快取壓力：** arXiv:2608.14191 證明注意力感知變換編碼在 Llama-3.1-8B 與 Qwen-2.5-7B 上達成約5.8×壓縮且近無損準確率，優於至少在某些設定下退化的均勻量化基線。關鍵洞察：量化誤差應透過其注意力傳播影響測量，而非快取重建誤差。

2. **MoE路由不平衡：** arXiv:2608.13057（TEMPO）證明專家時間既非線性於token計數亦非線性於激活副本計數。低於n*≈156–168 tokens時，HBM權重串流主導（成本附著於激活副本而非tokens）；高於此閾值時，分組GEMM將tokens圓整至128磁磚M-tiles。最大仿射輪廓 t=max(a+bG, c+βN) 涵蓋兩 regimes。實際解碼批次同時讓熱專家處於線性 regime、冷專家處於平坦 regime；代理調度在建模區塊時間上相差1.4–1.6×（p95高達1.7×）。

3. **Apple Silicon部署脆弱性：** arXiv:2608.13987 記錄五個獨立bug阻止 Nanbeige4.2-3B 透過 Hugging Face transformers 在 MPS 上運行，包括靜默置零的RoPE緩衝區。修復後，Looped Transformer的層重用策略仍使峰值注意力記憶體翻倍。分塊prefill策略將32 GiB共用記憶體上的可允許上下文寬度擴展2.7×。除錯後，模型完成高達30%的真實代理任務（從0%提升）。

4. **批次推理剪枝崩潰：** arXiv:2608.14003 顯示免訓練自适应剪枝方法在批次推理下嚴重退化，因為單一剪枝掩碼必須服務批次，且在未聚合激活上校準的閾值不再匹配聚合分佈。所提解決方案——對聚合重要性分數的週期性top-k選擇加上激活記憶——在穩定稀疏率的同時保留加速比。

5. **代理工作流的token膨脹：** arXiv:2608.13571 定義token膨脹為真實工作流成本與单次呼叫成本的比率。FrugalGPT等系統基於单次呼叫成本路由，在困難任務上低估真實成本超過2×。InflationAgent Across模型層級測量膨脹，引入CoT分支熵（AUROC 0.887）作為預執行難度訊號，並透過最大化語義交換率選擇模型。在固定預算的GSM8K上，達成94.7% vs. FrugalGPT的91.0%，同時減少31% tokens。

---

## 5. 📈 Strategic & Ecosystem Implications / 產業生態戰略影響

**English:**
Three strategic implications emerge with high confidence:

**(a) Evaluation must become process-aware, not outcome-only.** The Behavioral Consistency Metric (BCM, arXiv:2608.13598) demonstrates that cross-task and within-task consistency are distinct axes that can diverge. Systems can be locally reproducible yet globally fragmented. Consistency is not reducible to success rate. This demands evaluation frameworks that track execution traces, not just final answers—a shift from score-centric to process-centric benchmarking.

**(b) The model-harness coupling is the new frontier.** HELIX (arXiv:2608.13951) formalizes model-harness co-evolution: build harnesses for a fixed model, update the model from verified sibling trajectories, rebuild harnesses as model capabilities change. A 65-candidate portfolio discovers a fixed harness improving task coverage by 4.0% over Pi, while exposing up to 58.0% more verified coverage. The harness shapes both what a model can accomplish and the trajectories from which it learns—making harness design as consequential as model architecture.

**(c) Safety defense requires structural, not statistical, approaches.** Structural abstention (arXiv:2608.13926) proposes a trusted kernel with a generative shell: a component that can fabricate may influence which question the system answers, never which value it returns. This invariant separates interpretation from computation. Meanwhile, Trigger-style safety neuron clamping (arXiv:2608.14392) holds selected neurons at harmful-conditional mean activations, triggering refusal behavior learned during alignment—without compromising utility on benign requests. Both point toward a shift from output filtering to architectural containment.

The ecosystem is bifurcating: frontier models optimized for benchmark scores will continue to show diminishing generalization returns, while system-level engineering—harness design, evaluation transparency, structural safety, and deployment robustness—will capture increasing marginal value. The gap between "model capability" and "system reliability" is widening, and the agents that survive will be those engineered at the intersection.

**繁體中文：**
三項戰略含義以高置信度浮現：

**(a) 評估必須成為流程感知，而非僅關注結果。** 行為一致性指標（BCM，arXiv:2608.13598）證明跨任務與任務內一致性是可以分歧的兩個獨立軸線。系統可以局部可復現卻全局碎片化。一致性不可還原為成功率。這要求追蹤執行軌跡而非僅最終答案的評估框架——從分數中心轉向流程中心的基準測試。

**(b) 模型-執行框架耦合是新邊界。** HELIX（arXiv:2608.13951）形式化模型-執行框架共同演化：為固定模型構建執行框架，從經過驗證的同輩軌跡更新模型，隨模型能力提升重建執行框架。65個候選組合發現固定執行框架相對於Pi提升4.0%任務覆蓋率，同時揭示高達58.0%額外經過驗證的覆蓋。執行框架既塑造模型能完成的事，也塑造其學習的軌跡——使執行框架設計與模型架構同等重要。

**(c) 安全防護需要結構性而非統計性方法。** 結構性拒絕（arXiv:2608.13926）提出可信內核搭配生成外殼：可能捏造的組件可影響系統回答哪個問題，但絕不影响其返回哪個值。此不變量將解釋與計算分離。同時，觸發式安全神經元鉗制（arXiv:2608.14392）將選定神經元保持在有害條件均值激活上，觸發訓練期間學到的拒絕行為——而不損害良性請求的效用。兩者指向從輸出過濾到架構約束的轉變。

生態系正在二分：為基準分數優化的前沿模型將持續顯示 diminishing 泛化回報，而系統級工程——執行框架設計、評估透明性、結構性安全與部署穩健性——將捕捉日益增加邊際價值。「模型能力」與「系統可靠性」之間的差距正在擴大，存活下來的代理將在交匯處進行工程化。