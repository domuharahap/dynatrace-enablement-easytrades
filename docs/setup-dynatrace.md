# 1. Dynatrace Operator & OneAgent

The **Dynatrace Kubernetes Operator** manages the lifecycle of Dynatrace components inside your cluster. In **CloudNative FullStack** mode it automatically injects the OneAgent into every pod, captures distributed traces, metrics, logs, and topology — all without code changes.

We have simplify the Dynatrace agent installation for kubernetes instrumentation, you can refere to documentation here just in case you want know more details: [K8s Manual Intrumentation](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/deployment)

---

## Step 1 — Install the Dynatrace Operator

```bash
dynatraceDeployOperator
```

Verify the Operator pod is running:

```bash
kubectl get pods -n dynatrace
```

Expected output:
```
NAME                                    READY   STATUS    RESTARTS   AGE
dynatrace-operator-xxxx-xxxx            1/1     Running   0          60s
dynatrace-operator-csi-driver-xxxx      2/2     Running   0          60s
```

---

## Step 2 — Installed agent and Create the DynaKube custom resource

Lets deploy agent in the nodes and connect the Kubernetes Tenant to your Dynatrace tenant.


```bash
deployApplicationMonitoring
```

---

## Step 6 — Validate the deployment

### Check the DynaKube status

```bash
kubectl get dynakube -n dynatrace
```

Expected output (after 2–3 minutes):
```
NAME       APIURL                                          STATUS    AGE
dynakube   https://abc12345.live.dynatrace.com/api         Healthy   3m
```

### Check all Dynatrace pods

```bash
kubectl get pods -n dynatrace
```

Expected pods:
```
NAME                                          READY   STATUS    RESTARTS   AGE
dynakube-activegate-0                         1/1     Running   0          3m
dynakube-oneagent-xxxxx   (one per node)      1/1     Running   0          2m
dynatrace-operator-xxxx                       1/1     Running   0          5m
dynatrace-operator-csi-driver-xxxx            2/2     Running   0          5m
```

!!! info "What to expect in Dynatrace"
    Within 5 minutes of the OneAgent pods starting, your cluster will appear in Dynatrace under **Infrastructure > Kubernetes > Explorer**. Node and workload data will populate automatically.

??? tip "Troubleshooting — DynaKube not Healthy"
    If the DynaKube stays in `Progressing` or `Error` state:

    1. Check Operator logs: `kubectl logs -n dynatrace -l app.kubernetes.io/name=dynatrace-operator`
    2. Verify the secret exists: `kubectl get secret dynakube -n dynatrace`
    3. Confirm `apiUrl` is reachable from within the cluster: `kubectl run -it --rm curl --image=curlimages/curl -- curl -k <YOUR_DT_TENANT>/api/v1/time`
    4. Verify your API token has the `activeGateToken.create` scope

<div class="grid cards" markdown>
- [2. EasyTrade Installation :octicons-arrow-right-24:](setup-easytrade.md)
</div>
