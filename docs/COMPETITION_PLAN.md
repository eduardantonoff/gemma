# Gemma 4 Developer Agent competition plan

Status: working plan, 25 September 2026. The official competition package is extracted under `data/` and its `HARNESS_README.md` has been inspected. Competition rules still need an authenticated read before training-data or final-submission decisions.

## Objective and constraints

Build an autonomous coding agent that produces patches for unseen repository issues. Kaggle scores the percentage of issues whose patches pass validation tests. Submit `submission.zip` with a root `agent.yaml`; prompts, ADK skills, subagents and PEFT LoRA adapters are optional. Every model backed agent must use `gemma-4-31b-it-qat-w4a16-ct`. The evaluator allows 12 hours total for patch generation across tasks, including sandbox setup. Available tools include shell, file read/edit/write, patch submission, status, and code graph/embedding lookup.

Sources: [competition overview](https://www.kaggle.com/competitions/gemma-4-developer-agent/overview), [competition rules](https://www.kaggle.com/competitions/gemma-4-developer-agent/rules), [official model](https://huggingface.co/google/gemma-4-31B-it-qat-w4a16-ct).

The supplied guide specifies 4 L4 GPUs (96 GB total VRAM), a 32,768-token context, under 3 GiB unpacked submission size, and an offline task sandbox limited to 4 GiB RAM and 2 vCPUs. The base model is hosted by the evaluator; it does not belong in the ZIP. `submit_patch()` is free of the tool-call limit and should be the last tool action. Local evaluation is via `swegemma eval`; its results include per-task patches, test outputs, traces, and a summary.

## What was downloaded

The ignored `data/` directory contains `HARNESS_README.md`, `tasks.jsonl`, `sample_submission/`, `snapshots/`, `graphs/`, `embeddings/`, `wheels/`, `docker/`, and `sandbox/`. Extracted size is about 21 GiB, mostly 20 GiB of snapshots. The 129 released tasks span FastAPI (67), Rich (48), Requests (13), and HTTPX (1). Task records include reference patches and test patches; keep these out of the agent's inputs and any held-out evaluation set.

All 129 task IDs have a named graph and embedding file, but 61 task graph files and 66 task embedding files are zero bytes. Availability must be checked per task; a graph or semantic lookup must not be assumed. The guide says `search_similar_code` resolves symbol names against precomputed nodes, so natural-language issue text is not a reliable query. Start with shell search and file reads, then measure graph-tool value on tasks with nonempty files.

The sample submission references two small adapter files (about 213 KiB each); they are sample artifacts, not evidence of a trained useful LoRA. Its `eval_config.yaml` sets 60 seconds and 10 tool calls per task. Use a deliberate budget after measuring a baseline, rather than carrying that sample setting into a serious run. Its prompt also says to check patch size after calling `submit_patch()`, while the guide says that action ends the agent loop; do the diff check before submitting.

The included `data/wheels/` directory contains repository test dependencies, not the `swegemma`, `adk-submission`, or `adk-eval-core` evaluation packages. Obtain the official evaluation wheelhouse separately before attempting the documented CLI command. The `HARNESS_README.md` example uses a `competition_data/published/` prefix; in this extraction, the data files are directly under `data/`, so commands must use those paths.

Deadlines (UTC): optional paper 12 November; entry and team merger 25 November; final submission 2 December. Treat the paper track as a later decision, after a reproducible result.

## Sequence of work

1. **Freeze the actual contract.** Record file hashes and wheel versions. Read the authenticated rules for external data, external teachers, publication obligations and daily submission caps. Resolve any ambiguity from organizer discussion before training on third party outputs.
2. **Make a valid zero-adapter baseline.** Copy only the sample's config structure, remove its adapter references and unused subagent, set an intentional `eval_config.yaml`, and use the exact permitted model, documented tools, a short prompt, and a reliable `submit_patch` path. Validate config compilation and a local smoke run. Submit early enough to learn whether the hosted harness accepts the package.
3. **Measure on development tasks.** Split released tasks by repository or issue family into development and untouched holdout sets. Log each run's config hash, model revision, task ID, token/time use, tool trace, patch, test result and failure category. Score patches using the official local verifier where possible; do not expose reference patches/tests to the agent at inference.
4. **Improve one bottleneck at a time.** First test repository search and focused file reads; then editing and test-loop instructions; then graph and embedding tools where they improve localization. Compare each candidate against the same task set and time budget. Preserve a final untouched holdout check.
5. **Consider LoRA only with evidence.** Train an adapter only after the baseline exposes a repeatable failure that prompt/tool changes cannot fix. Keep training data provenance and train/dev separation. Validate the adapter against the exact quantized competition model and submission size limits before spending on cloud GPUs.
6. **Package and submit.** Re-run config validation, ZIP inspection, and local evaluation on the frozen candidate. Make a hosted submission and retain its artifact hash, public score and error logs. Select the final candidate by holdout quality, runtime and hosted compatibility, not leaderboard fluctuations alone.

## DGX Spark feasibility gate

The DGX Spark's published 128 GB unified memory and ARM64 Grace Blackwell platform make one local copy of the 23.3 GB compressed-tensors checkpoint plausible for **inference experiments**. This is a capacity argument, not a measured performance claim. The hosted evaluator's GPU setup differs, so benchmark throughput, tool latency and total task time separately. QLoRA or LoRA training may be feasible for small runs, but must be proven with a real memory/throughput pilot; full-model training is outside the initial plan.

Before installing or downloading anything, verify SSH reachability, `uname -m`, OS/CUDA/driver, available memory and disk, Docker GPU access, Python environment and compatible ARM64 vLLM/Transformers/PEFT packages. Reserve model cache, dataset/archive extraction, container images and run logs on disk. Run one generation with the exact model and chat/tool template, then one complete local development issue through the competition harness. Record tokens/second, peak memory, wall time, errors and thermal/power behavior. A different Gemma checkpoint can test plumbing, but cannot validate competition quality or adapter compatibility.

Current status: `spark.local` did not resolve from this environment on 25 September 2026; no live Spark inspection or benchmark has succeeded. Use a current host/IP supplied by the owner, rather than the older saved IP.

The Mac currently has about 16 GiB free after extracting the 21 GiB package. The Kaggle CLI is not installed in the active PATH. Move the large development data to Spark or other storage before downloading model weights or building evaluation containers. Keep the small plan and submission source in Git; keep snapshots, graphs, embeddings, wheels, models, results and credentials out of Git.

## Cloud burst criteria

Rent GPUs only when a measured task needs more throughput, memory, or parallel runs than Spark can provide: e.g. a validated adapter training recipe, a larger evaluation sweep, or a deadline bound rerun. First specify required VRAM, CPU/RAM, storage, ARM64 versus x86 package compatibility, expected run hours and maximum spend. Reproduce a small local run before scaling; checkpoint frequently and track cost per verified improvement. Prefer the same model revision and harness versions across Spark and cloud.

## Immediate prerequisites and open questions

- Kaggle account joined to the competition with rules accepted; the package is available, but authenticated rules and submission access still need confirmation.
- Current Spark SSH host/IP and available disk; confirm whether SSH, Docker, and Hugging Face model download are usable.
- Obtain the separate official evaluation wheelhouse, then install and version-pin `swegemma`, `adk-submission`, and `adk-eval-core` on the evaluation host. Kaggle CLI installation/authentication may help with downloads and submissions. Provide enough Spark disk for the 21 GiB extracted data, 23.3 GB checkpoint, containers and logs; keep credentials outside Git.
- Whether to pursue the optional paper track, a teammate, or a fixed cloud budget can be decided after the first baseline result.
- Confirm a current Spark SSH address and whether the ARM64 host can install/run the released harness wheels; avoid assuming the supplied x86-oriented Docker environment runs unchanged on ARM64.
- Read authenticated rules and current organizer clarifications for external model generated training data.

## First milestone

One valid `submission.zip` that the local compiler accepts, solves at least one released issue end to end, and records a reproducible baseline across a small fixed development panel. No training is required for this milestone.
