# Lab 11 - Network Policy

## Question

Namespace `ckad-network` contains an `app` Deployment, a `database` Deployment, and a `client` Pod. Services `app` and `database` expose the corresponding Deployments on TCP port 80. NetworkPolicies are already configured, but Pod `client` cannot connect to either Service.

Ensure Pod `client` can connect only to `app` and `database` on TCP port 80, with no access to other workloads. Do not modify, delete, or replace any existing NetworkPolicy, and do not create additional NetworkPolicies.

## Expectation

- Existing policies remain unchanged, and no new policies are created.
- Pod `client` can connect to `app` and `database` on TCP port 80, but cannot connect to other workloads.

## Setup

```bash
kubectl apply -f setup.yaml
kubectl -n ckad-network wait --for=condition=Ready pod/client --timeout=90s
kubectl -n ckad-network rollout status deployment/app
kubectl -n ckad-network rollout status deployment/database
```

This lab requires a CNI plugin that enforces NetworkPolicy.

Check the initial connectivity:

```bash
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://app
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://database
```

The workloads use nginx as an HTTP test endpoint; `database` is not a real database server.

## Answer steps

**Limitation:** The requested connectivity cannot be achieved under the current constraints. With only these three policies, labels alone cannot restore access because client egress, including DNS, is denied. The steps below demonstrate this limitation rather than a successful solution.

Inspect all three policies and the Pod labels:

```bash
kubectl -n ckad-network get networkpolicy default-deny-all app-ingress database-ingress -o yaml
kubectl -n ckad-network get pods --show-labels
```

The `default-deny-all` policy selects every Pod with `podSelector: {}` and denies both ingress and egress. NetworkPolicy allow rules are additive, but `app-ingress` and `database-ingress` provide ingress exceptions only. None of the three policies allows client egress.

The two ingress policies require different source labels: `app-ingress` requires `app-access: granted`, while `database-ingress` requires `database-access: granted`. Add both labels to the client without changing its existing `app: client` label to demonstrate that satisfying ingress alone is insufficient:

```bash
kubectl -n ckad-network label pod client app-access=granted database-access=granted
```

## Verification

```bash
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://app
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://database
kubectl -n ckad-network get networkpolicy default-deny-all app-ingress database-ingress -o yaml
```

Both requests should still fail. Each added label makes `client` match the allowed ingress source selector in the corresponding policy, but its outbound traffic remains blocked. All three policies remain unchanged.

To confirm that DNS is not the only blocker, retrieve the Service IPs and replace `<APP_CLUSTER_IP>` and `<DATABASE_CLUSTER_IP>` below with those values:

```bash
kubectl -n ckad-network get service app database -o wide
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://<APP_CLUSTER_IP>
kubectl -n ckad-network exec client -- wget -qO- --timeout=3 http://<DATABASE_CLUSTER_IP>
```

These requests should also fail because client egress is denied, even without DNS lookups.

Restoring connectivity would require policy changes or additional allow policies for client egress to both backends on TCP port 80 and DNS on UDP/TCP port 53. Those changes are intentionally outside this lab's constraints. Reply traffic for allowed connections is implicitly permitted; separate backend egress rules are not needed for replies.

## Cleanup

```bash
kubectl delete namespace ckad-network
```
