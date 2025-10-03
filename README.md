# pipeline-dispatcher

Azure DevOps dispatcher templates that sit between consumer repositories and `pipeline-common`. The dispatcher keeps the consumer surface minimal while enforcing shared defaults and locking the `PipelineCommon` resource reference.

## Quick Links
- `AGENTS.md` – AI-facing handbook that explains how defaults merge and how configuration flows.
- `wesley-trust/pipeline-common` – shared templates consumed by the dispatcher.
- `wesley-trust/pipeline-examples` – end-to-end samples showing the required `*.pipeline.yml` and `*.settings.yml` pairing.

## How It Works
1. Consumer settings files extend `/templates/pipeline-configuration-dispatcher.yml@PipelineDispatcher`.
2. That template merges `configurationDefaults` with the supplied `configuration` object and normalises environment metadata.
3. It then re-extends `/templates/pipeline-common-dispatcher.yml@PipelineDispatcher`, which declares the `PipelineCommon` repository resource (locking `ref`) and calls `templates/main.yml@PipelineCommon` with the composed `configuration`.
4. Downstream stages/jobs are owned by `pipeline-common`; consumers only manage parameters and action groups.

## Repository Layout
- `templates/pipeline-configuration-dispatcher.yml` – consumer entry point responsible for default merging and schema alignment.
- `templates/pipeline-common-dispatcher.yml` – intermediate template that declares the `PipelineCommon` resource and forwards configuration.
- `README.md` / `AGENTS.md` – human- and AI-focused documentation.

## Defaults and Overrides
- `parameters.configurationDefaults` seeds values such as agent pool, service connection, validation toggles, variable behaviour, additional repositories, and key vault metadata.
- Consumers override defaults simply by providing a value in their `configuration` object; explicit empty strings revert to the default.
- A baseline environment map (`dev`, `qa`, `ppr`, `prd`) is exposed. Consumers can adjust it via `parameters.environments` or disable environments with `parameters.skipEnvironments`.
- To pin a different `pipeline-common` version, set `configuration.pipelineCommonRef` in the consumer settings file.

## Consumer Workflow
1. Declare the dispatcher repository resource in the consumer `*.settings.yml` file with the required service connection.
2. Extend `/templates/pipeline-configuration-dispatcher.yml@PipelineDispatcher`, passing `parameters.configuration` that mirrors the schema described in `pipeline-common/docs/CONFIGURE.md`.
3. Optionally override `parameters.configurationDefaults` or `parameters.environments` when the consumer needs non-standard defaults.
4. Use pipeline parameters in `<service>.pipeline.yml` to control feature toggles (review, DR invocation, action enables) that flow into the configuration object.

## Quality Checks
- When defaults or schema change, run the preview scripts in `pipeline-common/tests` (see that repo’s README) to validate real Azure DevOps compilation with your branch.
- Keep README/AGENTS aligned with template changes so service teams understand how to adopt new defaults.

## Releasing Changes
- Tag or branch this repository when shipping breaking changes so consumers can opt in via `configuration.pipelineCommonRef` safely.
- Communicate updates through release notes that summarise new defaults, removed flags, or required consumer actions.

## Related Repositories
- `wesley-trust/pipeline-common` – shared pipeline implementation.
- `wesley-trust/pipeline-examples` – sample consumers for onboarding new services.
