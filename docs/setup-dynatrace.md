# 1. Dynatrace Operator & OneAgent

The **Dynatrace Kubernetes Operator** manages the lifecycle of Dynatrace components inside your cluster. In **CloudNative FullStack** mode it automatically injects the OneAgent into every pod, captures distributed traces, metrics, logs, and topology — all without code changes.

---

## Step 1 — Add the Dynatrace Helm repository

```bash
helm repo add dynatrace \
  https://raw.githubusercontent.com/Dynatrace/dynatrace-operator/main/config/helm/repos/stable

helm repo update
```

Verify the repo is available:

```bash
helm search repo dynatrace
```

---

## Step 2 — Create the `dynatrace` namespace

```bash
kubectl create namespace dynatrace
```

---

## Step 3 — Create the API token secret

The Operator needs your Dynatrace API token to register with the tenant and create internal tokens (e.g., for the ActiveGate).

```bash
kubectl -n dynatrace create secret generic dynakube \
  --from-literal="apiToken=${DT_API_TOKEN}"
```

!!! note "Secret name must match the DynaKube CR"
    The secret name `dynakube` must match the `name:` field of the DynaKube custom resource you create in Step 5.

---

## Step 4 — Install the Dynatrace Operator via Helm

```bash
helm install dynatrace-operator dynatrace/dynatrace-operator \
  -n dynatrace \
  --set installCRDs=true \
  --atomic
```

The `--atomic` flag waits for all resources to be ready before returning. This typically takes 60–90 seconds.

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

## Step 5 — Create the DynaKube custom resource

The `DynaKube` CR tells the Operator what to deploy and how to connect to your Dynatrace tenant.

Create a file named `dynakube.yaml`:

```yaml
apiVersion: dynatrace.com/v1beta1
kind: DynaKube
metadata:
  name: dynakube
  namespace: dynatrace
  annotations:
    feature.dynatrace.com/automatic-kubernetes-api-monitoring: "true"
spec:
  apiUrl: "<YOUR_DT_TENANT>/api"  # e.g. https://abc12345.live.dynatrace.com/api

  # Skip TLS verification — set to false for production
  skipCertCheck: false

  # OneAgent — CloudNative FullStack (injects into every pod automatically)
  oneAgent:
    cloudNativeFullStack:
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/master
          operator: Exists
        - effect: NoSchedule
          key: node-role.kubernetes.io/control-plane
          operator: Exists

  # ActiveGate — for Kubernetes API monitoring and routing
  activeGate:
    capabilities:
      - kubernetes-monitoring
      - routing
      - metrics-ingest
    resources:
      requests:
        cpu: 500m
        memory: 512Mi
      limits:
        cpu: 1000m
        memory: 1.5Gi
```

!!! tip "Replace the placeholder"
    Replace `<YOUR_DT_TENANT>` with the value of your `$DT_TENANT` environment variable. Do not include a trailing slash.

Apply the CR:

```bash
# Substitute your tenant URL before applying
sed "s|<YOUR_DT_TENANT>|${DT_TENANT}|g" dynakube.yaml | kubectl apply -f -
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
