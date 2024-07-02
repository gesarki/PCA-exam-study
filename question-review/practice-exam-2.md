    # Practice exam 2 - incorrect questions review
1. Which of the following file systems is NOT recommended/supported by Prometheus?
Answer: NFS

Explanation: Prometheus doesn't recommend or support Network File System (NFS) usage for prometheus storage.
> NFS filesystems (including aws EFS) arenot supported. NFS could be POSIX-compliant, but most implementations aren't. It is strongly recommended to use a local filesystem for reliability

2. Given the following time series and assuming a scrape interval of 1m, what is the result of the query given below?

Series:

up{job="node", instance="node1"} 0 1 1 1 0
up{job="node", instance="node2"} 1 1 1 1 1


Query: sum(up{job="node"}) / count(up{job="node"})

Answer: 1/2 = 0.5 => `{} 0.5`

Explanation: 
- both the sum and count aggregations drop the metric name from the result, so `up` is dropped
- both the sum and count aggregations act over dimentions, so only one result series is produced
- since no labels were specified for the dimentational aggregations sum and count, they aggregate across *all* dimentions. Any labels for dimensions that are not explicitly aggregated across are dropped from the label set of the result.

3. What is the definition of an event?

Wrong answer: "A textual representation of an occurrence, commonly structured with key-value pairs"
Correct answer: "An occurrence of something"

Explanation: "textual representation ..." is the definition of a *log*

4. Which of the following exporters is BEST for monitoring Java Virtual Machine (JVM) metrics?

Incorrect answer: Java exporter
Correct answer: JMX exporter

Explanation: JMX is the officially maintained java exporter and can export from a wide variety of JVM-based apps including Kafka and Cassandra

5. What is a Prometheus annotation?

Wrong answer: metadata used to expose identifying information about targets discovered via various service discovery mechanisms
Answer: metadata that provides longer-form information about an alert

Explanation: The annotations clause specifies a set of informational labels that can be used to store longer additional information such as alert descriptions or runbook links. Annotation values can be templated

6. Which attribute of a Prometheus alerting rule defines the alerting condition?

Wrong answer: `cond`
correct answer: `expr`

explanation: alert rule syntax:
```yaml
# The name of the alert. Must be a valid label value.
alert: <string>

# The PromQL expression to evaluate. Every evaluation cycle this is
# evaluated at the current time, and all resultant time series become
# pending/firing alerts.
expr: <string>

# Alerts are considered firing once they have been returned for this long.
# Alerts which have not yet fired for long enough are considered pending.
[ for: <duration> | default = 0s ]

# How long an alert will continue firing after the condition that triggered it
# has cleared.
[ keep_firing_for: <duration> | default = 0s ]

# Labels to add or overwrite for each alert.
labels:
  [ <labelname>: <tmpl_string> ]

# Annotations to add to each alert.
annotations:
  [ <labelname>: <tmpl_string> ]
```

7. Which of the following queries contains a dimensional aggregation?

Wrong answer: avg_over_time(up[1d])

Correct answer: avg(up)

Explanation:
- dimensional aggregation operators aggregate the elements of a single instant vector, resulting in a new vector of fewer elements with aggregated values
    - e.g.: `prometheus_http_requests_total` returns 4 time series. code=200, 400, 422 and 499. 
    `sum(prometheus_http_requests_total)` returns the sum of all 4 time series in `prometheus_http_requests_total`
- The available dimensional aggregators are:
    - sum
    - min
    - max
    - avg
    - group (all values in the resulting vector are 1)
    - stddev (population standard deviation over dimensions)
    - stdvar (population standard variance over dimensions)
    - count (count number of elements/time series in a vector)
    - count_values("count", http_requests_total) (count how many elements are sending each value)
    - bottomk (5, http_requests_total)
    - topk (5, http_requests_total)
    - quantile(0.95, http_request_seconds_bucket)
- they all allow for the `by (<label>)` or `without (<label>)` keywords

8. Which type of observability data is BEST represented by the following description?

`time="2022-10-18 12:00:00 UTC" level=info msg="User logged in" user=bob`

Wrong answer: Event

Correct answer: Log

Explanation: Although this does represent an event, log is a better answer because log is a textual record of an event often structured with key-value metadata.

9. You want to configure a Blackbox Exporter probe that checks whether or not your servers respond successfully to ping.

What type of probe should you use?

Wrong answer: `TCP`

Correct answer: `ICMP`

Explanation: Ping leverages the Internet Control Message Protocol (ICMP). This is on the layer below TCP. It doens't need ports since it just uses the IP address of the target. For TCP, HTTP, gRPC, they all require a port listening for connections which is not given in the question.
