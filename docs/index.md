# Dynatrace Observability Workshop — EasyTrade

Welcome to the **Dynatrace Observability Workshop** built around [EasyTrade](https://github.com/Dynatrace/easytrade){target=_blank}, a realistic microservices trading application designed for hands-on observability practice. Over the course of this workshop you will deploy EasyTrade on Kubernetes, instrument it with Dynatrace OneAgent, and work through three real-world use cases that demonstrate how Dynatrace detects, diagnoses, and explains production problems.

---

## What you will learn

- How to deploy the **Dynatrace Kubernetes Operator** and instrument a cluster with **CloudNative FullStack** monitoring
- How to explore **EasyTrade microservices** observability — topology, services, pods, distributed traces, and logs — from a single pane of glass
- How to perform **root-cause analysis** using distributed traces, service maps, and database query analytics
- How **Davis AI** automatically detects anomalies, correlates impacted services, and produces a causal explanation
- How to build **DQL dashboards** in Dynatrace Grail for infrastructure and application health

---

## Application architecture

EasyTrade is a polyglot microservices application that simulates a financial trading platform. It is deployed entirely on Kubernetes and monitored end-to-end by Dynatrace OneAgent.

### Services and technology stack

| Service | Language / Technology | Role |
|---|---|---|
| **Proxy / Nginx** | Nginx | Reverse proxy and ingress |
| **Frontend** | React (Node.js) | Single-page web application |
| **BrokerService** | .NET Core (C#) | Trade execution and routing |
| **Engine** | Java | Trade matching engine |
| **Manager** | Java | Order lifecycle management |
| **PricingService** | Java | Real-time instrument pricing |
| **LoginService** | .NET Core (C#) | Authentication and sessions |
| **AccountService** | Golang | Account and balance management |
| **OfferService** | Java | Instrument offer catalogue |
| **CreditCardOrderService** | Java | Credit card payment flow |
| **CalculationService** | C++ | Low-level financial calculations |
| **AggregatorService** | Java | Data aggregation layer |
| **ThirdPartyService** | Node.js | Simulated external integrations |
| **ContentCreator** | Python | Background data seeder |
| **Headless load generator** | Node.js | Synthetic load (Playwright) |
| **FlagController** | OpenFeature / flagd | Feature flag management |
| **RabbitMQ** | RabbitMQ | Async messaging between services |
| **MSSQL** | Microsoft SQL Server | Relational database |

### Architecture diagram

```
Browser
  └─► Nginx Proxy
         ├─► React Frontend
         └─► BrokerService (.NET)
                  ├─► Engine (Java)       ──► MSSQL
                  ├─► Manager (Java)      ──► MSSQL
                  ├─► PricingService      ──► RabbitMQ
                  ├─► OfferService
                  ├─► LoginService (.NET)
                  ├─► AccountService (Go)
                  └─► FlagController (flagd)
```

---

## How Dynatrace ingests data

Dynatrace **OneAgent** is deployed as a DaemonSet on every Kubernetes node. It auto-instruments each container without any code changes.

```
OneAgent (DaemonSet on each node)
  │
  ├── Topology discovery    ──► Smartscape / Entity Model
  ├── Distributed traces    ──► PurePath / Grail Trace store
  ├── Metrics               ──► Grail Metrics store
  ├── Logs                  ──► Grail Log store
  └── Business events       ──► Grail BizEvents store
                                        │
                              Dynatrace SaaS Tenant
                              (Notebooks, Dashboards,
                               Davis AI, DQL)
```

Data flows from OneAgent through the **Dynatrace ActiveGate** into **Grail**, Dynatrace's unified data lakehouse. All analysis — including Davis AI root-cause detection and DQL queries — operates directly on Grail-stored data with no data movement required.

---

!!! info "Workshop duration"
    This workshop is designed to be completed in **2–3 hours**. Each section builds on the previous one, so please follow the steps in order.

<div class="grid cards" markdown>
- [Pre-requisites :octicons-arrow-right-24:](pre-requisites.md)
</div>
