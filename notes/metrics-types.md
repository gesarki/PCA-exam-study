# Metrics Types
- prometheus client offers 4 core metrics types
- only differentiated in the client libraries - to enable APIs tailored to the usage of the specific types -   
    - and in the wire protocol
- prom server doesn't make use of the type info
- prom server flattens all data types into untyped time series
- may change in the future

## Counter
- single monotonically increasing counter
    - value can only increase or reset to zero on restart
- e.g.: # of requests served, # of tasks completed
- do NOT use for value that can decrease. 
    - e.g. DO NOT use for # of currently running processes; instead use a gauge

## Gauge
- single numerical value that can arbitrarily go up or down
- e.g.: temperature, curent mem usage, numebr of concurrent requests

## Histogram
- histogram samples observations (e.g. request durations, response sizes)
    - and counts them in configurable buckets
- also provides sum of all observed values

### Example
- histogram w/ base metric name of `<basename>` exposes multiple time series
    - cumulative counters for the observation buckets, exposed as `<basename>_bucket{le="<upper inclusive bount>"}`
    - total sum of all observed values, exposed as `<basename>_sum`
    - count of all events that have been observed, exposed as `<basename>_count` 
        - identical to `<basename>_bucket{le="+Inf"}`

- use the `histogram_quantile` function to calculate quantiles from histograms or even aggregations. 

## Summary
- similar to histograms, also samples observations like req durations and response times
- calculates configurable quantiles over a sliding window

## Example
- summary with basename `<basename>` exposes multiple times series during a scrape
    - streaming 

## Histogram vs Summary
- both sample observations, typically req durations or resp sizes
- track the number of observations AND the sum of observed values, allowing you to calculate the AVERAGE of the observed values
    - aside: the number of observations (`<basename>_count`) is inherently a counter

- To calculate the Avg request duration during the last 5 minutes from a histogram or summary called `http_request_duration_seconds`, use the followign expression:
```promql
  rate(http_request_duration_seconds_sum)[5m]
/
  rate(http_request_duration_seconds_count)[5m]
```

- You can use both summaries and histograms to calculate so-called φ-quantiles, where 0 ≤ φ ≤ 1. The φ-quantile is the observation value that ranks at number φ*N among the N observations. Examples for φ-quantiles: The 0.5-quantile is known as the median. The 0.95-quantile is the 95th percentile.

- The essential difference between summaries and histograms is that summaries calculate streaming φ-quantiles on the client side and expose them directly, while histograms expose bucketed observation counts and the calculation of quantiles from the buckets of a histogram happens on the server side using the `histogram_quantile()` function.

