# Kubernetes for platform engineers: reference repo

The finished setup of the [Kubernetes for platform engineers](https://platformengr.com/learn/kubernetes/)
course: one kind cluster shared by three teams, every object declared in Git. You copy from this repo and
compare your work with it. You never push to it.

## What is in here

| Folder | What it holds | Owner |
| --- | --- | --- |
| [cluster/](cluster/) | The kind cluster and the values for Traefik, the ingress controller | Platform team |
| [tenants/](tenants/) | Per team: namespace, quota, defaults, RBAC, network policies | Platform team |
| [apps/](apps/) | Per team: Deployment, Service, Ingress, ConfigMap, disruption budget | Each product team |
| [labs/](labs/) | Pods the labs apply from this clone; never copied into your repo | Course |
| [docs/](docs/) | Secrets Git does not hold, the debugging table, the onboarding checklist | Platform team |

In a company `tenants/` and `apps/` would be separate repositories, or one repository with CODEOWNERS.
Here they are two folders so the course stays about Kubernetes.

## Versions

| What | Version | Where it is pinned |
| --- | --- | --- |
| kind node image (Kubernetes) | v1.37.0 | `cluster/kind.yaml` |
| Pod Security level definitions | v1.37 | `pod-security.kubernetes.io/*-version` labels in `tenants/*/namespace.yaml` |
| Traefik Helm chart | 42.4.0 | `cluster/traefik.yaml`, rendered from `cluster/traefik-values.yaml` (render command in its header) |
| checkout, catalog and payments image | ghcr.io/stefanprodan/podinfo:6.17.0 | `apps/*/deployment.yaml` |
| busybox (labs) | 1.38.0 | course commands |

A version bump is one pull request: the node image and the Pod Security version labels move together,
and the course text changes in the same week.

## What is simplified, and what production looks like

| Here | In production |
| --- | --- |
| One kind cluster on a laptop | Several clusters, nodes in several zones, a managed control plane |
| You run `kubectl apply` by hand | A pull request is merged and a controller (Argo CD) applies it |
| One namespace per team, `-dev` only | One per environment, often on separate clusters |
| Secrets typed in with `kubectl create secret` | A secret store and the External Secrets Operator |
| Users are impersonated with `--as` | People log in through your identity provider (OIDC) |
