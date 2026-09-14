# Pre-requisites

Before you start the workshop, verify that each item below is in place. The workshop will not work without a functioning Github Codespaces (buildin with k8s), a valid Dynatrace tenant.

---

## Github with Codespace

You need a Github account and codespace credit to build the Kubernetes cluster
- Any other CNCF-certified Kubernetes distribution

!!! warning "Node requirements"
    The full EasyTrade stack requires a machine type with **4 vCPU / 16 GB RAM** each to run comfortably. Smaller clusters will experience OOMKill and scheduling failures.


---

## Dynatrace SaaS tenant

You need a Dynatrace SaaS tenant. A **15-day free trial** is available at [dynatrace.com/trial](https://www.dynatrace.com/trial/){target=_blank}.

Get the following environment variables ready or save somewhere and keep it handy for the next excercise. They are referenced throughout the workshop:

```bash
DT_TENANT="https://XXXXXXXX.apps.dynatrace.com"   # no trailing slash
DT_OPERATOR_TOKEN="dt0c01.XXXX..."
DT_INGEST_TOKEN="dt0c01.XXXX..."
```

!!! tip "Where to find your tenant URL"
    Your tenant URL is visible in the browser address bar when you are logged in to Dynatrace. It follows the pattern `https://<environment-id>.apps.dynatrace.com`.
    
### Required API token scopes

When creating the `DT_OPERATOR_TOKEN` and `DT_INGEST_TOKEN`  in Dynatrace (**Apps > Access tokens > Generate new token**), enable the following scopes with template:

| Template | Purpose |
|---|---|
| `Kubernetes: Dynatrace Operator` | This is for K8s Monitoring Connection |
| `Kubernetes: Data ingest` | This is for Data Ingestion |


!!! warning "Token security"
    Never commit/safe your API token to a Git repository/public access. Use environment variables or a secrets manager.

---

## Checklist

Before continuing, confirm:

- [ ] `kubectl get nodes` returns at least 3 nodes in `Ready` state
- [ ] `installMkdocs` or `Step by Steps docs` are ready and accessible
- [ ] `Dynatrace Tenant` or `Dynatrace Web UI` are accessible
- [ ] `DT_OPERATOR_TOKEN` and `DT_INGEST_TOKEN` are ready with required scopes
- [ ] Your API token has all required scopes listed above

<div class="grid cards" markdown>
- [1. Dynatrace Operator & Agent :octicons-arrow-right-24:](setup-dynatrace.md)
</div>
