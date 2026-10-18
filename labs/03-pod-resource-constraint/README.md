# Lab 03 - Create a Pod with CPU constraints

## Question

A Pod manifest is provided in `pod.yaml`. Inspect the namespace `ckad-pod-cpu` and set the Pod's container CPU limit to half of the maximum allowed CPU value. Keep the existing CPU request unchanged. Create Pod `worker` in namespace `ckad-pod-cpu` using the updated file.

## Expectation

- You inspect the namespace constraint rather than assuming its value.
- You add only the CPU limit to `pod.yaml`, preserve the existing CPU request, and create the Pod using that file.
- CPU request is `200m`; CPU limit is half of the LimitRange maximum (`500m`).

## Setup

```bash
kubectl apply -f setup.yaml
```

## Answer steps

Inspect the constraint:

```bash
kubectl -n ckad-pod-cpu get limitrange cpu-constraints -o yaml
```

The LimitRange maximum is `1` CPU, so half is `500m`. Add `limits.cpu: 500m` under the existing `resources` section of the `worker` container, leaving `requests.cpu: 200m` unchanged. The resulting section should be (see `solution.yaml` for the complete answer):

```yaml
      resources:
        requests:
          cpu: 200m
        limits:
          cpu: 500m
```

Create the Pod using the updated file:

```bash
kubectl create -f pod.yaml
kubectl -n ckad-pod-cpu wait --for=condition=Ready pod/worker --timeout=90s
```

## Verification

```bash
kubectl -n ckad-pod-cpu get pod worker \
  -o jsonpath='{.spec.containers[0].resources}{"\n"}'
```

## Cleanup

```bash
kubectl delete namespace ckad-pod-cpu
```
