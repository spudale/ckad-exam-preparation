# Lab 07 - Create an Ingress

## Question

Deployment and Service `web` already exist in namespace `ckad-ingress` on a kind cluster. Create Ingress `web` using class `nginx`. Route host `ckad.local` and path prefix `/` to Service `web` on port `80`. Use curl to request `http://ckad.local/` through the Ingress controller and show the HTTP status and response body.

## Expectation

- API version is `networking.k8s.io/v1`.
- `ingressClassName` is `nginx`.
- Host, path, path type, backend Service, and port match the question.
- Path is `/` with `pathType: Prefix`; no rewrite annotation is needed.
- Curl returns `HTTP/1.1 200 OK` and an HTML page containing `Welcome to nginx!`.

## Setup

An ingress-nginx controller with IngressClass `nginx` must already be installed in the kind cluster. Check it before starting:

```bash
kubectl get ingressclass nginx
kubectl -n ingress-nginx get pods
```

```bash
kubectl apply -f setup.yaml
kubectl -n ckad-ingress rollout status deployment/web --timeout=120s
```

## Answer steps

```bash
kubectl apply -f solution.yaml
```

Possible imperative start:

```bash
kubectl -n ckad-ingress create ingress web \
  --class=nginx --rule='ckad.local/*=web:80'
```

## Verification

```bash
kubectl -n ckad-ingress get ingress web
kubectl -n ckad-ingress describe ingress web
```

On kind, an Ingress address of `localhost` does not guarantee that the controller's port is exposed to your host. A LoadBalancer Service can also show `<pending>`. Use a port-forward to the **controller Service**, not the backend `web` Service, so the request still tests the Ingress rule.

In one terminal, leave this running:

```bash
kubectl -n ingress-nginx port-forward --address 127.0.0.1 service/ingress-nginx-controller 8080:80
```

In another terminal, run:

```bash
curl --noproxy "*" --fail-with-body -i --resolve ckad.local:8080:127.0.0.1 http://ckad.local:8080/
```

On Windows PowerShell, use `curl.exe` instead of `curl` to avoid the `Invoke-WebRequest` alias:

```powershell
curl.exe --noproxy "*" --fail-with-body -i --resolve ckad.local:8080:127.0.0.1 http://ckad.local:8080/
```

`--resolve` connects to the local forwarded port while preserving the `ckad.local` Host header; no hosts-file change is needed. Port `8080` is only the local access port; the Ingress backend remains Service `web` port `80`.

Expected output (headers and HTML abbreviated):

```text
HTTP/1.1 200 OK
...
<h1>Welcome to nginx!</h1>
...
```

The backend serves its welcome page at `/`, so no path rewrite is needed. If the controller has just reloaded, retry the curl command.

## Cleanup

Stop the port-forward with `Ctrl+C`, then remove the lab resources:

```bash
kubectl delete namespace ckad-ingress
```
