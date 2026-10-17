# Lab 01 - Move hard-coded database values to a Secret

## Question

The `postgres-client` Deployment in namespace `ckad-secret` contains the PostgreSQL database name and password as literal environment variable values. Create Secret `postgres-credentials` with keys `database` and `password`, update the Deployment to consume those keys, and redeploy it.

## Expectation

- No database value or password remains directly under `env[].value`.
- `POSTGRES_DB` references key `database`.
- `POSTGRES_PASSWORD` references key `password`.
- The replacement Pod reaches `Running`.

## Setup

```bash
kubectl apply -f setup.yaml
```

## Answer steps

```bash
kubectl apply -f solution.yaml
```

`kubectl set env --from=secret/...` would use the Secret key names as the environment variable names. Because this task requires different names, edit the Deployment:

```bash
kubectl -n ckad-secret edit deployment postgres-client
```

Replace the two environment entries with:

```yaml
env:
  - name: POSTGRES_DB
    valueFrom:
      secretKeyRef:
        name: postgres-credentials
        key: database
  - name: POSTGRES_PASSWORD
    valueFrom:
      secretKeyRef:
        name: postgres-credentials
        key: password
```

Then wait for the automatic rollout:

```bash
kubectl -n ckad-secret rollout status deployment/postgres-client
```

## Verification

```bash
kubectl -n ckad-secret get deployment postgres-client -o yaml
kubectl -n ckad-secret exec deploy/postgres-client -- sh -c \
  'test "$POSTGRES_DB" = inventory && test "$POSTGRES_PASSWORD" = ckad-password'
```

## Cleanup

```bash
kubectl delete namespace ckad-secret
```
