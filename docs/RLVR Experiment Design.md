# Verifiable subtask training for terminal agents

Status: proposed pilot, 2026-11-04. No training jobs or paid resources have been launched.

## Research question

Does training a small dense terminal agent on independently verifiable recovery and discovery subtasks improve held-out end-to-end task completion, compared with outcome-only RL at matched rollout compute?

Keep verifier validity, diagnostic accuracy, and policy improvement as separate measurements. A convincing trajectory diagnosis is not evidence that its proposed training intervention works.

## Decisions

- Student candidate: `Qwen/Qwen3-4B`, a dense open-weight model. Pin checkpoint and tokenizer revisions before execution. Use a fixed thinking mode and terminal protocol across arms. Select a larger student only if baseline capability is too low on genuinely simple tasks; first check harness compatibility and task difficulty.
- Teacher/diagnostician: existing Luna runs, plus new runs on training tasks if needed. Freeze judge model, rubric, prompt, and decoding settings. Audit its labels against human assessments and controlled interventions.
- Reward: executable outcome verification initially. Judge labels are diagnostic metadata and curriculum proposals, not reward bonuses.
- Evaluation: freeze the existing Terminal-Bench 2.0 snapshot as an external transfer set; record its exact revision and known defects. Use independent development and internal test families for routine iteration. Do not claim this old snapshot represents the current leaderboard.
- Training: independently authored small terminal tasks first; consider external datasets only after provenance, training permissions, overlap, and executor quality checks.
- Trainer candidate: TRL GRPO with a custom multi-turn environment adapter. Harbor supplies task packaging and evaluation; saved Harbor trajectories alone do not supply a GRPO training integration.
- Compute budget: up to $70 in available GCP credits, with Colab for additional small RL experiments. GPU access, credit eligibility, rollout lengths, and measured memory/throughput remain to be checked. Do not assume 4B parameter size determines total training memory.

## Dataset boundaries and provenance

Terminal-Bench 2.0 is a curated task collection. The Harbor/Terminal-Bench registry supports multiple independent datasets; those datasets are not automatically constituent training splits of Terminal-Bench.

Never export Terminal-Bench prompts, solutions, trajectories, or judge-generated derivatives into training. Its task files explicitly prohibit benchmark data in training corpora. Use current runs for verifier auditing and evaluation-method development. Because the existing 16-task subset has already been inspected, report it as an audited evaluation subset, not an untouched final test.

Create roughly 12 small training task families initially, with multiple seeded instances. Split whole families or underlying templates into train/development/internal-test before generating instances. More instances of one template do not constitute more independent tasks. Start with approximately eight train families and two each for development and internal test; this is a feasibility experiment with limited generalization evidence.

Candidate families: locate a configuration file; locate a symbol in a small repository; parse a local log; repair a command invocation; discover an installed executable; recover using an available alternative; transform a structured file; fix a small failing program. Include composed tasks linking two or three skills.

External candidate: `open-r1/codeforces` has an existing executable-code training ecosystem. Check the pinned dataset card and source terms before adoption. It is useful for coding coverage, but is not by itself a tool-discovery curriculum. Do not infer training permission from registry availability. Record source URL/revision, license, original problem ID, parent family, overlap checks, and any adapters.

For every admitted task:

1. Write explicit instructions and independently specified success criteria.
2. Supply a reference solution and verify it in the actual image, within resource limits.
3. Check no-op and deliberately incorrect outputs; test valid alternate solutions.
4. Freeze local inputs, dependencies, image digest, hidden tests, and resource budgets.
5. Keep verifier code and reference answers inaccessible to the actor.
6. Record oracle feasibility separately from student solvability. Strong-model success is additional evidence, not proof of a sound task.

## Diagnosis and controlled interventions

Store observations separately from inferred causes. A failed search does not prove context is absent; `command not found` does not establish that no viable alternative exists.

| Observable event | Candidate diagnosis | Stronger evidence |
| --- | --- | --- |
| Named executable missing | Tool unavailable | Trusted image inventory and PATH/permission checks |
| Repeated empty searches | Poor search strategy or absent context | Ground-truth file/symbol location and search scope |
| Tool call rejected | Invalid arguments or harness mismatch | Validated schema and raw error |
| Service unavailable | Infrastructure issue | Health checks or independent reproduction |
| Essential file absent | Unanswerable instance or intended missing-context task | Task manifest and explicit success criteria |

