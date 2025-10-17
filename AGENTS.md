# Agent Handbook

## Mission Overview
- **Repository scope:** Azure DevOps dispatcher templates that sit between consumer pipelines and the shared `pipeline-common` repo. Anything here should stay thin and focus on composing defaults and locking refs.
- **Primary entry point:** `templates/pipeline-configuration-dispatcher.yml` – consumers extend this template from their `*.settings.yml` files.
- **Contract:** Fetch `pipeline-common` as repository `PipelineCommon`, merge defaults with consumer overrides, and re-extend `/templates/main.yml@PipelineCommon`.
- **Stable surface:** Keep behaviour changes backwards compatible; emit tags or branches for controlled roll-out via consumer refs.

## Files to Know
- `templates/pipeline-configuration-dispatcher.yml` – merges `configurationDefaults` with the consumer-supplied `configuration`, exposes default environment metadata, and returns the object expected by `pipeline-common`.
- `templates/pipeline-common-dispatcher.yml` – declares the `PipelineCommon` resource, locks the ref, and re-extends the shared main template.
- `README.md` – summarises purpose, usage, and default behaviour. Update alongside template changes.

## Defaults & Overrides
- `configurationDefaults` seeds pools, service connections, validation toggles, tokens, additional repositories, and key vault metadata. Treat defaults as opinionated scaffolding – consumer configuration wins by simply providing a value.
- Environments: the dispatcher exposes a baseline map (`dev`, `qa`, `ppr`, `prd`). Consumers can drop or override entries through their settings template parameters (`skipEnvironments`, `environments`), and can append an environment suffix by setting `configuration.pipelineType` (e.g. `AUTO` for automated lanes).
- Region controls: `skipRegions` toggles primary/secondary deployment stages in bulk, while `globalDependsOn` injects a dependency across every environment stage (most commonly `validation`).
- Locking refs: default ref is `refs/heads/main`. Consumers can pin a tag or branch via `configuration.pipelineCommonRef`. Keep release notes so teams know when to upgrade.

## Working with Consumers
1. Consumer repo holds two files (`*.pipeline.yml`, `*.settings.yml`). The pipeline file extends its settings file; the settings file declares `PipelineDispatcher` and extends `/templates/pipeline-configuration-dispatcher.yml@PipelineDispatcher`.
2. `parameters.configuration` is the sole payload that flows into `pipeline-common`. Keep its schema aligned with `docs/CONFIGURE.md` in that repo.
3. When defaults need to evolve, add a feature flag or schema version where possible. Coordinate changes with `pipeline-common` so both sides stay compatible.

## Related References
- `pipeline-common/AGENTS.md` – full deep-dive into shared templates, stages, and configuration semantics.
- `pipeline-common/docs/CONFIGURE.md` – canonical parameter and environment documentation.
- `pipeline-examples` repo – reference consumer pipelines demonstrating how to call this dispatcher.

## Typical Tasks
- Add or tweak defaults -> update `configurationDefaults`, adjust documentation, and add tests in consumer repos if needed.
- Update the locked ref -> change `resources.repositories[*].ref` in `pipeline-common-dispatcher.yml` and publish a release note.
- Investigate consumer issues -> reproduce by checking out the consumer repo, inspecting their settings overrides, and tracing the merged configuration through this dispatcher into `pipeline-common`.

## Before You Ship
- Run consumer smoke tests (e.g., `pipeline-examples`) against your branch using the preview scripts in `pipeline-common/tests` when defaults or wiring change.
- Ensure documentation updates accompany behavioural changes so other repos know what to adopt.
