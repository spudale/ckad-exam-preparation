# Lab 04 - Replace a deprecated Deployment API

## Question

Manifest `deprecated-deployment.yaml` uses an API version no longer served by current Kubernetes releases. Update it to a supported API version and deploy it in namespace `ckad-api`.

## Expectation

- API version is `apps/v1`.
- A required selector is present.
- Selector labels match Pod-template labels.
- Two Pods become ready in `ckad-api`.

## Setup

```bash
kubectl apply -f setup.yaml
kubectl apply -f deprecated-deployment.yaml
```

The second command should fail on a modern cluster. That failure is the starting condition.

## Answer steps

Change:

```yaml
apiVersion: extensions/v1beta1
```

to:

```yaml
apiVersion: apps/v1
```

Add `spec.selector.matchLabels` matching the Pod-template label. The corrected file is `solution.yaml`:

```bash
kubectl apply -f solution.yaml
kubectl -n ckad-api rollout status deployment/legacy-web
```

## Verification

```bash
kubectl -n ckad-api get deployment legacy-web
kubectl -n ckad-api get pods -l app=legacy-web
```

## Cleanup

```bash
kubectl delete namespace ckad-api
```