Construct matched training/development variants: preferred tool present; preferred tool absent with equivalent alternative present; required context present at a different location; truly required context absent. Hold unrelated conditions constant and randomize seeds/order. Verify that each variant has an appropriate success criterion. For an intentionally impossible variant, truthful identification of the missing prerequisite must be specified as success; otherwise exclude it from the completion curriculum.

The judge emits event spans, evidence references, candidate cause, alternative causes, and abstention status. Treat numerical judge confidence as uncalibrated until tested. Measure diagnostic accuracy on known interventions and a blinded human-reviewed sample. Keep reviewer-visible verifier verdicts hidden during initial annotation. Delimit untrusted trajectory text and prevent actor text from becoming judge instructions.

## Subtasks and credit assignment

Start with predefined, checkable skills rather than arbitrary judge-created segments. A training subtask needs an initial state, available observations/tools, a bounded action horizon, a terminal predicate, and a reproducible reset.

Example: identify an executable satisfying a stated capability in an image where the preferred command is unavailable. Verify the selected executable by running a capability probe. Searching or mentioning a tool earns no reward by itself.

Retrospective segments can propose new tasks. Before training, recreate the exact pre-segment filesystem/process state or author a fresh equivalent task with a verified starting state. A text transcript alone does not reproduce a terminal environment. Avoid placing information learned later in the trajectory into the subtask prompt.

Use independent subtask episodes for the first curriculum experiment. This shortens the learning horizon without claiming that vanilla GRPO performs explicit per-step causal credit assignment. Test whether acquired skills transfer to composed tasks. Process-reward shaping can be a later ablation after its validity is independently established.

## Roles of trajectories from other models

Teacher trajectories can support task discovery, executable reference solutions, audited SFT demonstrations, and preference datasets, subject to source/model terms and leakage checks. Dense versus mixture-of-experts teacher architecture does not prevent knowledge transfer. Different action formats, skill levels, and inaccessible tools can make demonstrations unsuitable.

For standard GRPO, generate fresh student-policy rollouts. An archived teacher completion is not an on-policy student sample. Off-policy policy-gradient learning needs an explicitly supported method and appropriate behavior-policy information; do not feed teacher traces into ordinary GRPO as if generated by the student.

If the student cannot use the terminal protocol, optionally perform a small common SFT warm start on training-only examples. Give every experimental arm the same warm-start checkpoint. This becomes the baseline for RL comparisons.

## GRPO rollout and update loop

1. Sample a batch of training task instances.
2. For each instance, create G=4 isolated environments with identical initial state and tool conditions.
3. Generate four independent multi-turn trajectories from the same current student checkpoint. Different sampling seeds create action diversity; they must not silently change task difficulty within a group.
4. Verify final states outside actor control. Record valid binary completion rewards. Exclude/resample infrastructure-invalid episodes under a predeclared rule; do not silently convert them into student failures. Log their count and resampling cost. Agent errors and budget exhaustion remain failures under the specified task contract.
5. Compute relative advantages within each task group. For an illustrative reward vector [0, 0, 1, 0], mean-centering yields [-0.25, -0.25, 0.75, -0.25]; the selected GRPO implementation may additionally normalize by reward standard deviation.
6. Update using the trainer's clipped policy objective and configured regularization. Train only on actor-generated tokens, including generated tool calls; mask tool outputs, prompts, verifier data, and padding. Preserve actual token IDs, action masks, policy revision, and rollout log probabilities required by the integration.
7. Synchronize generation with updated weights before collecting the next batch. Record policy staleness if asynchronous generation is later introduced.

When all group rewards are equal, the task-relative reward advantage is zero; regularization can still contribute to the loss. Track the fraction of mixed-success groups. Adapt difficulty on training/development evidence if nearly all groups are all-zero or all-one. Do not select tasks using final-test performance.

Example sizing only: eight task instances times four trajectories gives 32 episodes per collection batch. At a 12-action cap this is at most 384 tool actions, plus generation, verification, and reset costs. Several episodes can run sequentially if sandbox quotas are small. Measure tokens, wall time, GPU-hours, and environment usage before committing to a training length.

## Experimental arms

| Arm | Training |
| --- | --- |
| A | Common starting checkpoint, no RL |
| B | Outcome-only GRPO on composed training tasks |
| C | Outcome-only GRPO on a fixed mixture of verifiable subtasks and composed training tasks |

