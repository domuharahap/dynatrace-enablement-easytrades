# 2. EasyTrade Installation

EasyTrade is a fully containerised microservices application that runs entirely on Kubernetes. This section walks you through cloning the repository, deploying all services, and verifying the application is up before you proceed to the use cases.

---

## Step 1 — Clone the EasyTrade repository

```bash
git clone https://github.com/Dynatrace/easytrade.git
cd easytrade
```

The repository contains Kubernetes manifests in the `kubernetes-manifests/` directory and Helm charts in `chart/`.

---

## Step 2 — Create the `easytrade` namespace

```bash
kubectl create namespace easytrade
```

---

## Step 3 — Deploy EasyTrade

### Option A — kubectl (recommended for workshops)

Apply all Kubernetes manifests at once from the repository:

```bash
kubectl apply -f kubernetes-manifests/ -n easytrade
```

This creates Deployments, Services, ConfigMaps, and the flagd (FlagController) deployment for all EasyTrade components.

### Option B — Helm

```bash
helm install easytrade ./chart \
  --namespace easytrade \
  --create-namespace \
  --wait
```

!!! note "Image pull time"
    First deployment pulls ~18 container images. On a slow connection this can take 5–10 minutes. The `--wait` flag in Helm or the watch command below will show progress.

---

## Step 4 — Wait for pods to become ready

```bash
kubectl get pods -n easytrade -w
```

All pods should reach `Running` status. A healthy deployment looks like:

```
NAME                                          READY   STATUS    RESTARTS   AGE
easytrade-aggregator-service-xxxx             1/1     Running   0          4m
easytrade-broker-service-xxxx                 1/1     Running   0          4m
easytrade-calculation-service-xxxx            1/1     Running   0          4m
easytrade-content-creator-xxxx                1/1     Running   0          4m
easytrade-credit-card-order-service-xxxx      1/1     Running   0          4m
easytrade-engine-xxxx                         1/1     Running   0          4m
easytrade-feature-flag-service-xxxx           1/1     Running   0          4m
easytrade-frontend-xxxx                       1/1     Running   0          4m
easytrade-headless-xxxx                       1/1     Running   0          4m
easytrade-login-service-xxxx                  1/1     Running   0          4m
easytrade-manager-xxxx                        1/1     Running   0          4m
easytrade-mssql-xxxx                          1/1     Running   0          4m
easytrade-offer-service-xxxx                  1/1     Running   0          4m
easytrade-pricing-service-xxxx                1/1     Running   0          4m
easytrade-rabbitmq-xxxx                       1/1     Running   0          4m
easytrade-third-party-service-xxxx            1/1     Running   0          4m
nginx-xxxx                                    1/1     Running   0          4m
```

Press `Ctrl+C` once all pods are `Running`.

!!! warning "CrashLoopBackOff on first start"
    `easytrade-mssql` and `easytrade-broker-service` sometimes crash once on first start because the database is not yet initialised. They will recover automatically. If a pod stays in `CrashLoopBackOff` for more than 5 minutes, check its logs: `kubectl logs -n easytrade <pod-name>`.

---

## Step 5 — Feature Flag service (flagd)

EasyTrade uses [OpenFeature](https://openfeature.dev/){target=_blank} with **flagd** as the backend. The `easytrade-feature-flag-service` deployment is already included in the manifests you applied above.

The flag service exposes a REST API on port **8080** inside the cluster and a UI accessible through the EasyTrade frontend (wrench icon in the navbar).

Verify the flag service is running:

```bash
kubectl get svc easytrade-feature-flag-service -n easytrade
```

---

## Step 6 — Get the EasyTrade external URL

### If using a LoadBalancer service (cloud clusters)

```bash
kubectl get svc nginx -n easytrade
```

The `EXTERNAL-IP` column shows the URL once provisioned (may take 1–2 minutes on cloud providers).

```
NAME    TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
nginx   LoadBalancer   10.100.x.xxx   34.xx.xx.xx      80:30080/TCP   5m
```

Set an environment variable for convenience:

```bash
export EASYTRADE_URL="http://$(kubectl get svc nginx -n easytrade -o jsonpath='{.status.loadBalancer.ingress[0].ip}')"
echo $EASYTRADE_URL
```

### If using port-forward (local clusters)

```bash
kubectl port-forward svc/nginx 8080:80 -n easytrade
```

Then open `http://localhost:8080` in your browser.

---

## Step 7 — Verify EasyTrade in the browser

Open `$EASYTRADE_URL` in your browser. You should see the EasyTrade trading dashboard with:

- A header showing the EasyTrade logo and navigation
- A list of available trading instruments
- A **wrench icon** (🔧) in the top-right of the navbar — this opens the Feature Flags panel

!!! tip "Feature Flags panel"
    The wrench icon is the key tool for injecting failures in this workshop. Click it to open a panel listing all available feature flags. Each flag has a name, description, toggle switch, and Save button. Flags take effect within 30–60 seconds of being enabled.

---

## Step 8 — Confirm Dynatrace sees EasyTrade

1. Open your Dynatrace tenant
2. Navigate to **Infrastructure > Kubernetes > Explorer**
3. Filter by your cluster name
4. Select the `easytrade` namespace
5. Confirm all workloads are shown as `Healthy`

!!! info "Trace data takes a few minutes"
    The headless load generator (`easytrade-headless`) starts generating traffic immediately. Distributed traces will appear in Dynatrace within 2–3 minutes of all pods being ready.

<div class="grid cards" markdown>
- [3. Platform Walkthrough :octicons-arrow-right-24:](platform-walkthrough.md)
</div>
