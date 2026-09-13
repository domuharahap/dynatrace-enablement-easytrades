# Pre-requisites

Before you start the workshop, verify that each item below is in place. The workshop will not work without a functioning Kubernetes cluster, a valid Dynatrace tenant, and the required CLI tools.

---

## Kubernetes cluster

You need a Kubernetes cluster running version **1.24 or later**. Any certified distribution works:

- **GKE** (Google Kubernetes Engine)
- **EKS** (Amazon Elastic Kubernetes Service)
- **AKS** (Azure Kubernetes Service)
- **k3s / k3d** for local testing
- Any other CNCF-certified Kubernetes distribution

!!! warning "Node requirements"
    The full EasyTrade stack requires at least **3 nodes** with **4 vCPU / 8 GB RAM** each to run comfortably. Smaller clusters will experience OOMKill and scheduling failures.

---

## kubectl

`kubectl` must be configured and pointing at your target cluster.

```bash
# Verify connectivity
kubectl cluster-info
kubectl get nodes
```

Expected output: node list with `Ready` status.

---

## Helm 3.x

Helm is used to install the Dynatrace Operator.

```bash
# Verify Helm version (must be 3.x)
helm version
```

```bash
# Install Helm if not present (Linux/macOS)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## Dynatrace SaaS tenant

You need a Dynatrace SaaS tenant. A **15-day free trial** is available at [dynatrace.com/trial](https://www.dynatrace.com/trial/){target=_blank}.

Set the following environment variables. They are referenced throughout the workshop:

```bash
export DT_TENANT="https://XXXXXXXX.live.dynatrace.com"   # no trailing slash
export DT_API_TOKEN="dt0c01.XXXX..."
```

!!! tip "Where to find your tenant URL"
    Your tenant URL is visible in the browser address bar when you are logged in to Dynatrace. It follows the pattern `https://<environment-id>.live.dynatrace.com`.

### Required API token scopes

When creating the `DT_API_TOKEN` in Dynatrace (**Settings > Access tokens > Generate new token**), enable the following scopes:

| Scope | Purpose |
|---|---|
| `metrics.ingest` | Push metrics from OneAgent and extensions |
| `logs.ingest` | Push logs from OneAgent |
| `openTelemetryTrace.ingest` | Push traces (OTLP) |
| `entities.read` | Read entity topology |
| `settings.write` | Write Dynatrace Settings (monitoring configs) |
| `settings.read` | Read Dynatrace Settings |
| `activeGateToken.create` | Allow Operator to create ActiveGate tokens |

!!! warning "Token security"
    Never commit your API token to a Git repository. Use environment variables or a secrets manager.

---

## Optional tools

These tools are not strictly required but are useful for debugging and automation:

```bash
# git — clone EasyTrade and this repo
git --version

# curl — test EasyTrade endpoints and feature flag API
curl --version

# jq — parse JSON responses from the API
jq --version
```

---

## Checklist

Before continuing, confirm:

- [ ] `kubectl get nodes` returns at least 3 nodes in `Ready` state
- [ ] `helm version` returns a 3.x version
- [ ] `DT_TENANT` and `DT_API_TOKEN` are exported in your shell
- [ ] Your API token has all required scopes listed above

<div class="grid cards" markdown>
- [1. Dynatrace Operator & Agent :octicons-arrow-right-24:](setup-dynatrace.md)
</div>
