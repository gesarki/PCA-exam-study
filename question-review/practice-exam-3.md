# Practice exam 3 - incorrect questions review
1. Which of the following functions is NOT appropriate for use with counter metrics?

Wrong answers: `resets`, `irate`, `increase`
Correct answer: `idelta`

Explanation: the `idelta` function is only appropriate for use with gauge metrics.
> `idelta(v range-vector)` calculates the difference between the last two samples in the range vector `v`, returning an instant vector with the given deltas and equivalent labels

`idelta` and `delta` should only be used with gauges
`deriv` should only be used with gauges

`irate` and `rate` should only be used with counters
`increase` should also only be used with counters

2. Which attribute(s) of an alerting rule allow adding additional metadata? (Select all that apply.)

Correct answers: `labels` and `annotations`
Wrong answers: `metadata`, `meta`, `template`

Explanation: 
- The `labels` clause allows specifying a set of additional labels to be attached to the alert.
- The `annotations` clause specifies a set of informational labels that can be used to store longer additional information such as alert descriptions or runbook links.

3. You are instrumenting an HTTP API with Prometheus metrics. You want to be able to get the rate in requests per day made to your API.

What is the BEST way to accomplish this?

Wrong answer: Define a counter metric called `http_requests_total`, use the PromQL `rate` function to calculate the requests per day

Correct answer: Define a counter metric called `http_requests_total`, use the PromQL `rate` function to calculate the requests per second and multiply by 86400 to get the per-day request rate

Explanation: The PromQL rate function returns the *per-second rate*, not the per-day rate.
Also, it is an instrumentation best practice to expose the most "raw" metric possible and perform desired calculations in PromQL. This increases the flexibility of the metric.

4. What is the difference between the rate and deriv query functions

Wrong answer: `rate` calculates the average rate of change of a counter metric whereas `deriv` calculates the derivative (instantaneous rate of change) of a counter metric

Correct answer: `deriv` operates on gauge metrics while `rate` operates on counter metrics

Explanation: `deriv` and `rate` both return the per-second rate of change of the time series, but rate is just for counters because it calculates the rate of *increase*, while delta is for gauges because it calculates the per-second derivative regardless of if the time series decreases or not

5. How is metrics instrumentation for Prometheus achieved?

Wrong answer: using a prometheus exporter
Correct answer: using a prometheus client library

Explanation: 
- prometheus exporters themselves are instrumented with metrics that are ultimately exposed. They are not used for instrumentation. 
- a prometheus client library for your application code's language lets you define and expose internal metrics via an HTTP endpoint on your application's instance.

6. What is the meaning of the `repeat_interval` attribute in an Alertmanager route?

Wrong answer: How long to wait before sending a notification about new alerts that are added to a group of alerts for which an initial notification has already been sent

Correct answer: How long to wait before sending a notification again if it has already been sent successfully for an alert

Explanation:
- `group_interval` sets how long to wait before sending a notification about new alerts that are added to a group of alerts for which an initial notification has already been sent
- `repeat_interval` sets how long to wait before sending a notification again if it has already been sent successfully for an alert.

alertmanager config file
```yaml
# The root route with all parameters, which are inherited by the child
# routes if they are not overwritten.
route:
  receiver: 'default-receiver'
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  group_by: [cluster, alertname]
  # All alerts that do not match the following child routes
  # will remain at the root node and be dispatched to 'default-receiver'.
  routes:
  # All alerts with service=mysql or service=cassandra
  # are dispatched to the database pager.
  - receiver: 'database-pager'
    group_wait: 10s
    matchers:
    - service=~"mysql|cassandra"
  # All alerts with the team=frontend label match this sub-route.
  # They are grouped by product and environment rather than cluster
  # and alertname.
  - receiver: 'frontend-pager'
    group_by: [product, environment]
    matchers:
    - team="frontend"

  # All alerts with the service=inhouse-service label match this sub-route.
  # the route will be muted during offhours and holidays time intervals.
  # even if it matches, it will continue to the next sub-route
  - receiver: 'dev-pager'
    matchers:
      - service="inhouse-service"
    mute_time_intervals:
      - offhours
      - holidays
    continue: true

    # All alerts with the service=inhouse-service label match this sub-route
    # the route will be active only during offhours and holidays time intervals.
  - receiver: 'on-call-pager'
    matchers:
      - service="inhouse-service"
    active_time_intervals:
      - offhours
      - holidays

```

7. Say you have a dynamic etcd database that contains scrape targets for Prometheus. How should you configure service discovery in this scenario?

Wrong answer: Use the built-in `etcd_sd_configs` scrape attribute.

Correct answer: Write a program that periodically retrieves the targets from etcd and writes them to files according to the static_configs schema. Configure the scrape job to reference those files via `file_sd_configs`.

Explanation: There is no etcd_sd_configs scrape attribute. There is no etcd service discovery built-in to prometheus.

