# Gemma 4 Developer Agent

Working repository for the [Kaggle Gemma 4 Developer Agent competition](https://www.kaggle.com/competitions/gemma-4-developer-agent/overview). The [competition plan](docs/COMPETITION_PLAN.md) records the constraints, experiment sequence, and DGX Spark feasibility checks.

## Layout

| Path | Purpose |
| --- | --- |
| `docs/` | Research plan and decisions |
| `submission/` | Our versioned ADK agent config, prompts, and optional skills/adapters |
| `scripts/` | Reproducible local preparation and evaluation helpers |
| `data/` | Extracted Kaggle package, including `HARNESS_README.md`, tasks, snapshots, and the organizer's sample; ignored by Git |
| `results/` | Local evaluation traces, patches, and scores; ignored by Git |
| `models/` or `gemma-4-31b-it-qat-w4a16-ct/` | Downloaded model weights; ignored by Git |

The extracted Kaggle package is expected directly under `data/`. Its `sample_submission/` is an organizer reference, not our candidate submission. Keep credentials and large data out of Git.

## Next milestone

Create a minimal no-adapter agent in `submission/`, validate it with the official `swegemma` harness, and record a baseline on a fixed development subset. See [the plan](docs/COMPETITION_PLAN.md) before using the sample config: its time and tool limits are unusually small.
