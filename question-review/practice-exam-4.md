# Practice exam 4 - incorrect questions review

1. What determines which labels are included in an alert generated from an alerting rule? (Select all that apply.)


Correct answer: 
- The labels that are present in the PromQL result
- The labels that are attached via the `labels` attribute of the alerting rule

Wrong answer: 
- The annotations that are attached via the `annotations` attribute of the alerting rule

Explanation: The annotations attached via the `annotations` attribute of the alerting is wrong because the question is asking which *labels* are included in the result, not annotations. The labels passed to the alert generated are the ones from the Promql result and the ones attached via the `labels` attribute of the alerting rule.

2. Which of the following is an HTTP header set by Prometheus on each scrape?

Correct answr: X-Prometheus-Scrape-Timeout-Seconds

Explanation:
This header is often used in prometheus exporters (such as the Blackbox exporter) to ensure the exporter completes its work before prometheus times out the scrape.

3. Which of the following can be used to drop specific Prometheus metrics upon ingestion?

Wrong answer: `relabel_configs` with action `drop`

Correct answer: `metric_relabel_configs` with action `drop`

4. You are writing a custom service discovery mechanism for Prometheus called “Windows Discovery” which leverages file_sd_configs. Some of the Windows targets must be scraped via HTTP while others should be scraped via HTTPS.

How should you configure Prometheus and Windows Discovery to scrape these targets appropriately with the fewest number of scrape and relabel configs?

Wrong answer: Configure Windows Discovery to apply label scheme: https to targets that require scraping via HTTPS. Configure two Prometheus scrape jobs, each with a single relabel. One job should drop targets matching scheme: https and the other should keep targets matching scheme: https. The latter job should also specify scheme: https in the job configuration.

Correct answer: Configure Windows Discovery to apply label __scheme__: https to targets that require scraping via HTTPS. Configure a single Prometheus scrape job with no relabels.

Explanation: The wrong method works as intended but is overly complicated. Windows Discovery can be configured to write directly to the __scheme__ label. On scrapes of each target, prometheus will check that label and use the appropriate scheme as secified for that target. 

5. What are the three principal components of the Prometheus exposition format?

Wrong answer: Metric type, metric name, metric value

Correct answer: Metric type, help text, one or more samples

Explanation: Here is the Text exposition format for prometheus:
- line oriented
- within a line, tokens can be separated by one or more space or tab characters
    - leading and trailing whitespace is ignored
- lines starting with `#` are **comments**

- comments starting with `# HELP` or `# TYPE` are **special**
    - `# HELP http_requests_total The total number of HTTP requests.`
        - "The total number of HTTP requests" is considered the docstring of `http_requests_total`
        - help line must appear only once per metric name

    - `# TYPE http_requests_total counter` 
        - the `http_requests_total` metric is a `counter` type
        - can be one of: `counter`, `gauge`, `histogram`, `summary`, or `untyped`
        - type line must appear only once per metric name
        - no TYPE line means the metric is set to `UNTYPED`
    
- remaining lines describe **samples**
- syntax: 
```
metric_name [
  "{" label_name "=" `"` label_value `"` { "," label_name "=" `"` label_value `"` } [ "," ] "}"
] value [ timestamp ]
```
- Examples
    - `http_requests_total{method="post",code="200"} 1027 1395066363000`
        - this has the name, labels, sample, and value
    - `metric_without_timestamp_and_labels 12.47`
    - `http_request_duration_seconds_bucket{le="0.05"} 24054`
    - `rpc_duration_seconds{quantile="0.01"} 3102`
        - these last two have just metric names, labels and values

Therefore there can be multiple values/samples, which is why the first answer is wrong.

5. What is the default web port of Alertmanager?

Wrong answer: 9094

Correct answer: 9093

Explanation: 9094 is the default port for HA AlertManager. 
- 9093 is the default port for prometheus.

6. You are creating an exporter that exposes Prometheus metrics from your fitness tracking app.

Which metric type would be BEST for a metric called run_mileage that tracks both the number of runs you have completed and the total mileage?

Wrong answer: counter

Correct answer: summary

Explanation:
- "Counter" is incorrect as we'd need to track BOTH the number of runs AND the mileage. Single counter can't do this
- a single "Summary" metric can be defined with no quantiles and will expose two metrics: `run_mileage_sum` and `run_mileage_count`. 
- "histogram" works but it's overkill since it would provide `run_mileage_sum` and `run_mileage_count`, but also would require defining buckets which are unnecessary.

7. Which of the following queries will return the currently firing alerts in Prometheus?

Wrong answer: `ALERTS`

Correct answer: `ALERTS{alertstate="firing"}`

Explanation: `Alerts` will return pending OR firing alerts. You must filter by adding the `{alertstate="firing"}` label