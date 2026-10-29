# Lab 10 - Configure security context and capabilities

## Question

Update Deployment `toolbox` in namespace `ckad-security`. Configure container `toolbox` to run as user ID `1000` and add Linux capabilities `NET_ADMIN` and `SYS_TIME`.

## Expectation

- `runAsUser` is configured at container level.
- Both capabilities appear under `securityContext.capabilities.add`.
- The Deployment rolls out successfully, subject to cluster admission policy.

## Setup

```bash
kubectl apply -f setup.yaml
```

## Answer steps

```bash
kubectl -n ckad-security patch deployment toolbox --patch-file solution.yaml
kubectl -n ckad-security rollout status deployment/toolbox
```

## Verification

```bash
kubectl -n ckad-security get deployment toolbox \
  -o jsonpath='{.spec.template.spec.containers[0].securityContext}{"\n"}'
kubectl -n ckad-security exec deploy/toolbox -- id
```

Restricted clusters may reject added capabilities. In that case, the manifest can still be reviewed, but the cluster's admission policy must not be bypassed.

## Cleanup

```bash
kubectl delete namespace ckad-security
```

