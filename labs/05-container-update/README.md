# Lab 05 - Update a container name and image

## Question

Update Deployment `api` in namespace `ckad-container`. Rename container `old-container` to `web-container` and set its image to `nginx:1.25-alpine`. Ensure the resulting Pod reaches `Running`.

## Expectation

- The Pod template contains one container named `web-container`.
- Its image is `nginx:1.25-alpine`.
- The rollout completes and the Pod is ready.

## Setup

```bash
kubectl apply -f setup.yaml
```

## Answer steps

Because strategic merge identifies containers by name, patching with a new name can add a second container. Use an explicit JSON patch to replace both fields:

```bash
kubectl -n ckad-container patch deployment api --type=json -p='[
  {"op":"replace","path":"/spec/template/spec/containers/0/name","value":"web-container"},
  {"op":"replace","path":"/spec/template/spec/containers/0/image","value":"nginx:1.25-alpine"}
]'
kubectl -n ckad-container rollout status deployment/api
```

On Windows PowerShell, place the JSON in a file or use `kubectl edit deployment api -n ckad-container` to avoid quoting issues. `solution.yaml` shows the desired Pod-template fragment.

## Verification

```bash
kubectl -n ckad-container get deployment api \
  -o jsonpath='{.spec.template.spec.containers[0].name}{" "}{.spec.template.spec.containers[0].image}{"\n"}'
kubectl -n ckad-container get pods
```

## Cleanup

```bash
kubectl delete namespace ckad-container
```

