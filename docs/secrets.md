# Secrets the cluster needs that Git does not hold

Every Secret below must exist in the cluster before the workload that uses it can start. None of them
is in this repo, on purpose: a value committed to Git stays readable in its history for ever.

| Secret | Namespace | Keys | Used by | Who owns the value | Where the value is kept |
| --- | --- | --- | --- | --- | --- |
| `checkout-db` | `checkout-dev` | `password` | Deployment `checkout`, env `DB_PASSWORD` | team-checkout | Typed in by hand with `kubectl create secret` (module 02). In production: a secret store, read by the External Secrets Operator. |

Rebuilding the cluster from this repo is not enough on its own: someone has to create `checkout-db`
again first, or the new checkout pods stay in `CreateContainerConfigError`. Closing that gap takes a
secret store and an operator; the [Argo CD course](https://platformengr.com/learn/argocd/) builds it.
