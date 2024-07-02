# To study
## prometheus configuration 
- data retention
- reload 
- utility CLI
- supported file system
- 

## Alert manager
- alert grouping 
- routing configurations
- best practices
- labels vs annotations 
- templating language? using values of alert labels/annotations in other alert label

## tracing
- terminology

## Prometheus
- writing prometheus exporters -> best practices
- metrics types ✅
    - histogram vs summary metrics
- instant vector (vs other vectors)
- "metrics registry"
- exporters (vs native instrumented apps)
- service discovery methods (`*_sd_configs`)
- metrics naming best practices
- remote writing

## promql
- notation
- `offset` modifier
- `histogram_quantile`
- `min` vs `up` vs `min_over_time`
- binary operators
- `rate` vs `irate`