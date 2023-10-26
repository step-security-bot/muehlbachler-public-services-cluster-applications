# Personal Public Services - Applications

[![Build status](https://img.shields.io/github/actions/workflow/status/muhlba91/muehlbachler-public-services-cluster-applications/pipeline.yml?style=for-the-badge)](https://github.com/muhlba91/muehlbachler-public-services-cluster-applications/actions/workflows/pipeline.yml)
[![License](https://img.shields.io/github/license/muhlba91/muehlbachler-public-services-cluster-applications?style=for-the-badge)](LICENSE.md)

This repository contains applications deployed on the `public-services-cluster` via [Flux](https://fluxcd.io) using [GitOps](https://opengitops.dev).

---

## Bootstrapping

The Kubernetes cluster needs to be bootstrapped with Flux pointing to this repository.

For [sops](https://github.com/viaduct-ai/kustomize-sops) and Flux to decrypt the initial secrets for configuring the [External Secrets Operator](http://external-secrets.io) using [Doppler](http://doppler.com), a [Google Cloud Service Account](https://cloud.google.com/docs/authentication#service-accounts) with access to the correct KMS key needs to be set in the `flux` namespace.

***Attention:*** some applications will be automatically deployed, others not (yet).

---

## App-of-Apps

The repository follows the app-of-apps pattern.

The first Flux `Kustomization` being defined needs to reference [`app-of-apps/`](app-of-apps/) and the environment specific `<ENVIRONMENT>/` directory.

These are bootstrapping the main Flux applications, referring to the respective `<PROJECT>/applications/` kosutomizations:

- [`infrastructure`](#infrastructure): core cluster infrastructure
- [`core`](#core-applications): core applications
- [`applications`](#user-applications): (user) applications running on the cluster/network

Each of these applications follows the app-of-apps pattern again using sub-kustomizations defined in the respective application directories.

---

## Hosted Services

### Infrastructure

The following applications are defined in [`infrastructure/`](infrastructure/).

- [x] [External Secrets Operator](http://external-secrets.io) - Synchronizes secrets from external stores to Kubernetes `Secret` objects.
  - [x] [External Secrets Stores](infrastructure/external-secrets/) - Deploys the required `ClusterSecretStore`s and Doppler [Service Tokens](https://docs.doppler.com/docs/service-tokens) as Kubernetes `Secret`s.
- [x] [Traefik](https://traefik.io) - Exposes Kubernetes `Ingress` resources to the "outside world".

### Core Applications

The following applications are defined in [`core/`](core/).

- [x] [cert-manager](https://cert-manager.io) - Certificate management using ACME Let's Encrypt.
- [x] [External DNS with Google Cloud DNS integration](https://github.com/kubernetes-sigs/external-dns) - Creates DNS records in Google Cloud DNS domains for publicly reachable services.

### (User) Applications

The following applications are defined in [`applications/`](applications/).

- [x] [Authentik](http://goauthentik.io) - Open Source identity provider.

---

## Backup and Restore

No (cluster-wide) backup and restore has been implemented as of yet.

---

## Continuous Integration and Automations

- [GitHub Actions](https://docs.github.com/en/actions) are linting all YAML files.
- [Renovate Bot](https://github.com/renovatebot/renovate) is updating Helm releases and used container images in the `values.yaml` files, and GitHub Actions.
