# Prometheus
- collects metrics from http scrape targets
- applications provide `/metrics` endpoint
- rich client library ecosystem
    - allows you to instrument your own code (reveal metrics about your app)
    - and provide the metrics endpoint
- service discovery for cloud native environments
- feature rich query lang PromQL
- own time-series db as backend

## Architecture
Prometheus Server:
- See image

## Terminology
### Service Discovery
- Lots of moving targets
    - autoscaling VMs in the cloud
    - services run in containers, orchestrators shuffle workloads
    - microservices and serverless apps

- `scrape_config`
    - dynamic list of targets

## prometheus operator
- provides k8s native deployment and mgt of prometheus and related monitoring components
- purpose is to simplify and automate the configuration of a prom-based monitoring stack
- includes:
    - k8s custom resources
        - to deploy and manage prom, alertManager, and related components
    - Simplified Deployment Configuration
        - use one CR to configure version, persistence, retention policie, replicas
    - prom target configuration
        - automatically generate monitoring target configurations based on a familiar k8s label queries
        - no need to learn prom specific config language
- "compactation"
    - term used in prometheus to reduce granularity for older metrics
    - e.g. metrics older than a month have a granularity of 1hr scrape interval, vs 30s scrape interval for metrics younger than 1 month

# Alerts with Prometheus
prometheus alert rules:
    - sends alerts to alert manager
        - can be made with prometheusrule CRD

Alert manager
    - Grouping: similar nature, avoiding duplicated alerts
    - inhibition: suppress alerts when others are fired already
    - silences - mute alerts for a given time

Transports
    - SMTP (email), webhook, pushover, SNS
    - Pager (victorOps, PagerDuty, Opsgenie)
    - Chat (Slack, wechat, telegram)

# Tracing
## Traces, like logs
- spans with start/end time
    - can provide logs in spans
- context
- metadata enrichment

## Instrumentation
- code changes by developers
- automated

# Opentelemetry
Specification and framework beyond vendors
- collector + sidecar
- bring your own backend
    - jaeger for Tracdes
    - prometheus for metrics

- client libraries and SDKs
    - manual instrumentation (C++, Go, Etc.)
    - Auto-instrumentation (java)

## Focus: traces
k8s
- components send traces
- app instrumentation with traces
    - Send to opentelemetry with traces
        - store in jaeger
            - visualize, correlate, alert