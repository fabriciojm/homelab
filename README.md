# Homelab

Kubernetes manifests for my homelab, managed with Flux, Kustomize, Helm, SOPS, and age.

The repo is organized around a single `staging` cluster. Flux watches the `main` branch and applies the cluster entrypoints under `clusters/staging`, which then pull in applications, infrastructure controllers, and monitoring.

## What's Included

- **Flux GitOps bootstrap** in `clusters/staging/flux-system`
- **Application manifests** in `apps`
  - `audiobookshelf`: audiobook and podcast server with persistent volumes for config, metadata, audiobooks, and podcasts
  - `linkding`: bookmark manager with persistent storage and a Traefik ingress
- **Infrastructure controllers** in `infrastructure/controllers`
  - `renovate`: hourly Renovate CronJob for dependency updates
- **Monitoring** in `monitoring`
  - `kube-prometheus-stack`: installed through a Flux `HelmRelease`
  - Grafana ingress and TLS secret configuration
- **Secrets management** with SOPS and age
- **Tooling setup** with `mise`

## Repository Layout

```text
apps/
  base/          Reusable app manifests
  staging/       Staging overlays
clusters/
  staging/       Flux entrypoints for the staging cluster
infrastructure/
  controllers/   Cluster services such as Renovate
monitoring/
  controllers/   Monitoring Helm releases and repositories
  configs/       Monitoring-related configuration and secrets
scripts/
  setup.sh       Installs local tools through mise
```

## Local Tools

This repo is set up to work well from a dev container. The included `mise.toml` installs the expected CLI tools and sets the local environment used by the manifests.

Tools managed by `mise`:

- `kubectl`
- `helm`
- `sops`

Run the setup script from the repo root:

```sh
./scripts/setup.sh
```

The `mise.toml` file also sets `KUBECONFIG` and the age public key used for SOPS encryption.

## Flux Entry Points

Flux applies these top-level paths:

- `./apps/staging`
- `./infrastructure/controllers/staging`
- `./monitoring/controllers/staging`
- `./monitoring/configs/staging`

## Secrets

Encrypted Kubernetes secrets are stored in the repo and decrypted by Flux using SOPS with the `sops-age` secret in the cluster. Keep private keys, kubeconfigs, TLS keys, and unencrypted credentials out of commits.

## Updates

Renovate is configured for Kubernetes YAML files and also runs inside the cluster as an hourly CronJob targeting this repository.
