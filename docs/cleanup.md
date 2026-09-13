# Cleanup

Once you have completed all workshop use cases, follow these steps to remove all resources from your cluster and avoid unnecessary costs.

---

## Step 1 — Disable all feature flags

Before deleting namespaces, disable any active feature flags so the EasyTrade app is in a clean state (this also ensures no lingering Kubernetes resource mutations).

=== "Feature flags UI"
    Open EasyTrade > wrench icon > Feature Flags.
    For each enabled flag, toggle to **Disabled** and click **Save**.

=== "curl"
    ```bash
    # Disable DB not responding
    curl -X PUT "${EASYTRADE_URL}/feature-flag-service/v1/flags/db_not_responding" \
      -H "Content-Type: application/json" \
      -d '{"enabled": false}'

    # Disable High CPU usage
    curl -X PUT "${EASYTRADE_URL}/feature-flag-service/v1/flags/high_cpu_usage" \
      -H "Content-Type: application/json" \
      -d '{"enabled": false}'
    ```

---

## Step 2 — Delete the EasyTrade namespace

This removes all EasyTrade workloads, services, ConfigMaps, and PersistentVolumeClaims in a single command:

```bash
kubectl delete namespace easytrade
```

This command blocks until all resources in the namespace are deleted. It may take 30–60 seconds.

---

## Step 3 — Uninstall the Dynatrace Operator (Helm)

```bash
helm uninstall dynatrace-operator -n dynatrace
```

This removes the Operator Deployment, CRDs, and RBAC resources created by the Helm chart.

---

## Step 4 — Delete the Dynatrace namespace

```bash
kubectl delete namespace dynatrace
```

This removes the ActiveGate StatefulSet, OneAgent DaemonSet, and all Dynatrace-related secrets.

---

## Step 5 — Remove any remaining PersistentVolumeClaims

Some storage providers retain PVCs even after namespace deletion. Check for orphaned volumes:

```bash
kubectl get pvc --all-namespaces
kubectl get pv
```

Delete any remaining PVCs manually:

```bash
kubectl delete pvc <pvc-name> -n <namespace>
kubectl delete pv <pv-name>
```

---

## Step 6 — (Optional) Delete the cluster

If you created a dedicated cluster for this workshop, delete it to avoid cloud compute charges.

=== "GKE"
    ```bash
    gcloud container clusters delete <CLUSTER_NAME> --zone <ZONE>
    ```

=== "EKS"
    ```bash
    eksctl delete cluster --name <CLUSTER_NAME> --region <REGION>
    ```

=== "AKS"
    ```bash
    az aks delete --name <CLUSTER_NAME> --resource-group <RESOURCE_GROUP>
    ```

---

## Step 7 — Revoke the Dynatrace API token

1. Log in to your Dynatrace tenant.
2. Navigate to **Settings > Access tokens**.
3. Find the token you created for this workshop.
4. Click **Revoke**.

!!! warning "Token cleanup is important"
    API tokens with broad scopes (especially `settings.write`) should be revoked after use. Do not leave workshop tokens active in production tenants.

---

!!! tip "Verification"
    After cleanup, run the following to confirm no EasyTrade or Dynatrace resources remain:
    ```bash
    kubectl get namespaces
    kubectl get pv
    ```
    Neither `easytrade` nor `dynatrace` should appear in the namespace list.

<div class="grid cards" markdown>
- [References :octicons-arrow-right-24:](references.md)
</div>
