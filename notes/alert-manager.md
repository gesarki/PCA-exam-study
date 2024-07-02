# Alert Manager
## Alerting rules
Alerting rules allow you to define alert conditions based on prometheus language expressions and to send notifications about firing alerts to an external service. Whenever the alert results in one or more vector elements at a given point in time, the alert counts as active for these elements' label sets.

### Defining alerting rules
- configured the same way as recording rules
    - create a file containing the necessary rule statements
    - load the file via the `rule_files` field in the prometheus configuration

An example rules file with an alert:
```yaml
groups:
- name: example
  rules:
  - alert: HighRequestLatency
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High request latency
```
- the optional `for` clause causes Prometheus to wait for the expr to result in 1+ vector elements for each scrape (evaluation) for 10 minutes before firing the alert
    - elements that are active but not firing yet are in `pending` state
- alert rules without the `for` clause will become `active` on the first evaluation

- `labels` clause allows you to add a set of additional labels to be attached to the alert. Any existing conflicting labels will be overwritten. Label values can be templated

- `annotations` specifies a set of informational labels such as descriptions or runbook links. Can also be templated

### Templating
- `$labels` holds the label key/value pairs of an alert instance
    - `{{ $labels.<labelname> }}`
- `$value` holds the evaluated value of an alert instance
    - `{{ $value }}`

example:
```yaml
groups:
- name: example
  rules:

  # Alert for any instances that is unreachable for > 5m
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels: 
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes"

  # Alert for any instance that has a median request latency >1s
  - alert: APIHighRequestLatency
    expr: api_http_request_latencies_second{quantile="0.5"} > 1
    for: 10m
    annotations:
      summary: "High request latency on {{ $labels.instance }}"
      description: "{{ $labels.instance }} has a median request latency above 1s (current: {{ $value }}s)."
```

### Inspectin alerts during runtime
To manually see which alerts are active (firing or pending) go to the "Alerts" tab of prometheus. 
- this will show the exact label sets for which each defined alert is currently active

Prometheus also store synthetic time series of the form:
-  `ALERTS{alertname="<alert name>", alertstate="<pending or firing>", <additional alert labels>}`