Suggested initial C mixture: 50% subtask episodes, 50% composed episodes, treated as a pilot choice. Match B and C primarily by generated actor tokens and report environment steps and GPU-hours too. Hold initialization, task exposure policy, optimizer, limits, and verifier versions fixed where applicable. A later diagnostic-guided curriculum versus a random subtask curriculum is needed to isolate the value of LLM diagnosis from simply adding easier training.

Primary outcome: held-out composed-task completion under executable verification. Secondary outcomes: terminal benchmark transfer, performance in controlled tool/context variants, verifier error rates, diagnostic accuracy, cost per success, and tool-recovery success. Report paired task-family comparisons and task/family-level uncertainty; rollout repetitions do not create independent task families. One training seed establishes feasibility only; confirm across multiple seeds before claiming a robust training effect.

Use internal development for checkpoint selection. Run the frozen external evaluation after selection, with fixed harness, tools, budgets, prompts, and repeated evaluation settings across arms. Report audited benchmark defects separately from official scores rather than silently changing the test set.

## First milestones

1. Build three task families and an artifact-complete Harbor evaluation path. Confirm oracle, negative controls, final-state capture, and state resets.
2. Run the student four times on ten training/development instances. Inspect protocol competence, cost, reward variation, and failure attribution.
3. Implement one end-to-end training batch and verify weight synchronization, identical group states, action masks, finite loss, and reproducible reward computation.
4. Run B and C only after that integration passes. Decide training length from measured compute; do not launch a full benchmark sweep as a substitute for the integration check.

## Budget-constrained execution plan

Use Colab for notebook development, reward checks, and short trainer smoke tests when a GPU is available. Runtime duration and GPU type are variable; save adapters, optimizer state, configurations, and metrics externally so interruption is recoverable. Do not assume GCP credits pay for hosted Colab subscriptions or compute units; treat those as separate products unless the account explicitly supports the intended billing arrangement.

Use GCP for a reproducible short pilot. First check whether the credits are eligible for GPU use, whether billing is still in restricted Free Trial mode, and whether regional GPU quota is available. Free Trial restrictions can prevent GPU provisioning; the credit balance alone does not establish access. Target one L4 (24 GB) as a candidate, not a guaranteed fit. Use LoRA/quantization if supported by the selected trainer, short contexts, bounded tool outputs, and sequential rollout generation to reduce memory pressure. Preserve the same setup across arms. If the 4B model does not fit, use a smaller dense student for integration before spending on larger hardware.

Proposed allocation, not a price quotation:

| Stage | Maximum allocation |
| --- | --- |
| GPU compatibility, memory, and throughput smoke test | $10 |
| Short matched B/C training pilots | $35 total |
| Held-out evaluation | $15 |
| Storage, retries, and reserve | $10 |

Quote the full regional VM price, disk, and any networking before provisioning. Derive runtime from remaining stage dollars divided by the measured all-in hourly rate. Set VM runtime limits/automatic shutdown and checkpoint regularly; billing alerts are notifications, not hard spending caps. Spot instances are optional after checkpoint/restart works. Track credits consumed and any non-credit-eligible charges separately. Existing API and sandbox costs must either fit these allocations or be explicitly budgeted outside them.

The $70 target is a feasibility pilot, not a commitment to statistically powered multi-seed training or a full external benchmark sweep. Prioritize a correct update, a measurable short-run learning signal, and a small held-out comparison. Expand only after observed costs justify it.

## Sources

- Terminal-Bench 2.0 announcement and benchmark-data restriction: https://www.tbench.ai/news/announcement-2-0
- Terminal-Bench task collection paper: https://arxiv.org/abs/2601.11868
- Harbor dataset registry: https://hub.harborframework.com/
- Dense student candidate and model card: https://huggingface.co/Qwen/Qwen3-4B
- GRPO, tool environments, custom rollout hooks: https://huggingface.co/docs/trl/grpo_trainer
- Executable coding-training candidate: https://huggingface.co/datasets/open-r1/codeforces/blob/main/README.md
- Open-R1 training examples: https://github.com/huggingface/open-r1
- DeepSeek-R1 and verifiable rewards: https://arxiv.org/abs/2501.12948
- GCP GPU types and memory: https://docs.cloud.google.com/compute/docs/gpus
- GCP Free Trial restrictions: https://cloud.google.com/free/docs/free-cloud-features
- Colab availability and runtime limits: https://research.google.com/colaboratory/faq.html

Pin implementation versions before use: live trainer documentation contains experimental APIs and does not establish compatibility with this repository's installed Harbor environment.