You would need to write a program as described in the correct answer to implement service discovery with etcd.

8. You are instrumenting an HTTP API with Prometheus metrics. You want to define a metric that tracks the latency of HTTP requests made to your API. You desire to arbitrarily break down the latency into percentiles at query time, such as the 90th and 99th.

Which of the following metric names and types is the MOST appropriate for this scenario?

Wrong answer: `http_request_duration_seconds`, Summary

Correct answer: `http_request_duration_seconds`, Histogram

Explanation: 
- in histograms you define buckets based on metrics' *values* beforehand
    - e.g. http_request_duration_seconds: {le=0.1, 0.2, 0.4, ..., +Inf}
    - to calculate the 90th percentile of request durations over the last 10m (i.e. the request duration within which you have served 90% of requests), use:
        - `histogram_quantile(0.9, rate(demo_api_request_duration_seconds_bucket[5m]))`
    - you can set anything to be the quantile when you call the histogram_quantile function

- in summaries you set the quantiles beforehand 
    - when the client is exporting these metrics, it calculates the quantiles on the fly
    - to calculate the 90th percentile of request durations over the last 10m, 0.9 quantile must have been configured beforehand and you can use this:
        - `avg_over_time(demo_api_request_duration_seconds_bucket{instance="prometheus:9090"}[10m])`

9. Which of the following describes the summary metric type?

Wrong answer: Samples numerical observations and counts them in configurable buckets

Correct answer: Samples numerical observations and calculates configurable quantiles over a sliding time window

Explanation: Histograms takes numerical observations and counts them into configurable buckets. Summaries calculates the final quantiles as the observations come in.

10. What is an exemplar?

Wrong answer: The time series with athe maximum value for a metric.

Correct answer: A reference to some external data associated with a time series, commonly a trace

Explanation:
> an exemplar is designed to correlate an external type telemetry data with time series metrics. Typically this external data comes in the form of a trace ID.

> Exemplars are references to data outside of the MetricsSet. A common use case are IDs of program traces.

11. Which Prometheus relabeling configuration can be used to scrape a subset of all discovered targets for a job via modulo division?

Wrong answer: Two `relabel_configs`, one with action `modulus` and the other with action `keep` or `drop`

Correct answer: Two relabel_configs, one with action `hashmod` and the other with action `keep` or `drop`

Explanation: There is no such action `modulus`

> Relabeling is a tool to dynamically rewrite the label set of a target before it gets scraped. One of the actions you cna take is a `hashmod`, which sets `target_label` to the modulus of a hash of the concatenated source labels

12. Which of the following CLI commands can be used for unit testing Prometheus rules?

Wrong answer: promtool check rules

Correct answer: promtool test rules

Explanation: The `check` subcommand is used for checking the syntax validity of rule files, not for unit testing their rules.

13. What is the difference between an agent deployment of Prometheus and a regular deployment?

Wrong answer: The Agent mode optimizes Prometheus for the remote read use case

Correct answer: The Agent mode of Prometheus disables querying, alerting and local storage whereas the normal mode of Prometheus has each of these enabled

Explanation: 
- The Agent mode actually optimizes for the *remote write* use case
- Agent mode disables querying, alerting and local storage
- it can be used as a drop-in replacement for Prometheus if you want to just forward your data to a remote Prometheus server or any other Remote-write compliant project. 

14. Given the following two time series, what is the result of the query given below?

Series:

`up{job="node", instance="node1"} 0 1 1 0 1`
`up{job="node", instance="node2"} 1 0 1 1 1`


Query: `absent(up{job="node", instance="node3"})`

Wrong answer: "{} 1"

Correct answer: "{job="node", instance="node3"} 1"

Explanation: The result should include the labels attached to the metric passed to the absent function
```
absent (
    up{job="node", instance="node3"}
)
```
The metric passed to `absent()` specifies the `job` and `instance` labels.

15. Let http_requests_total be a counter metric representing the number of requests that have hit a given type of web server. Assume there are multiple instances of this type of server. Let this metric have a label called status_code where the values are HTTP status codes (200, 201, 404, etc).

Which of the following queries yields the per-minute rate of requests broken down by status code?

Wrong answer: `rate(http_requests_total[1m]) by (status_code)`

Correct answer: `sum by (status_code) (rate(http_requests_total[5m])) * 60`

Explanation:
- rate doesn't take any `by` arguments, it can't aggregate by anything
- rate returns the per-second average rate of increase of the time series in the range vector
    - it'll return all the rates of all the labels
- summing by (`status_code`) will add up the per-second rates per status code
- multiplying by 60 will get the per-minute rates

16. Let a Node Exporter scrape job be denoted by the label job="node". Assume there are many instances of this scrape.

Which of the following queries yields the average duration of the last scrape across all instances?

Wrong answer: `avg by (instance) (scrape_duration_seconds{job="node"})`

Correct answer: `avg(scrape_duration_seconds{job="node"})`

