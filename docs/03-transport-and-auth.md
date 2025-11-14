# 03 · Transport & Authentication

- All HTTPS endpoints MUST enforce TLS 1.3 with modern cipher suites.
- Requests carry an `Authorization: AIP-HMAC keyId=...,signature=...` header where the signature covers the canonical JSON body plus critical headers.
- Clock drift of ±2 minutes is tolerated; anything outside that window is rejected with `401 DRIFT`.
- Long-lived credentials must be rotated every 90 days and scoped to the minimal set of operations.

## Publish/Subscribe Distribution

The asynchronous auction workflow relies on a cloud-agnostic publish/subscribe transport. Google Pub/Sub is the reference implementation for v1.0, but operators may substitute AWS SNS/SQS, Azure Event Grid, Kafka, or any comparable message bus provided it preserves message ordering within category pools, enforces authenticated publishing, and delivers contexts within the configured auction window.
