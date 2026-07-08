# multi-tenancy-gitops-infra

The cluster-level foundation for the GitOps deployment: namespaces, RBAC, storage, machine config, security policy, and other cluster-scoped resources, delivered declaratively through Argo CD.

> Part of the **IBM Client Engineering Cloud Pak for Integration production-deployment demo** — the infrastructure layer of a four-repo GitOps split (config → **infra** → services → apps).

## What this is

This repo holds the substrate every workload sits on. It is the **first** layer Argo CD reconciles (earliest sync waves), so that by the time services and applications sync, their namespaces, quotas, identities, storage classes, and security constraints already exist. It is owned by the infra/ops team — the people responsible for the cluster itself, not the products running on it.

Resources are plain Kubernetes YAML or YAML packaged as Helm charts, referenced by Argo CD Applications defined in the config layer.

## What's inside

- **`namespaces/`** — ~35 tenant and product namespaces (`dev`/`staging`/`prod` tiers plus `mq`, `db2`, `cp4a`, `cp4ba`, `odm`, `tools`, `vault`, `cert-manager`, `sealed-secrets`, and more), each with its `Namespace` and, where needed, an `OperatorGroup` to scope operator installs.
- **`serviceaccounts/`** — per-namespace `ServiceAccount` identities (mq, db2, sccm, pem, b2bi, vault, tools, …) that give workloads a stable RBAC subject.
- **`storageclass/` + `storage/`** — `StorageClass` definitions (CP4A gold/silver/bronze tiers) and a Helm chart deploying OpenShift Data Foundation (ODF).
- **`machinesets/` + `machine-configuration/` + `infraconfig/`** — worker node pools, `MachineConfig` (e.g. chrony/NTP), and infra-node component placement.
- **`*-clusterwide/`** (`sccm-`, `scd-`, `pem-b2bi-`, `sfg-b2bi-`) — cluster-scoped `SecurityContextConstraints`, RBAC, and CRs required by the Sterling family (SCCM, Connect:Direct, PEM, B2Bi/SFG).
- **`daemonsets/` + `norootsquash/`** — node-level daemons (global pull-secret sync, NFS idmapd fix).
- **`consolelink/` + `consolenotification/`** — OpenShift web-console links and banners.

## Why it's built this way

- **Separation of duties.** Cluster foundations live in their own repo with their own owners and review gate, kept apart from product configuration and application code. Infra changes don't ride along in an app PR, and vice versa.
- **GitOps auditability.** Every namespace, RBAC grant, SCC, and storage class is declared in Git. The cluster's ground truth is a reviewable history, not a pile of `oc apply` commands — and Argo CD continuously reconciles drift back to what's committed.
- **Promotion as a pull request.** Changing the substrate — a new tenant namespace, a quota bump, a tightened security constraint — is a PR: proposed, reviewed, merged, and auto-rolled out. No console clicking, fully repeatable across clusters.
- **Sync-wave ordering.** Because infra reconciles first, downstream service and app syncs never race a missing namespace or identity. The dependency graph is explicit, not implicit.
- **Reusable, layered building blocks.** Plain YAML and toolkit Helm charts are composed rather than copied, so the same patterns stand up dev, staging, and prod consistently.

## How it fits the bigger picture

This is one of four coordinated repos:

1. **`multi-tenancy-gitops`** — the config/bootstrap layer that installs Argo CD and points it at the other three.
2. **`multi-tenancy-gitops-infra`** *(this repo)* — cluster foundations.
3. **`multi-tenancy-gitops-services`** — operators and shared services that install onto this foundation.
4. **`multi-tenancy-gitops-apps`** — the running workloads.

Product factory repos such as **mq-infra** and **ace-infra** generate the deployment overlays consumed by **-apps**, landing in the namespaces, service accounts, and storage this repo provisions. Together they demonstrate a Cloud Pak for Integration environment stood up entirely from Git.

> **Provenance:** Maintained by IBM Client Engineering (github.com/ibmclientengineering). Derived from cloud-native-toolkit/multi-tenancy-gitops-infra (Apache-2.0); modernized for CP4I 16.x / OpenShift 4.18+ (July 2026).

Maintained by IBM Client Engineering.