Explanation:
- `avg by (instance) ...` will get you the average per each instance, which isn't what we want
- `avg(scrape_duration_seconds{job="node"})` will get you the average of all the values in `scrape_duration_seconds{job="node", instance=~".*"}`

17. What's necessary for an HA setup of Alertmanager?

Wrong answer: Configuring alerting clients to send to a single, static instance of Alertmanager

Correct answer: Configuring the instances of Alertmanager to expose port 9094, Running multiple instances of Alertmanager, Configuring the instances of Alertmanager with the addresses of their peers

Explanation: HA for alert manager means setting up multiple instances and having them all communicate with eachother for redundancy in case one instance goes down.

18. Which of the following is NOT a benefit of service discovery

Correct answer: It is easier to tell and alert about a service not being deployed compared to static configuration of targets

Explanation: It is NOT easier to tell and alert about a service not being deployed compared to static configuration of targets. It's *harder*. For static configs, a simpled `up` query can tell you if a given target is down or not deployed at all. With service discovery, if a service is not deployed it will not be discovered, and thus it won't have an `up` metric associated with it. You'd need an `absent` function with labels targeting the specific service. 

19. Which of the following query functions can be used to forecast the future value of a time series?

Correct answer: predict_linear

Explanation: `predict_linear(v range-vector, t scalar)` predicts the value of time series `t` seconds from now, based on the range vector `v`, using simple linear regression. It should only be used with gauges.

20. Where are alerting rules defined?

Wrong answer: In designated files, referenced by the Alertmanager configuration file

Correct answer: In designated files, referenced by the Prometheus configuration file

Explanation: It is actually prometheus that references and evaluates alerting rules, not Alertmanager.

21. What does the Prometheus data model consist of?

Correct answer: Metrics names, metrics labels and samples

Explanation: Prometheus stores all data as *time series*: streams of timestamped values belonging to the same metric and the same set of labeled dimensions.

22. Let cert_expiry be a gauge metric whose value is a Unix timestamp (epoch time) representing the time a given certificate will expire.

Which of the following queries gives the time in seconds until certificate expiration?

Correct answer: cert_expiry – time()

Explanation: `time()` returns the unix timestamp (# of seconds since january 1, 1970)

23. Which of the following components is responsible for evaluating alerting rules?

Wrong answer: Alertmanager

Correct answer: Prometheus

Explanation: 
- prometheus scrapes metrics 
- it stores all scraped samples locally and runs rules over this data to either aggregate and record new time series from existing data or generate alerts

24. How do you specify which targets Prometheus should scrape?

Wrong answer: Globally in the Prometheus configuration file via the `static_configs` and `sd_configs` attributes

Correct answer: On a per-job basis in the Prometheus configuration file via the `static_configs` and `*_sd_configs` attributes

Explanation: Scrapes are configured per job. Each job has a static config or one of the service-discovery mechanisms like `kubernetes_sd_configs`.

25. Which of the following exporters is BEST for monitoring an HTTP web server endpoint to ensure that it returns a 200 status code?

Wrong answer: HTTP Exporter

Correct answer: Blackbox exporter

Explanation: There is no prometheus-maintained exporter called HTTP exporter. Blackbox exporter allows blackbox probing of endpoints over HTTP(S), DNS, TCP, ICMP and gRPC

26. Given the following two time series, and assuming a scrape interval of 1m , what is the result of the query given below?

Series:

up{job="node", instance="node1"} 0 1 1 0 1
up{job="node", instance="node2"} 1 0 1 1 1


Query: avg_over_time(up{job="node"}[5m])

Wrong answer: `{job="node"} 0.7`

Correct answer: 
`{job="node", instance="node1"} 0.6`
`{job="node", instance="node2"} 0.8`

Explanation: Temporal aggregations act over time on each individual input time series, producing the same number of results as inputs

27. How many uniquely-named series are generated by a histogram metric type and what form does it/do they take?

Correct answer: 3 series: `<basename>_bucket`, `<basename>_sum`, and `<basename>_count`

Explanation: From the docs
> A histogram with a base metric name of <basename> exposes multiple time series during a scrape:
- cumulative counters for the observation buckets, exposed as <basename>_bucket{le="<upper_inclusive_bound>"}
- the total sum of all observed values, exposed as `<basename>_sum`
- the count of events that have been observed, exposed as `<basename>_count` (identical to `<basename>_bucket{le="+Inf"}` above)

28. Which of the following is a benefit of the push model of metrics ingestion?

Wrong answer: Push-based monitoring provides a built-in way to determine if a service is down

Correct answer: All monitoring targets can configure the same centralized point of ingestion

Explanation:
- it doesn't provide a built-in way of determining if a service is down. Unless the centralized ingestion point knows exactly which targets should be sending it data, it cannot determine if a service is down based on which ones do not send data during a certain time.
- It is true that all monitoring targets can configure the same centralized point of ingestion in a push-based model