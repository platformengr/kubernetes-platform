# Onboarding a new team

What it takes to give a product team a namespace on the shared cluster, in order. Every item has a
command that proves it, and some answers must be `no`.

## Platform team

1. `tenants/<team>/namespace.yaml`: Namespace `<team>-dev`, label `team: <team>`, Pod Security labels
   (`warn` and `audit` first, `enforce: restricted` once the team's pods pass).
   Check: `kubectl label --dry-run=server --overwrite namespace <team>-dev pod-security.kubernetes.io/enforce=restricted` prints no `Warning:`.
2. `tenants/<team>/quota.yaml` and `limits.yaml`: a quota sized for peak replicas plus one pod for a rolling
   update, no NodePort or LoadBalancer Services, and container defaults.
   Check: `kubectl -n <team>-dev describe resourcequota team-quota`.
3. `tenants/<team>/rbac.yaml`: `view` for group `team-<team>` in its namespace only, a `deployer`
   ServiceAccount, and a workload ServiceAccount with no token.
   Check: `kubectl auth can-i list pods -n <team>-dev --as=someone --as-group=team-<team>` gives `yes`;
   `kubectl auth can-i get secrets -n <team>-dev --as=someone --as-group=team-<team>` gives `no`;
   `kubectl auth can-i list pods -n checkout-dev --as=someone --as-group=team-<team>` gives `no`.
4. `tenants/<team>/networkpolicy.yaml`: deny incoming by default, then allow exactly what the team needs;
   restrict egress to DNS and the team's own pods, plus named callees. Remember the other end: a caller's
   egress must allow the call too.
   Check: a `wget -T 3` from inside a pod that must get through, and one that must not.

## Product team

1. `apps/<team>/deployment.yaml`: pinned image, requests and a memory limit, three probes, a spread
   constraint, the `restricted` security context, `serviceAccountName: <team>`.
2. `apps/<team>/configmap.yaml`: versioned name, `immutable: true`.
3. `apps/<team>/service.yaml`, and `ingress.yaml` only if the service is reached from outside.
4. `apps/<team>/pdb.yaml`: `maxUnavailable: 1`.
   Check: `kubectl drain` of a worker finishes and the service keeps answering.
5. Secrets: listed in `docs/secrets.md`, never committed.
