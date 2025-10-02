# pipeline-dispatcher

Dispatcher template for Azure DevOps pipelines that centralises the link between consumer repositories and the shared `pipeline-common` templates.

## Purpose
- Declares the `PipelineCommon` repository resource and locks the reference (branch/tag).
- Re-extends `templates/main.yml@PipelineCommon`, forwarding the consumer-provided `configuration` object unchanged.
- Keeps dispatcher logic isolated so consumers only manage their pipelines/settings files.

## Files
- `templates/pipeline-configuration-dispatcher.yml` – first entry point referenced by consumer settings files (`template: /templates/pipeline-configuration-dispatcher.yml@PipelineDispatcher`).
- `templates/pipeline-common-dispatcher.yml` – second entry point referenced by consumer settings files or `pipeline-configuration-dispatcher.yml` (`template: /templates/pipeline-common-dispatcher.yml@PipelineDispatcher`).

## Usage
1. In the consumer repo, declare this dispatcher as a repository resource (see `wesley-trust/pipeline-examples`).
2. Pass a single `configuration` object from the consumer `pipeline-configuration-dispatcher.yml` file into the dispatcher (`parameters.configuration`).
3. Optionally override the version of `pipeline-common` by setting `configuration.pipelineCommonRef`.

## Related Repositories
- `wesley-trust/pipeline-common` – shared templates, steps, scripts, and documentation.
- `wesley-trust/pipeline-examples` – end-to-end sample pipelines that demonstrate how to call this dispatcher.
