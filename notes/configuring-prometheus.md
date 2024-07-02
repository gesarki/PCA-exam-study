# Configuring Prometheus

## `<relabel_config>`
Relabeling is a tool to dynamically rewrite the label set of a target **before** it gets scraped
- can have multiple per scrape configuration
- they are applied to the label set of each target in order of appearance in the config file

Before relabeling: 
- `job` is set to the `job_name` of the scrape configuration
- `__address__` is set to the `<host>:<port>` address of the target

After relabeling: (unless set during relabeling)
- `instance` set to the value of `__address__`
- `__scheme__` and `__metricspath__` set to the scheme and metricspath of the target
- `__scrape_interval__` set
- `__scrape_timeout` set

`__meta_*` Labels may be set by the service discovery mechanism that provided the target

**labels starting with ``__`` will be removed from the label set after target relabeling is completed.

Available relable actions:
- `replace`: match `regex` against the concatenated `source_labels`, then set `target_label` to `replacement`, with match group references `${1}`, `${2}`, etc. 

## `<metric_relabel_configs>`
Applied to samples as the **last step before ingestion**. 
- Has the same configuration format and actions as targret relabeling
- does not apply to automatically generated timeseries like `up`

Useful for excluding time series that are too expensive to ingest

## `<alert_relabel_configs>`
Applied to alerts before they are sent to AlertManager
- Same configuration format and actions as target relabeling
- appled after external labels

Useful for ensuring different prometheus servers in HA config with different external labels send identical alerts

