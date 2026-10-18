# Lab 02 - Configure, perform, and roll back an update

## Question

For Deployment `frontend` in namespace `ckad-rollout`, configure rolling updates with `maxUnavailable: 1` and `maxSurge: 2`. Update image `nginx` from `nginx:1.24-alpine` to `nginx:1.25-alpine`, verify the rollout, and then roll back to the previous revision.

## Expectation

- RollingUpdate values match the question.
- The new image is observed before rollback.
- Rollback restores `nginx:1.24-alpine`.

## Setup

```bash
kubectl apply -f setup.yaml
```

## Answer steps

```bash
kubectl -n ckad-rollout patch deployment frontend --patch-file solution.yaml
kubectl -n ckad-rollout set image deployment/frontend nginx=nginx:1.25-alpine
kubectl -n ckad-rollout rollout status deployment/frontend
kubectl -n ckad-rollout rollout history deployment/frontend
kubectl -n ckad-rollout rollout undo deployment/frontend
kubectl -n ckad-rollout rollout status deployment/frontend
```

## Verification

```bash
kubectl -n ckad-rollout get deployment frontend \
  -o jsonpath='{.spec.strategy.rollingUpdate}{"\n"}{.spec.template.spec.containers[0].image}{"\n"}'
```

Expected final image: `nginx:1.24-alpine`.

## Cleanup

```bash
kubectl delete namespace ckad-rollout
```

