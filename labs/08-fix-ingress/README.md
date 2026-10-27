# Lab 08 - Troubleshoot an inaccessible Ingress

## Question

Ingress, Service, and Deployment `catalog` exist in namespace `ckad-ingress-fix`, but `http://catalog.ckad.local/` is inaccessible. Identify and fix the configuration issue.

## Expectation

- Inspect the request path from Ingress to Service to Pod.
- Preserve working resources and change only the incorrect configuration.
- Service `catalog` has at least one endpoint after the fix.

## Setup

```bash
kubectl apply -f setup.yaml
```

## Answer steps

Inspect each layer:

```bash
kubectl -n ckad-ingress-fix describe ingress catalog
kubectl -n ckad-ingress-fix get service catalog -o yaml
kubectl -n ckad-ingress-fix get endpointslice -l kubernetes.io/service-name=catalog
kubectl -n ckad-ingress-fix get pods --show-labels
```

The Service selector uses `app: catalogue`, while the Pod label is `app: catalog`. Correct the Service:

```bash
kubectl -n ckad-ingress-fix patch service catalog --patch-file solution.yaml
```

## Verification

```bash
kubectl -n ckad-ingress-fix get endpointslice \
  -l kubernetes.io/service-name=catalog
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
curl -H 'Host: catalog.ckad.local' http://localhost:8080
```

## Cleanup

```bash
kubectl delete namespace ckad-ingress-fix
```

