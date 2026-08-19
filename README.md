# Reusable GitOps Workflows

This repository contains reusable GitHub Actions workflows for validating, rendering, and deploying GitOps ApplicationSet templates.

The workflows are designed to remain workload-agnostic so they can support multiple workload repositories, including application and operator installation use cases.

## Repository Purpose

This repository provides reusable automation for:

- validating ApplicationSet Helm charts;
- resolving deployment environment from branch or explicit input;
- deriving cluster-selection values;
- rendering ApplicationSet manifests;
- optionally deploying rendered manifests to OpenShift GitOps;
- verifying deployment status.

## Architecture Role

This repository is one component of a reusable GitOps architecture.

Its responsibility is to provide reusable GitHub Actions workflows that can be shared by multiple GitOps workload repositories.

The repository is intentionally independent of any specific workload implementation.

Future workload repositories may include:

- ConsoleNotification
- Cert Manager
- Additional platform workloads and operators

The reusable workflows interact with a reusable ApplicationSet template repository, while workload-specific repositories provide the application or operator configuration.

## Workflow Structure

The repository contains four workflows:

### validate-applicationset.yaml

Validates workflow inputs, resolves the target environment, determines the cluster selector, and runs Helm lint.

Default branch mapping:

| Branch | Environment |
|--------|-------------|
| `sbx` | `sbx` |
| `stg` | `stg` |
| `main` | `prd` |

An explicit `environment` input can override the branch-derived value.

### render-applicationset.yaml

Renders the reusable ApplicationSet Helm chart using:

- `clusterSelector.key`
- `clusterSelector.value`
- `environment`

The rendered manifest is uploaded as a GitHub Actions artifact.

### deploy-applicationset.yaml

Downloads the rendered ApplicationSet artifact, authenticates to OpenShift, applies the manifest, and verifies ApplicationSet resources.

### gitops-applicationset-pipeline.yaml

Provides the reusable pipeline entry point that connects validation, rendering, and optional deployment.

## Cluster Selection Model

The reusable workflow resolves an environment and passes it to the ApplicationSet template as a cluster selector.

Example:

```text
branch: sbx
    ↓
environment: sbx
    ↓
gox.li9.com/environment=sbx
    ↓
ApplicationSet cluster generator
    ↓
matching registered clusters



