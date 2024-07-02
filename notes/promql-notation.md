# PromQL Notation

Prometheus provides query lang called promql that lets users select and aggregate time series data in real time
- result can be graphed, viewed as tabular data, or consumed by external systems via HTTP API

## Data types
in promql, an expression or subexpression can evaluate to one of 4 types

- **instant vector** set of time series containing single sample for each time series, all sharing same timestamp
- **range vector** set of time series containing range of data points over time for each time series
- **scalar** smple numeric floating point value
- **string** simple string value, (UNUSED)

## Literals
Designated by single ', double quotes " or backicks
- can use `\` to escape
- can't escape in backticks

## Float literals
Examples:
- 23
- -2.43
- 3.4e-9
- 0x8f
- -Inf
- NaN

## Time series selectors
Responsible for selecting time series and raw or inferred sample timestamps and values
- t.s. *selectors* are not to be confused with higher level concept of instance and range vector *queries* that can execute the time series *selectors*.

### Instant vector selectors
allow the selection of a set of time series and a single sample value for each at a given point in time. 
- example: `http_requests_total`

Can filter by appending comma-separated list of label matchers in `{}`
- example: `http_request_total{job="prometheus",group="canary"}`

Following label matching operators exist:
- `=`
- `!=`
- `=~` (regex match)
- `!~` (NOT regex-match)

- example: `http_requests_total{environment=~"staging|testing|development",method!="GET"}`

#### label=""
If you specify an empty string it'll return all the timeseries that do not have the specific label set at all.
Example: given this dataset:
```
http_requests_total
http_requests_total{replica="rep-a"}
http_requests_total{replica="rep-b"}
http_requests_total{environment="development"}
```
The query `http_requests_total{environment=""}` would match and return:

```
http_requests_total
http_requests_total{replica="rep-a"}
http_requests_total{replica="rep-b"}
```

#### at least specify name or one label matcher that isn't empty string
Examples:
- `http_requests_total{job=~".*"}`  # Good!
- `{job=~".*"}`                     # Bad!
- `{job=~".+"}`                     # Good!
- `{job=~".*",method="get"}`        # Good!

#### __name__ label
it's the name of the time series
example:
- `http_requests_total` <=> `{__name__="http_requests_total"}`
- `{__name__=~"job:.*"}` selects all metrics that have a name starting with "job:"

#### reserved keywords
metric name must not be one of these:
- `bool`, `on`, `ignoring`, `group_left` and `group_right`

But you can use the `__name__` label
- `{__name__="on"}` # Good!

### Range vector selectors
work just like instant vector literals, except they select a range of samples back form the current instant. 
- append a time duration in square brackets `[]` at the end of a vector selector to specify how far back
- range selector is a "closed interval", i.e. timestamps coinciding with either boundary are still included in the selection

example: 
- `http_requests_total{job="prometheus"}[5m]`
    - select all the values we have recorded within the last 5 minutes for all time series that have the metric name `http_requests_total` and a job label set to `prometheus`

#### Valid time durations
- 5h
- 1h30m
- 5m
- 10s

#### Offset modifier
Allows changing the time offset for individual instant and range vectors in a query

- ex: `http_requests_total offset 5m`
    - returns the value of htp_requests_total 5 minutes in the past relative to the current query evaluation time

- ex: `rate(http_requests_total[5m] offset -1w)`
    - returns the 5 minute rate of http_requests_total 1 week forward in time.

#### @ modifier
changes evaluation time for individual instant and range vectors in a query

- ex: `http_requests_total @ 1609746000`
    - returns the value of `http_requests_total` at 2021-01-04T07:40:00+00:00

needs to follow the selector immediately. I.e. this is correct:
- `sum(http_requests_total{method="GET"} @ 1609746000) // GOOD.`
and this is incorrect
- `sum(http_requests_total{method="GET"}) @ 1609746000 // INVALID.`

