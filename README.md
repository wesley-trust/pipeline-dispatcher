# pipeline-dispatcher

Dispatcher template for Azure DevOps pipelines that centralises the link between consumer repositories and the shared `pipeline-common` templates.
It now also ships an opinionated default configuration so consumer pipelines can omit common boilerplate.

## Purpose
- Declares the `PipelineCommon` repository resource and locks the reference (branch/tag).
- Re-extends `templates/main.yml@PipelineCommon`, forwarding the consumer-provided `configuration` object after applying defaults.
- Keeps dispatcher logic isolated so consumers only manage their pipelines/settings files.
- Supplies shared defaults (pool, service connection, variables, validation, etc.) when the consumer leaves values empty.

## Files
- `templates/pipeline-configuration-dispatcher.yml` – consumer entry point that merges `configurationDefaults` with the consumer-provided `configuration` object and exposes default environment metadata.
- `templates/pipeline-common-dispatcher.yml` – intermediate template that locks the `pipeline-common` repository and forwards the composed configuration (`template: /templates/pipeline-common-dispatcher.yml@PipelineDispatcher`).

## Default configuration
- `configurationDefaults` seeds values for pool, service connection, action groups, validation flags, variable behaviour, additional repositories, and key vault metadata.
- Defaults apply when a consumer omits a property or sets it to an empty string. Providing any other value overrides the default.
- A baseline environment map (`dev`, `qa`, `ppr`, `prd`) is exposed. Consumers can override individual environments or toggle them off by setting `skipEnvironments.envName: true`.

## Usage
1. In the consumer repo, declare this dispatcher as a repository resource (see `wesley-trust/pipeline-examples`).
2. Extend `/templates/pipeline-configuration-dispatcher.yml@PipelineDispatcher`, passing your `configuration` overrides only for values that differ from the defaults.
3. Optionally adjust per-repository defaults (e.g. pool image, variable groups) by overriding `parameters.configurationDefaults`.
4. Override or prune environments by supplying `parameters.environments` and/or `parameters.skipEnvironments`.
5. Override the version of `pipeline-common` by setting `configuration.pipelineCommonRef`.

## Related Repositories
- `wesley-trust/pipeline-common` – shared templates, steps, scripts, and documentation.
- `wesley-trust/pipeline-examples` – end-to-end sample pipelines that demonstrate how to call this dispatcher.
