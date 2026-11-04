
**Primary question:** How does verifier quality influence benchmark reliability and post-training data quality?

**Daily allocation:** 45 minutes reading, 3–4 hours implementation, 60 minutes analysis, 30 minutes documentation or upstream contribution.

---

## Sprint 1 — Measurement & Benchmark Design

**Goal:** Build a statistically defensible evaluation dataset and establish baselines.

### Day 1 — What makes a benchmark valid?

**Read:**

- [Evaluation and Benchmarking of LLM Agents: A Survey (KDD 2025)](https://doi.org/10.1145/3711896.3736570)
    
- [Introducing Harbor-Index](https://harbor-index.org/)
    

**Learn:** Construct validity, task distributions, metric selection, benchmark difficulty versus broken tasks.

**Exercises:**

- Analyze three existing agent benchmarks. Identify their target capabilities, task distributions, success criteria, and possible shortcuts.
    
- Define five failure categories for document-investigation agents: missing findings, wrong evidence, incorrect citations, conflicting statements, and hallucinated claims.
    
- Write a short benchmark specification identifying exactly what success means.
    

**Build:** Create the VerifierLab repository, initialize Harbor, implement one task with an oracle reference solution and a no-op baseline.

**Deliverable:** `benchmark_spec.md` and one validated Harbor task.

### Day 2 — Statistical reasoning and experimental design

**Read:**

- [SciPy: Bootstrap Confidence Intervals](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.bootstrap.html)
    
- Revisit the sampling and metric sections of the KDD agent-evaluation survey.
    

**Learn:** Paired experimental designs, bootstrap intervals, pass@k, pass^k, within-task variance, and statistical power.

**Exercises:**

- Simulate two models with different underlying success probabilities and investigate when their rankings become unreliable.
    
- Implement task-level bootstrapping for an estimated difference in success rates.
    
- Compare uncertainty from repeated rollouts against uncertainty from sampling different tasks.
    
- Explain why 100 repeated runs on one task are not equivalent to 100 independent tasks.
    

**Build:** Implement an evaluation analyzer with task-level pass rates, paired model comparisons, confidence intervals, and seed recording.

**Deliverable:** `analysis/statistics.py`, tests, and an initial analysis notebook.

### Day 3 — Synthetic benchmark generation and quality control

**Read:**

- [Benchmarking LLMs Under Data Contamination (EMNLP 2025)](https://aclanthology.org/2025.emnlp-main.511.pdf)
    
- [Harbor Task Quickstart](https://github.com/harbor-framework/docs/blob/main/quickstart.mdx)
    

**Learn:** Synthetic data generation, data provenance, benchmark contamination, difficulty control, train/test leakage, and human validation.

**Exercises:**

- Generate fictional document collections with known inconsistencies and obligations.
    
- Construct easy, medium, and difficult task variants.
    
- Compare randomly generated splits against splits separated by underlying document templates.
    
- Manually review 10 generated tasks for ambiguity, answerability, and unintended shortcuts.
    

**Build:** Create an initial 24-task Harbor-compatible benchmark with immutable task IDs, hidden reference answers, and train/development/test splits.

**Deliverable:** `taskgen/`, `tasks/`, `dataset_manifest.jsonl`, and Benchmark v0.1.

**Sprint checkpoint:** Can another engineer reproduce your task set and distinguish genuine agent failures from defective instructions?

---

## Sprint 2 — Verifier Design & Reliability

**Goal:** Demonstrate that verifier design changes the quality of evaluation measurements.

### Day 4 — Deterministic verifiers and failure modes

**Read:**

- [Harbor Task and Verifier Architecture](https://github.com/harbor-framework/harbor/blob/main/AGENTS.md)
    
- Revisit Harbor-Index's discussion of instruction-verifier mismatch.
    

**Learn:** Outcome verification, execution-based grading, false positives, false negatives, reward exploits, and grader isolation.

**Exercises:**

- Create a weak verifier based on JSON validity and keyword matching.
    
- Generate deliberately incorrect submissions that satisfy the weak verifier.
    
- Build tests for incorrect citations, omitted findings, duplicated evidence, and unsupported claims.
    
- Identify which failures cannot reliably be detected using simple string matching.
    

**Build:** Implement `weak_verifier.py`, a deterministic test suite, and a collection of labeled negative-control outputs.

**Deliverable:** A reproducible demonstration of at least one verifier false positive.

### Day 5 — Strong verifiers and LLM-as-Judge

**Read:**

- [Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena](https://papers.neurips.cc/paper_files/paper/2023/hash/91f18a1287b398d378ef22505bf41832-Abstract-Datasets_and_Benchmarks.html)
    
- [AgentJudgeBench (EMNLP 2026)](https://aclanthology.org/2026.emnlp-main.659/)
    

**Learn:** Rubric construction, judge calibration, evidence verification, position bias, inter-rater agreement, and reward validity.

**Exercises:**

- Design a rubric with separate criteria for finding correctness, evidence support, citation validity, and completeness.
    
- Implement structural and evidence-grounded checks.
    
- If using an LLM judge, compare its decisions against a manually reviewed sample and test sensitivity to wording or response order.
    
- Create a disagreement report comparing weak, strong, and independent gold judgments.
    

**Build:** Implement `grounded_verifier.py` and an audit interface for reviewing disagreements.

**Deliverable:** Verifier v1 plus an initial calibration dataset.

### Day 6 — Verifier reliability experiments

**Read:**

- [Harbor Contribution Guidelines](https://github.com/harbor-framework/harbor/blob/main/CONTRIBUTING.md)
    
- Review Harbor's verifier implementation and test coverage.
    

**Learn:** Verifier flakiness, oracle validation, negative controls, sensitivity analysis, test isolation, and regression testing.

**Exercises:**

- Repeatedly execute oracle, no-op, and deliberately corrupted submissions.
    
- Test whether repeated grading produces identical outputs when task state is held fixed.
    
- Measure the false-positive rate and false-negative rate of each verifier.
    
- Examine whether failure rates change across task-difficulty categories.
    
- Reproduce one upstream Harbor issue related to verifier execution or result integrity.
    

**Build:** Implement a verifier reliability suite, regression tests, and a report showing how the verifiers disagree.

**Deliverable:** `verifier_audit.py`, a verifier reliability report, and one upstream issue or focused PR where justified.

**Sprint checkpoint:** Can you provide evidence that one verifier is more trustworthy than another without relying exclusively on that verifier's own scores?

---

## Sprint 3 — Evaluation Data to Post-Training Data

**Goal:** Convert agent trajectories into audited, provenance-aware training examples.

### Day 7 — Agent trajectories and training data schemas

**Read:**

- [Harbor Agent and Trajectory Architecture](https://github.com/harbor-framework/harbor/blob/main/AGENTS.md)
    
- [TRL Dataset Formats](https://github.com/huggingface/trl/blob/main/docs/source/dataset_formats.md)
    

**Learn:** Agent trajectory formats, tool-call serialization, provenance, reward annotations, SFT examples, and preference pairs.

**Exercises:**

- Inspect Harbor's saved agent traces and reward artifacts.
    
- Identify the information needed to reconstruct an agent's action sequence.
    
- Design a schema containing task ID, model, harness version, rollout, verifier version, score, and label provenance.
    
- Implement validation for missing steps, invalid rewards, duplicate examples, and missing references.
    

**Build:** Create a trajectory ingestion and validation pipeline.

**Deliverable:** `trajectory_loader.py`, a canonical rollout schema, and round-trip serialization tests.

### Day 8 — Generating useful training trajectories

**Read:**

- [RewardBench: Evaluating Reward Models](https://arxiv.org/abs/2403.13787)
    
- [Harvey: Post-Training RLM Agents for M&A Diligence](https://www.harvey.ai/blog/post-training-rlm-agents-for-m-and-a-diligence)
    

**Learn:** Outcome rewards, reward density, rejection sampling, successful trajectory selection, and diversity of training examples.

**Exercises:**

- Run multiple agent trials on training tasks with fixed configurations.
    
- Collect successful, failed, and partially successful trajectories.
    
- Compare filtering based on weak versus grounded verification.
    
- Identify cases where both verifiers assign similar scores but independent gold labels disagree.
    
- Measure how much data each filtering method retains.
    

**Build:** An automated pipeline that runs agents, captures trajectories, regrades outputs, and assigns quality metadata.

**Deliverable:** An initial target of 100+ rollouts, subject to inference cost and available time.

### Day 9 — Constructing SFT and preference datasets

**Read:**

- [Direct Preference Optimization (NeurIPS 2023)](https://papers.neurips.cc/paper_files/paper/2023/hash/a85b405ed65c6477a4fe8302b5e06ce7-Abstract-Conference.html)
    
- [TRL DPO Trainer](https://github.com/huggingface/trl/blob/main/docs/source/dpo_trainer.md)
    

**Learn:** Rejection-sampling SFT, preference labeling, reward-model agreement, pairwise ranking, and training-data bias.

**Exercises:**

- Construct SFT examples from high-quality agent outcomes.
    
- Construct chosen/rejected pairs from different responses to the same task.
    
- Compare preference labels derived from weak versus strong verifiers.
    
- Identify and eliminate examples where preference order is ambiguous.
    
- Verify that test tasks and their reference answers do not enter training exports.
    

**Build:** Implement SFT and DPO-compatible dataset exporters with provenance and quality filters.

**Deliverable:** `build_sft.py`, `build_preferences.py`, dataset cards, and quality-control results.

**Sprint checkpoint:** Can you demonstrate that the grounded verifier selects training examples that are more often genuinely correct according to independent labels?

---

## Sprint 4 — Post-Training Experiment & Publication

**Goal:** Validate the full pipeline, publish the evidence, and make it reproducible.

### Day 10 — Pilot post-training experiment

**Read:**

- [Hugging Face TRL SFT Trainer](https://huggingface.co/docs/trl/sft_trainer)
    
- Review Harvey's rejection-sampling self-distillation experiment.
    

**Learn:** SFT, parameter-efficient fine-tuning, held-out evaluation, controlled training comparisons, and why reward improvements can fail to generalize.

**Exercises:**

- Construct two training datasets of matched size: one selected by the weak verifier and one by the grounded verifier.
    
- Fine-tune the same small open-weight model using identical training settings, if the dataset size and compute budget make this feasible.
    
- Otherwise, run a training-pipeline smoke test and evaluate the exported datasets without claiming model improvement.
    
- Record training configurations, data counts, seeds, and compute usage.
    

**Build:** Minimal reproducible fine-tuning script using TRL and optional PEFT/LoRA.

**Deliverable:** Training script and experiment configurations, with checkpoint comparison when feasible.

### Day 11 — Held-out evaluation and ablations

**Read:**

- Revisit [Harvey's post-training methodology](https://www.harvey.ai/blog/post-training-rlm-agents-for-m-and-a-diligence).
    
- Review the statistical testing methods from Day 2.
    

**Learn:** Ablation studies, generalization, uncertainty, performance regressions, and separating selection bias from capability gains.

**Exercises:**

- Freeze the configurations and run on previously untouched test tasks.
    
- Compare weak- and grounded-verifier selection precision.
    
- If fine-tuning succeeded, compare base, weak-selected SFT, and grounded-selected SFT models using independent gold evaluation.
    
- Bootstrap paired task-level differences.
    
- Analyze failure cases where the stronger verifier performs worse.
    

**Build:** Evaluation report generator and summary plots.

**Deliverable:** `results/`, experiment tables, figures, and an initial research findings document.

### Day 12 — Reproducibility, release, and upstream contribution

**Read:**

- Review Harbor's contribution standards and examples of recently accepted PRs.
    
- Revisit your original hypotheses and benchmark specification.
    

**Learn:** Scientific reproducibility, engineering documentation, technical communication, and honest reporting of uncertainty.

**Exercises:**

- Reproduce the complete pipeline from a clean environment.
    
- Run automated tests and confirm train/test isolation.
    
- Document limitations, unsuccessful experiments, and unanswered questions.
    
- Prepare focused upstream Harbor contributions that arose from actual debugging.
    
- Write a research-style technical report and create a short demonstration.
    

**Build:** A public GitHub repository with setup instructions, sample data, experiment configurations, generated artifacts, and research results.

**Deliverable:** VerifierLab v1, technical report, demo, and evidence-ready resume/LinkedIn material.

**Final checkpoint:** Another engineer should be able to reproduce the reported verifier comparison and generate training-ready data without relying on unpublished manual steps.