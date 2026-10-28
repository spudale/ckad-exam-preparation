# Lab 09 - Create a canary deployment

## Question

Namespace `ckad-canary` may contain no more than ten application Pods. Deployment `storefront` runs five Pods with image `nginx:1.24-alpine`. Scale it to eight Pods and create a canary Deployment named `storefront-canary` with two Pods using image `nginx:1.25-alpine`. Both versions must receive traffic through the existing Service `storefront`, targeting approximately 80% stable traffic and 20% canary traffic.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: storefront
  namespace: ckad-canary
spec:
  replicas: 5
  selector:
    matchLabels:
      app: storefront
  template:
    metadata:
      labels:
        app: storefront
    spec:
      containers:
        - name: storefront
          image: nginx:1.24-alpine
          ports:
            - containerPort: 80
```

## Expectation

- Stable replica count increases from five to eight.
- Canary replica count is two.
- Total application Pods are ten.
- Both Deployments share label `app: storefront`.
- A second label distinguishes stable from canary.
- The Service selector matches both versions.
- With all ten Pods ready, eight stable endpoints and two canary endpoints provide an approximate 80/20 traffic split.

## Setup

```bash
kubectl apply -f setup.yaml
```

## Answer steps

```bash
kubectl apply -f solution.yaml
kubectl -n ckad-canary rollout status deployment/storefront
kubectl -n ckad-canary rollout status deployment/storefront-canary
```

## Verification

```bash
kubectl -n ckad-canary get deployments
kubectl -n ckad-canary get pods -l app=storefront --show-labels
kubectl -n ckad-canary get pods -l app=storefront --no-headers | wc -l
kubectl -n ckad-canary get endpointslice \
  -l kubernetes.io/service-name=storefront
```

The Pod count command should return `10`. The Deployments should show `8/8` stable and `2/2` canary ready replicas, and the Service's EndpointSlices should contain ten ready Pod endpoints.

### Port-forward and curl: connectivity check

In one terminal, run:

```bash
kubectl -n ckad-canary port-forward service/storefront 8080:80
```

In another terminal, check the response and nginx version:

```bash
curl -sS -I http://localhost:8080
```

Expect `HTTP/1.1 200 OK` and either `Server: nginx/1.24.0` (stable) or `Server: nginx/1.25.5` (canary). The patch version can vary with the image tag.

**Port-forwarding cannot validate the traffic split.** Even when targeting a Service, `kubectl port-forward` selects a single Pod and forwards every connection to that Pod. Repeating curl against `localhost:8080` will not load-balance across all ten Pods. Stop the port-forward with `Ctrl+C` when finished.

### Curl through the Service: sample the 80/20 split

Use the following Bash command to run a temporary curl client in the `default` namespace. It calls the Service's cluster DNS name, exercising Service load balancing rather than port-forwarding. Keeping the client outside `ckad-canary` leaves that namespace at ten Pods.

First, ensure both Deployments are ready:

```bash
kubectl -n ckad-canary rollout status deployment/storefront
kubectl -n ckad-canary rollout status deployment/storefront-canary
```

Then send 1,000 requests, opening a new connection for each request and identifying the backend by its nginx `Server` header:

```bash
kubectl -n default run storefront-traffic-check \
  --image=curlimages/curl:8.10.1 \
  --restart=Never --rm -i --command -- sh -c '
    set -eu
    stable=0
    canary=0
    i=0
    while [ "$i" -lt 1000 ]; do
      headers=$(curl --fail --silent --show-error --head \
        --connect-timeout 5 --max-time 10 \
        -H "Connection: close" \
        http://storefront.ckad-canary.svc.cluster.local) || exit 1
      case "$headers" in
        *nginx/1.24.*) stable=$((stable + 1)) ;;
        *nginx/1.25.*) canary=$((canary + 1)) ;;
        *) printf "Unexpected response headers:\n%s\n" "$headers" >&2; exit 1 ;;
      esac
      i=$((i + 1))
    done
    printf "Stable: %d/1000 (%d.%d%%)\n" "$stable" "$((stable / 10))" "$((stable % 10))"
    printf "Canary: %d/1000 (%d.%d%%)\n" "$canary" "$((canary / 10))" "$((canary % 10))"
  '
```

Example output (your counts will vary):

```text
Stable: 803/1000 (80.3%)
Canary: 197/1000 (19.7%)
```


```bash
kubectl -n default delete pod storefront-traffic-check --ignore-not-found
```

## Cleanup

```bash
kubectl delete namespace ckad-canary
```
