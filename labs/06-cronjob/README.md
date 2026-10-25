# Lab 06 - Create a CronJob

## Question

In namespace `ckad-cron`, create CronJob `half-hour-report` that runs every 30 minutes. It must use container name `sp`, image `busybox:1.28`, and restart policy `Never`. Run `/bin/sh -c 'date; echo Hello from the Kubernetes cluster'`.

Set each Job's `activeDeadlineSeconds` to `3`. Retain up to `3` successful Jobs and `1` failed Job using `successfulJobsHistoryLimit` and `failedJobsHistoryLimit`.

Create a Job named `manual-test` from CronJob `half-hour-report` to run it immediately.

## Expectation

- Schedule is `*/30 * * * *`.
- The CronJob uses `batch/v1`.
- The Job template uses container `sp`, image `busybox:1.28`, command `["/bin/sh", "-c", "date; echo Hello from the Kubernetes cluster"]`, and restart policy `Never`.
- Job deadline is set at `spec.jobTemplate.spec.activeDeadlineSeconds: 3`.
- History limits are set at `spec.successfulJobsHistoryLimit: 3` and `spec.failedJobsHistoryLimit: 1`.
- Job `manual-test` is created from the CronJob's Job template.

## Setup

```bash
kubectl create namespace ckad-cron
```

## Answer steps

```bash
kubectl apply -f solution.yaml
```

Imperative starting point:

```bash
kubectl -n ckad-cron create cronjob half-hour-report \
  --image=busybox:1.28 --schedule='*/30 * * * *' \
  -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
kubectl -n ckad-cron patch cronjob half-hour-report --type=merge \
  -p '{"spec":{"successfulJobsHistoryLimit":3,"failedJobsHistoryLimit":1,"jobTemplate":{"spec":{"activeDeadlineSeconds":3,"template":{"spec":{"restartPolicy":"Never","containers":[{"name":"sp","image":"busybox:1.28","command":["/bin/sh","-c","date; echo Hello from the Kubernetes cluster"]}]}}}}}}'
```

After creating the CronJob using either approach, create the immediate Job:

```bash
kubectl -n ckad-cron create job --from=cronjob/half-hour-report manual-test
```

## Verification

```bash
kubectl -n ckad-cron get cronjob half-hour-report
kubectl -n ckad-cron get cronjob half-hour-report \
  -o jsonpath='{.spec.schedule}{"\n"}'
kubectl -n ckad-cron get cronjob half-hour-report \
  -o jsonpath='{.spec.jobTemplate.spec.activeDeadlineSeconds}{"\n"}{.spec.successfulJobsHistoryLimit}{"\n"}{.spec.failedJobsHistoryLimit}{"\n"}'
```

Verify the immediate Job:

```bash
kubectl -n ckad-cron get job manual-test
kubectl -n ckad-cron wait --for=condition=complete job/manual-test --timeout=90s
kubectl -n ckad-cron logs job/manual-test
```

The Job inherits the 3-second deadline, so slow Pod startup can cause it to fail with `DeadlineExceeded`.

## Cleanup

```bash
kubectl delete namespace ckad-cron
```
