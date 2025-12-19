mode: deployment

image:
  repository: otel/opentelemetry-collector-contrib
  tag: "0.98.0"

config:
  receivers:
    otlp:
      protocols:
      grpc: {}
      http: {}

  processors:
    resource:
      attributes:
        - key: service.namespace
          value: platform
          action: upsert

        - key: deployment.environment
          value: local
          action: upsert

        # A PLATAFORMA define o service.name
        - key: service.name
          value: unknown-service
          action: upsert

    batch: {}

  exporters:
    logging:
      loglevel: info

    otlp/tempo:
      endpoint: tempo.observability.svc.cluster.local:4317
      tls:
        insecure: true

    loki:
      endpoint: http://loki.observability.svc.cluster.local:3100/loki/api/v1/push

    prometheusremotewrite:
      endpoint: http://mimir.observability.svc.cluster.local:9009/api/v1/push

  service:
    pipelines:
      traces:
        receivers: [otlp]
        processors: [resource, batch]
        exporters: [otlp/tempo, logging]

      logs:
        receivers: [otlp]
        processors: [resource, batch]
        exporters: [loki]

      metrics:
        receivers: [otlp]
        processors: [resource, batch]
        exporters: [prometheusremotewrite]
