# Grafana API Processor

| Status                   |                       |
|--------------------------|-----------------------|
| Stability                | [alpha]               |
| Supported pipeline types | traces, metrics, logs |
| Distributions            | [grafana]             |

See the [design doc] that explains how this component is going to be used.

## Configuration

- `client`:
    - `endpoint` (default - https://www.grafana.com/api): URL for grafana.com API.
       Use `mock://` as a protocol to use mocked Grafana API.
    - `key`: API key for accessing grafana.com.
    - `timeout` (default - 1m): Timeout for requests to grafana.com API.
- `cache`:
    - `complete_refresh_duration` (default 5h): Duration until instance
       cache is completely refreshed.
    - `incremental_refresh_duration` (default 5m): Duration until instance
       cache is updated with changes.

Example of usage with the routing processor:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        include_metadata: true
      http:
        include_metadata: true

exporters:
  logging/1:
    logLevel: info
  logging/2:
    logLevel: info

processors:
  gcomapi:
    client:
      endpoint: "http://fake:3000"
      key: "fake"
  routing:
    attribute_source: context
    from_attribute: "X-Scope-InstanceURL"
    table:
      - exporters: [ logging/1 ]
        value: "https://tempo-dev-01-dev-us-central-0.grafana.net"
      - exporters: [ logging/2 ]
        value: "https://prometheus-dev-01-dev-us-central-0.grafana.net"

service:
  pipelines:
    traces:
      receivers: [ otlp ]
      processors: [ gcomapi, routing ]
      exporters: [ logging/1 ]
    metrics:
      receivers: [ otlp ]
      processors: [ gcomapi, routing ]
      exporters: [ logging/1 ]
```

[alpha]: https://github.com/open-telemetry/opentelemetry-collector#alpha
[design doc]: https://docs.google.com/document/d/1HsJr5eVH4WOdSSGIeaYRRUGAqx4Bzx-CSna1mspW4a4/edit#heading=h.89ldx0hih690
