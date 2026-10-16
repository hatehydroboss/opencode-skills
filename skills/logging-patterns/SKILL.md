---
name: logging-patterns
description: Implement structured logging, choose appropriate log levels, and design observability into applications. Use when adding logging to code, debugging production issues, setting up log aggregation, choosing between log levels, or designing monitoring dashboards. Trigger on "logging", "log levels", "structured logging", "observability", "debugging production", "log aggregation", or "monitoring".
---

# Logging Patterns

Log for debugging and monitoring. Every log line should answer: what happened, to what, and why it matters.

## Log Levels

| Level | When | Example |
|-------|------|---------|
| ERROR | User-affecting failure, needs alerting | Payment failed, DB connection lost |
| WARN | Unexpected but handled, may need attention | Retry succeeded, deprecated API called |
| INFO | Business events, state changes | User logged in, order created |
| DEBUG | Developer troubleshooting detail | Variable values, function entry/exit |
| TRACE | Extreme verbosity, rarely enabled | Every HTTP header, every loop iteration |

**Rule**: If you'd never look at it in production, it's not INFO.

## Structured Logging

Always use structured logs (JSON in production):

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "error",
  "message": "Payment processing failed",
  "service": "checkout-api",
  "trace_id": "abc-123",
  "user_id": "u_456",
  "order_id": "ord_789",
  "error": {
    "type": "GatewayTimeout",
    "message": "Stripe did not respond in 30s"
  }
}
```

### Essential Fields (include on every log)
- `timestamp` (ISO 8601, UTC)
- `level`
- `message`
- `service` / `component`
- `trace_id` / `request_id` (for correlation)

### Contextual Fields (add when relevant)
- `user_id`, `session_id`
- Resource IDs: `order_id`, `account_id`
- Performance: `duration_ms`
- Error details: `error.type`, `error.stack`

## What to Log

- Request entry/exit with timing (INFO for API endpoints)
- State transitions: `order.status changed: pending → confirmed`
- External service calls: success/failure with latency
- Validation failures with the specific field
- Every caught exception (at ERROR or WARN)

## What NOT to Log

- Passwords, tokens, API keys, credit card numbers — ever
- Full request bodies in production (log sizes/schemas instead)
- `print("here")` / `console.log("debug")` — use proper logger
- Log statements in hot loops without rate limiting
- Stack traces for expected errors (404, validation)

## Correlation IDs

Propagate a trace ID through the entire request:

```
Incoming request → generate trace_id → pass to all services → include in every log
```

```python
import contextvars
trace_id_var = contextvars.ContextVar('trace_id')

def get_logger():
    return logger.bind(trace_id=trace_id_var.get())
```

## Sampling

In high-traffic systems, don't log every request at INFO:
- Log 100% of errors and warnings
- Sample INFO logs (1-10%)
- Keep DEBUG off in production (enable per-request via header)

## Monitoring Integration

Logs feed into metrics and alerts:
- Count ERROR logs → alert threshold
- Track log volume anomalies → spike = something broke
- Correlate ERROR rate with latency p99

## Checklist

- [ ] Structured JSON logs in production (human-readable in dev)
- [ ] Trace IDs propagated across service boundaries
- [ ] No secrets or PII in logs (redact sensitive fields)
- [ ] Log levels used consistently across the team
- [ ] Alerting configured on ERROR rate thresholds
- [ ] Log retention policy defined (hot: 7d, warm: 30d, cold: 1yr)
