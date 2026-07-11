# Core (non-contrib) Collector: both destinations speak OTLP, so the small
# image is enough. Bump the pin deliberately; check release notes first.
FROM otel/opentelemetry-collector:0.156.0

COPY config/otel-router.yaml /etc/otelcol/config.yaml

EXPOSE 4317 4318
