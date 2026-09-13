# 5. Use Case 2 — High CPU Usage

## Overview

!!! info "Scenario"
    A performance degradation is reported for the trading platform. BrokerService response times increase significantly. On Kubernetes, a CPU resource limit change is also applied by the feature flag, compounding the degradation with CPU throttling.

In this use case you will observe how Dynatrace correlates infrastructure-level CPU metrics, Kubernetes resource limit changes, and application-level response time degradation into a single cohesive problem — without any manual correlation.

**Feature flag:** `high_cpu_usage`
**Root cause:** CPU-intensive processing loop injected in BrokerService + Kubernetes CPU limit reduced from `600m` to `300m`

---

## Step 1 — Record the baseline

Before injecting the failure, record the healthy state of BrokerService.

1. In Dynatrace, navigate to **Application & Microservices > Services > Explorer**.
2. Filter by `k8s.namespace.name = easytrade`.
3. Select **BrokerService**.
4. Note the following values (healthy baseline):

| Metric | Healthy baseline |
|---|---|
| Response time (p90) | ~6 s |
| Failure rate | 0% |
| Throughput | ~60 requests/min |

!!! note "Why 6 s response time at baseline?"
    BrokerService performs real financial calculations. A 6 s p90 is normal for this service at baseline. Watch for the response time climbing to 40 s+ after the flag is enabled.

---

## Step 2 — Inject the failure

1. Open the EasyTrade UI in your browser (`$EASYTRADE_URL`).
2. Click the **wrench icon** (🔧) in the top-right navigation bar.
3. Find the flag named **"K8s: high CPU usage"** (flag ID: `high_cpu_usage`).
4. Toggle the switch to **Enabled**.
5. Click **Save**.
6. Wait **2–3 minutes** for the CPU spike and Kubernetes resource limit change to propagate.

!!! tip "Alternative — curl"
    ```bash
    curl -X PUT "${EASYTRADE_URL}/feature-flag-service/v1/flags/high_cpu_usage" \
      -H "Content-Type: application/json" \
      -d '{"enabled": true}'
    ```

---

## Step 3 — Detect the impact in Services Explorer

1. Navigate back to **Services > Explorer > BrokerService**.
2. Click the **Response time** tab.
3. Observe the response time **spiking to 40 s or more** — a dramatic increase from the 6 s baseline.
4. Click the **Failure rate** tab — brief failure spikes may appear as the service becomes saturated.
5. Scroll down to the **Deployment events** section (or click the **Events** tab). You will see:
    ```
    Workload spec change detected
    Field 'cpu' in 'resources/limits' for container 'broker-service'
    changed from '600m' to '300m'
    ```

!!! warning "Double impact"
    Two things happen simultaneously when this flag is enabled:
    1. BrokerService runs a CPU-intensive loop, consuming all available CPU
    2. The Kubernetes CPU limit for the `broker-service` container is reduced from 600m to 300m

    The reduced limit means Kubernetes throttles the container exactly when it needs CPU most, amplifying the degradation.

---

## Step 4 — Infrastructure view — CPU metrics and throttling

1. On the BrokerService detail page, click the **Infrastructure** tab.
2. The infrastructure card shows the pod: **`easytrade-broker-service`** running in the `easytrade` namespace.
3. Observe the **CPU usage** chart:
    - Usage climbs to **300+ mcore** (exceeding the new 300 m limit)
    - Kubernetes throttles the container as soon as usage hits the limit
4. Observe the **CPU throttling** chart:
    - Throttling percentage increases steeply after the limit change takes effect
5. Hover over the **event annotation** on the chart (a vertical dashed line). The tooltip confirms:
    ```
    Workload spec change — cpu limit: 600m → 300m
    ```

!!! info "What CPU throttling means"
    When a container exceeds its Kubernetes `resources.limits.cpu`, the Linux kernel throttles the process using CFS (Completely Fair Scheduler). The container does not crash, but it runs at reduced speed — causing every request to take longer. This is invisible without infrastructure-level observability.

---

## Step 5 — Logs — correlate with problems

1. Still on BrokerService, click the **Logs** tab.
2. Set the time range to **Last 5 hours**.
3. Observe the **ERROR** and **WARN** log volume spike correlating with the CPU event.
4. In the **Recommended queries** panel (right side), you will see suggestions such as:
    - `Problems P-260842 "Multiple service problems"`
    - `Problems P-260848 "Response time degradation"`
5. Click one of these recommended queries to run it and see all log entries correlated to that specific problem.

!!! tip "Davis AI and Kubernetes events"
    Dynatrace Davis AI automatically links the Kubernetes `WorkloadSpecChange` event to the observed performance degradation. This means the problem card will show the CPU limit change as a contributing factor, even though it originated as a Kubernetes resource update rather than an application error.

---

## Remediation

Disable the feature flag to stop the CPU injection and restore the original CPU limit:

=== "Feature flags UI"
    Open EasyTrade > wrench icon > Feature Flags > K8s: high CPU usage > toggle to **Disabled** > Save.

=== "curl"
    ```bash
    curl -X PUT "${EASYTRADE_URL}/feature-flag-service/v1/flags/high_cpu_usage" \
      -H "Content-Type: application/json" \
      -d '{"enabled": false}'
    ```

The Kubernetes CPU limit will be restored to `600m` within 30–60 seconds of disabling the flag. Response times should return to baseline within 3–5 minutes.

<div class="grid cards" markdown>
- [6. Use Case 3 — Dashboarding :octicons-arrow-right-24:](usecase3-dashboarding.md)
</div>
