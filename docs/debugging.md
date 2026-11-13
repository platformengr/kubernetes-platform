# A pod is not running: find the stage, then read the one place that explains it

Say the stage out loud before you touch anything. Each stage has one place where the reason is written.

| What `kubectl get` shows | Stage | Who refused | Read this | Usual causes |
| --- | --- | --- | --- | --- |
| `READY 1/2` and no second pod at all | Not created | The API server at admission | `kubectl -n NS get events --field-selector reason=FailedCreate` (events of the ReplicaSet) | Quota exceeded, LimitRange max, Pod Security |
| `Pending` | Not scheduled | The scheduler | `kubectl -n NS describe pod POD` (Events: `FailedScheduling`) | Requests bigger than any node, nodeSelector or affinity no node has, taints |
| `ContainerCreating`, `CreateContainerConfigError` | Not started | The kubelet | `kubectl -n NS describe pod POD` (Events) | Missing Secret or ConfigMap, a volume that will not mount |
| `ErrImagePull`, `ImagePullBackOff` | Not started | The registry | `kubectl -n NS describe pod POD` (Events: `Failed`) | Tag does not exist (`not found`), private image without credentials (`unauthorized`), wrong registry host (`no such host`) |
| `CrashLoopBackOff`, `Error` | Crashing | The process | `kubectl -n NS logs POD --previous` and `kubectl -n NS get pod POD -o jsonpath='{.status.containerStatuses[0].lastState}'` | A bad flag or config, `OOMKilled` (exit code 137) |
| `Running`, `0/1`, RESTARTS 0 | Not ready | The readiness probe | `kubectl -n NS describe pod POD` (Events: `Unhealthy`) | The app is not ready, or the probe checks a dependency |
| `Running`, RESTARTS climbing, logs show a clean shutdown | Restarted from outside | The liveness probe | `kubectl -n NS get events --field-selector reason=Killing` | A slow or dependency-checking liveness endpoint, a timeout too short |

Events are kept for about an hour. Look while it is failing.
