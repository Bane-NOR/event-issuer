# Tracing

Event Issuer uses [Open Telemetry](https://opentelemetry.io/) and will deliver a tracing value for every outgoing event.

The standardized way to transmit tracing values is by following the [W3](https://www.w3.org/TR/trace-context/) standard by using:

- `traceparent` describes the position of the incoming request in its trace graph in a portable, fixed-length format. Its design focuses on fast parsing. Every tracing tool MUST properly set `traceparent` even when it only relies on vendor-specific information in `tracestate`
- `tracestate` extends `traceparent` with vendor-specific data represented by a set of name/value pairs. Storing information in `tracestate` is optional.
