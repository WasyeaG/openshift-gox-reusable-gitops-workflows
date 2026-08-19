# Reusable GitOps Workflow Design

## Goal

Provide reusable GitHub Actions workflows that can validate, render, and deploy a reusable ApplicationSet template for multiple workload repositories.

The workflow must remain workload-agnostic and must not contain ConsoleNotification-specific or Cert Manager-specific logic.

## Responsibilities

The reusable workflow is responsible for:

- Validating required inputs.
- Running Helm validation.
- Rendering the reusable ApplicationSet template.
- Resolving the target environment.
- Providing cluster-selection inputs to the ApplicationSet.
- Authenticating to OpenShift.
- Applying the rendered ApplicationSet.
- Reporting deployment status.

## Cluster Selection Model

The initial prototype derives the target environment from the Git branch when an explicit environment is not supplied.

Branch mapping:

| Branch | Environment |
|--------|-------------|
| sbx | sbx |
| stg | stg |
| main | prd |

The resolved environment is used as the value for the ApplicationSet cluster selector.

Example:

branch: sbx

resolves to:

environment: sbx

which provides:

cluster selector key: gox.li9.com/environment
cluster selector value: sbx

The reusable ApplicationSet template then selects all registered clusters matching that label.

## Explicit Environment Override

The workflow should also support an explicit environment input.

If an environment input is supplied, it takes precedence over branch-derived selection.

This keeps the workflow reusable for callers that do not follow the standard sbx/stg/main branch model.

## Workflow Inputs

Initial reusable workflow inputs:

- applicationset_chart_path
- values_file
- environment
- cluster_selector_key
- apply

Additional inputs may be introduced as the reusable ApplicationSet template is developed.

## Render-Only Mode

The workflow should support rendering and validation without modifying cluster resources.

When apply is disabled:

- validate inputs
- run Helm lint
- render ApplicationSet
- publish or display rendered output
- do not authenticate to OpenShift
- do not apply resources

## Design Principle

The workflow determines the deployment context.

The ApplicationSet template determines which registered clusters match the supplied selector.

The workload repository defines what is deployed.

This separation keeps automation, deployment orchestration, and workload implementation independently reusable.

## ApplicationSet Template Contract

The reusable workflow and reusable ApplicationSet template are separate components with a defined interface.

The workflow supplies the following values to the ApplicationSet Helm chart:

- `clusterSelector.key`
- `clusterSelector.value`
- `environment`

Example:

```yaml
clusterSelector:
  key: gox.li9.com/environment
  value: sbx

environment: sbx

```

The ApplicationSet template is responsible for translating these values into the Argo CD cluster generator selector.

The workflow must not contain workload-specific repository URLs, chart paths, namespaces, or application definitions. Those values belong to the ApplicationSet template inputs and workload configuration.
