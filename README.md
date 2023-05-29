# Homelab: Kubernetes Public Cluster - Applications

[![Build status](https://img.shields.io/github/actions/workflow/status/muhlba91/homelab-kubernetes-public-applications/pipeline.yml?style=for-the-badge)](https://github.com/muhlba91/homelab-kubernetes-public-applications/actions/workflows/pipeline.yml)
[![License](https://img.shields.io/github/license/muhlba91/homelab-kubernetes-public-applications?style=for-the-badge)](LICENSE.md)

This repository contains applications deployed on the `public-cluster` via [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) using [GitOps](https://opengitops.dev).

---

## Bootstrapping

A Kubernetes cluster needs to be bootstrapped with the [Cilium CNI](https://cilium.io) and ArgoCD with an `Application` pointing to this repository.

For [ksops](https://github.com/viaduct-ai/kustomize-sops) and ArgoCD to decrypt the initial secrets for configuring the [External Secrets Operator](http://external-secrets.io) using [Doppler](http://doppler.com), a [Google Cloud Service Account](https://cloud.google.com/docs/authentication#service-accounts) with access to the correct KMS key needs to be set in the `argocd` namespace. You can check out [`infrastructure/charts/argocd/values.yaml`](infrastructure/charts/argocd/values.yaml) on how this secret is passed to ArgoCD.

ArgoCD will then manage Cilium, itself, and all applications as defined in this repository.

***Attention:*** some applications will be automatically deployed, others not (yet).

---

## ArgoCD App-of-Apps

The repository layout follows ArgoCD's [app-of-apps pattern](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/).

The first ArgoCD `Application` being defined needs to reference [`app-of-apps/values.yaml`](app-of-apps/values.yaml) and the environment specific `values-<ENVIRONMENT>.yaml` files.

These are bootstrapping the main ArgoCD projects and applications, referring to the respective `<PROJECT>/applications/values[-<ENVIRONMENT>].yaml` files:

- [`infrastructure`](#infrastructure): core cluster infrastructure, like Cilium and ArgoCD
- [`core`](#core-applications): core applications, like [cert-manager](http://cert-manager.io) and [traefik](https://traefik.io)
- [`applications`](#user-applications): (user) applications running on the cluster/network

Each of these applications follows the app-of-apps pattern again using subcharts defined in the respective `charts` directory.

### Additional Helm Value Files

In addition to the included `values[-<ENVIRONMENT].yaml` files, ArgoCD `Application`s load additonal Helm value files from an external repository defined with `global.spec.values.repoURL`.

For example, values only defined in the external repository are ingress domains.

## Library Charts

### Applications

To support bootstrapping these app-of-apps `Application`s, the library chart [applications](library/charts/applications) creates the ArgoCD `Project` and `Application` definitions based on the provided values.

---

## Hosted Services

### Infrastructure

The following applications are defined in [`infrastructure/charts`](infrastructure/charts).

- [x] [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) - Manages the applications deployed on the cluster using GitOps.
- [x] [Cilium](https://cilium.io) - Provides the cluster CNI.
- [x] [Descheduler](https://github.com/kubernetes-sigs/descheduler) - Finds pods to be evicted for optimizing node usage.
- [x] [External Secrets Operator](http://external-secrets.io) - Synchronizes secrets from external stores to Kubernetes `Secret` objects.

#### Kustomizations

- [x] [External Secrets Stores](infrastructure/kustomizations/external-secrets-stores) - Deploys the required `ClusterSecretStore`s and Doppler [Service Tokens](https://docs.doppler.com/docs/service-tokens) as Kubernetes `Secret`s.

### Core Applications

The following applications are defined in [`core/charts`](core/charts).

- [x] [cert-manager](https://cert-manager.io) - Certificate management using ACME Let's Encrypt.
- [x] [External DNS with Google Cloud DNS integration](https://github.com/kubernetes-sigs/external-dns) - Creates DNS records in Google Cloud DNS domains for publicly reachable services.
- [x] [Traefik](https://traefik.io) - Exposes Kubernetes `Ingress` resources to the "outside world".

### (User) Applications

The following applications are defined in [`applications/charts`](applications/charts).

- [x] [Authentik](http://goauthentik.io) - Open Source identity provider.

---

## Backup and Restore

No (cluster-wide) backup and restore has been implemented as of yet.

---

## Continuous Integration and Automations

- [GitHub Actions](https://docs.github.com/en/actions) are linting and templating all Helm charts.
- [Renovate Bot](https://github.com/renovatebot/renovate) is updating Helm (sub)charts and used container images in the `values.yaml` files, and GitHub Actions.
