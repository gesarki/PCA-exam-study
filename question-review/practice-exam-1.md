# Practice exam 1 - incorrect questions review

1. Which of the following are valid Prometheus label values? (Select all that apply.)

Correct answer: `2nd_label_value`, `label-value-1`, `val`, `$__maybe_valid`

Explanation: Prometheus label values may contain any Unicode characters, which includes special characters like dollar sign. Even emojis are valid for use in Prometheus labels since they are Unicode-encoded.

As long as they don't start with `__`, which is reserved for internal use

2. You are instrumenting an application called agate for Prometheus. This application has a subsystem called remed that handles automated script-based remediations. You want to define a metric that gives the number of remediation processes currently running.

Which of the following is the most appropriate “fully-qualified” metric name?

Wrong answer: `remed_processes_running`, `agate-remed-processes-running`

Correct answer: `agate_remed_processes_running`

Explanation:
- metrics names can't contain hyphens
    - only ASCII letters, digits, underscores, and colons

- should have the fully qualified application and system in the name. 
- also should have suffix describing the unit (only base unit: seconds good, miliseconds bad)

3. You have configured the Elastic Kubernetes Service on AWS and have deployed Node Exporter to each of the hosts and configured it to be managed by systemd.

Which of the following service discovery methods should you use to configure a  Prometheus scrape for Node Exporter?

Wrong answer: `kubernetes_sd_configs`

Right answer: `ec2_sd_configs`

Explanation: This is because node exporter is managed by systemd, so kubernetes service discovery won't find it. You could use it if it were managed by a k8s daemonset.

4. What configuration is used to effectively disable grouping of alerts for a particular Alertmanager route?

Wrong answer: `disable_grouping: true`, `enable_grouping: false`

Correct answer: `group_by: ['...']`

Explanation: To aggregate by all possible labels, use the `...` special value as the sole label name in `group_by`. This effectively disables aggregations entirely, passing through all alerts as is. Maybe wanted if your upstream notification system performs its own grouping

5. Which of the following queries will return the currently firing alerts in Alertmanager?

Wrong answer: `ALERTS{alertstate="firing"}`

Correct answer: No such query exists

Explanation: Prometheus exposes the synthetic ALERTS time series based on the alerts it evaluates but there may be other sources for AlertManager than Prometheus. The only way to see all firing alerts is via AlertManager, not PRomQL

6. Select the following option where the binary query operators decrease from highest to lowest precedence.

Wrong answer: or, and, !=, -, *

Correct answer: ^, *, -, >, and, or

Explanation: This is the order of precedence for binary operators from highest to lowest:
    1. ^
    2. *, /, %, atan2
    3. +, -
    4. == , != , <=, < , >=, >
    5. and, unless
    6. or
Operators on the same level are left-associative

7. Which of the following is NOT a best practice when it comes to writing a Prometheus exporter?

Wrong answer: Drop statistical aggregations such as mean or median

Correct answer: Include a `version` label on all exported metrics

Explanation: It IS a best practice to drop statistical aggregations such as mean or median. It is NOT best practice to include a `version` label. Instead use a designated `build_info` metric with a `version` label and dynamically attach it via PromQL groupings as needed.

8. What is a Prometheus metrics registry?

Wrong answer: A documentation site detailing the metrics provided by a specific exporter or instrumentation

Correct answer: A set of metrics in an instrumented application that will be returned when scraped

Explanation: A metrics registry is a code concept within each prometheus client library where metrics are registered to be returned when scraped.

9. Which of the following is a valid way to trigger a Prometheus configuration reload?

Wrong answer: Sending an HTTP `GET` request to the `/-/reload` endpoint with the `--web.enable-enable-lifecycle` command line flag set

Correct answer: Sending a SIGHUP to the Prometheus process.

Explanation: The `/-/reload` endpoint with the `--web.enable-enable-lifecycle` flag would work if you sent a `POST` or `PUT` command instead, not a `GET` command.

Otherwise, you can trigger a config reload with the SIGHUB signal to the prometheus process.

Additionally, you can shutdown prometheus by sending a `PUT` or `POST` to `/-/quit`. The `--web.enabled-lifecycle` flag must be set.

10. You are instrumenting an HTTP API with Prometheus metrics. You want to define a metric that tracks the latency of HTTP requests made to your API. You only care about the average latency and do not have a need for percentiles.

Which of the following metric names and types is the MOST appropriate for this scenario?

Wrong answer: `http_request_duration_seconds`, Histogram

Correct answer: `http_request_duration_seconds`, Summary

Explanation: Histogram you need to specify a set of buckets to count request durations in, which we don't care about. Summary, if you don't specify quantiles, will just give you `http_request_duration_seconds_sum` and `http_request_duration_seconds_count`, which you can use to get the avg with: 
```
rate(http_request_duration_seconds_sum[5m])
/
rate(http_request_duration_seconds_count[5m])
```

11. Which of the following is NOT a metric generated by Prometheus automatically when performing a scrape?

Wrong answer: `scrape_samples_scraped`

Correct answer: `scrape_payload_bytes`

Explanation: For each instance scraped, prometheus stores a sample in the following time series:
- `up`
- `scrape_duration_seconds`
- `scrape_samples_post_metric_relabeling` (the number of samples remaining after metric relabeling was applied)
- `scrape_samples_scraped` (number of samples the target exposed)
- `scrape_series_added` (approx number of new series in this scrape)

scrape_payload_bytes is not generated by prometheus

12. How and at what level of granularity is Prometheus data retention configured?

Wrong answer: Globally via the configuration file

Correct answer: Globally via command line flags

Explanation: retention is configured globally via the `--storage.tsdb.retention.time` command line flag.