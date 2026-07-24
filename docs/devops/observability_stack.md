# Observability Stack: Complete Guide

## What is Observability?

**Observability** is the ability to understand the internal state of an application by examining its outputs (logs, metrics, and traces).

**Three Pillars of Observability:**
- **Metrics**: "What is happening?" (Real-time system state)
- **Logs**: "What happened?" (Detailed event history)
- **Traces**: "Why did it happen?" (Request journey through services)

---

## End-to-End Architecture

```
                 ┌────────────────────────────┐
                 │   Node.js Application      │
                 └────────────────────────────┘
                          │
      ┌───────────────────┼───────────────────┐
      │                   │                   │
      ▼                   ▼                   ▼
   Metrics             Traces              Logs
(OpenTelemetry)   (OpenTelemetry)       (Pino)
      │                   │                   │
      ▼                   ▼                   ▼
 Prometheus          OTEL Collector      Fluent Bit
      │                   │                   │
      ▼                   ▼                   ▼
    Mimir              Tempo               Loki
      │
      └───────────────┬───────────────────┐
                      ▼
                   Grafana
            (Metrics + Logs + Traces)
```

---

## Step-by-Step Pipeline

### 1. Application Instrumentation

**OpenTelemetry SDK** instruments your Node.js application to automatically capture:

```javascript
// Automatic tracing of:
- HTTP requests (Express middleware)
- Database queries (PostgreSQL)
- External API calls
- Redis operations
- Message queues (Kafka, RabbitMQ)

// Example: Every incoming request creates a Trace with multiple Spans
GET /users
  └─ Span: Express Middleware
     └─ Span: Authentication
        └─ Span: Database Query (PostgreSQL)
           └─ Span: Cache Lookup (Redis)
```

**Key Benefit:** Every request gets a unique `trace_id` and `span_id` that flows through the entire system.

---

### 2. Metrics Collection

**What are Metrics?**
- Time-series data points (e.g., CPU: 30%, 40%, 45%)
- Real-time health snapshots
- Aggregated over time

**OpenTelemetry Auto-Instrumentation Exposes:**
```
http_requests_total
request_duration_seconds
active_connections
memory_usage_bytes
cpu_usage_percent
db_query_duration_seconds
cache_hit_ratio
```

**Endpoint:** `GET /metrics` (Prometheus format)

**Prometheus** scrapes metrics every 15 seconds and stores them short-term.

---

### 3. Structured Logging with Pino

**Pino** provides structured JSON logging with automatic OpenTelemetry context injection.

**Example Log with OTEL Context:**
```json
{
  "timestamp": "2024-06-30T10:30:45.123Z",
  "level": "INFO",
  "trace_id": "8f4b2a3c5d6e7f8g",
  "span_id": "1a2b3c4d5e6f",
  "request_id": "req-123",
  "userId": 42,
  "message": "User created successfully",
  "http_method": "POST",
  "http_url": "/users"
}
```

**Benefit:** Every log entry belongs to a specific trace, enabling correlation.

---

### 4. Log Collection with Fluent Bit

**Fluent Bit** watches application logs and forwards them to Loki.

**Fluent Bit Responsibilities:**
- Tail container stdout/log files
- Parse JSON logs
- Add Kubernetes metadata (namespace, pod, container)
- Filter and enrich logs
- Batch and forward to Loki

```
Container Logs (stdout)
      │
      ▼
  Fluent Bit
   (Parser, Enricher, Router)
      │
      ▼
   Loki
  (Storage)
```

---

### 5. Log Storage with Loki

**Loki** is a log aggregation system optimized for Kubernetes.

**Key Difference from Elasticsearch:**
- Stores **labels** (searchable metadata) + **compressed logs**
- Lower cost and simpler operation

**Example Query:**
```
{namespace="prod", pod="user-service"} |= "ERROR" | json | duration > 1000
```

**Labels:**
```
namespace=prod
pod=user-service
container=nodejs
environment=production
```

---

### 6. Trace Collection with OpenTelemetry Collector

**OTEL Collector** is the central hub for all telemetry.

**Responsibilities:**
```
┌──────────┐
│ Receiver │ ← Accepts OTLP, Jaeger, Prometheus, Syslog
├──────────┤
│Processor │ ← Batch, filter, transform, sample telemetry
├──────────┤
│ Exporter │ ← Send to Tempo, Jaeger, Prometheus, multiple destinations
└──────────┘
```

**Architecture:**
```
Node App 1 ─┐
Node App 2 ─┼─→ OTEL Collector ─┬─→ Tempo (Traces)
Node App 3 ─┘                    ├─→ Prometheus (Metrics)
                                 └─→ Loki (Logs)
```

**Benefit:** Application doesn't change if you switch from Tempo to Jaeger.

---

### 7. Trace Storage with Tempo

**Tempo** stores distributed traces efficiently.

**Example Trace:**
```
Request ID: 8f4b2a3c5d6e7f8g

Trace (Duration: 250ms)
├─ HTTP Handler (50ms)
├─ Authentication Service (30ms)
│  └─ PostgreSQL Query (20ms)
├─ User Service (80ms)
│  ├─ Cache Check (5ms)
│  └─ PostgreSQL Query (70ms)
├─ Email Service (60ms)
│  └─ SMTP Call (55ms)
└─ Response Serialization (10ms)
```

**Benefit:** Complete request journey visible in one view.

---

### 8. Metrics Storage with Prometheus & Mimir

**Prometheus** scrapes metrics every 15 seconds.

**Limitations of Prometheus alone:**
- Single node storage
- Limited retention (15 days typical)
- No built-in high availability

**Mimir** extends Prometheus:
- Distributed storage (horizontal scaling)
- Long-term retention
- Multi-tenancy support
- High availability

**Flow:**
```
Application (/metrics)
      │
      ▼
  Prometheus (scraper & short-term storage)
      │
      ▼
  Mimir (distributed long-term storage)
      │
      ▼
  Grafana (query & visualize)
```

---

### 9. Visualization & Correlation with Grafana

**Grafana** unifies all data sources.

**Connected Data Sources:**
- **Prometheus/Mimir** → Metrics
- **Loki** → Logs
- **Tempo** → Traces

**Single Dashboard:**
```
┌─────────────────────────────────────┐
│      HTTP Latency Metric             │ ← Prometheus
│      [Graph showing spike at 12:00]  │
└─────────────────────────────────────┘
         │ (Click on spike)
         ▼
┌─────────────────────────────────────┐
│      Related Logs                    │ ← Loki
│      [ERROR] Database timeout 12:00  │
│      [WARN] Connection pool full     │
└─────────────────────────────────────┘
         │ (Click on trace ID)
         ▼
┌─────────────────────────────────────┐
│      Distributed Trace               │ ← Tempo
│      HTTP → DB Query (4.8sec)        │
│      → Timeout → Error Response      │
└─────────────────────────────────────┘
```

---

## Example Incident Investigation

**User Report:** "Login is slow"

**Investigation Steps:**

**1. Check Metrics (Grafana Dashboard)**
```
HTTP Login Endpoint
├─ Requests/sec: 45
├─ p95 Latency: 5000ms (normally 200ms)
└─ Error Rate: 2% (normally 0%)
```

**2. View Related Logs (Filter by trace_id)**
```
[ERROR] PostgreSQL: Connection timeout
[WARN] Authentication service: Slow response
[INFO] Database pool: 50/50 connections in use
```

**3. Open Distributed Trace**
```
GET /login (5000ms total)
├─ Middleware (50ms)
├─ Auth Service (4800ms) ← Bottleneck
│  └─ PostgreSQL Query (4700ms)
│     └─ Timeout waiting for connection
└─ Response (150ms)
```

**Root Cause:** Database connection pool exhausted.

**Fix:** Increase connection pool or investigate long-running queries.

---

## Component Comparison Table

| Component | Purpose | Type | Storage |
|-----------|---------|------|---------|
| **OpenTelemetry SDK** | Instrument application | Library | - |
| **Pino** | Structured logging | Logger | - |
| **Prometheus** | Scrape metrics | Time-series DB | Short-term |
| **Mimir** | Scalable metrics | Distributed DB | Long-term |
| **Fluent Bit** | Collect logs | Log processor | - |
| **Loki** | Store logs | Log storage | Long-term |
| **OTEL Collector** | Central telemetry hub | Pipeline | - |
| **Tempo** | Store traces | Trace storage | Long-term |
| **Grafana** | Visualize & correlate | Visualization | - |

---

## Data Flow Summary

```
1. Application Execution
   ↓
2. OpenTelemetry captures:
   ├─ Traces (HTTP → Service → DB)
   ├─ Metrics (/metrics endpoint)
   └─ Logs (Pino with OTEL context)
   ↓
3. Data Export:
   ├─ Traces → OTEL Collector → Tempo
   ├─ Metrics → Prometheus → Mimir
   └─ Logs → Fluent Bit → Loki
   ↓
4. Grafana connects to all sources
   ↓
5. Correlation via:
   ├─ trace_id (connects logs & traces)
   ├─ timestamps (connects metrics with events)
   └─ Labels (pod, service, environment)
```

---

## Key Benefits

✅ **Rapid Root Cause Analysis:** From alert to diagnosis in seconds

✅ **Correlation:** Automatically linked metrics, logs, and traces

✅ **Scalability:** Mimir for long-term metrics, Tempo for trace history

✅ **Efficiency:** Fluent Bit is lightweight, Loki uses label-based storage

✅ **Flexibility:** OTEL Collector abstracts vendor lock-in

✅ **Real-time Insights:** Prometheus provides immediate visibility

✅ **Debugging:** See complete request journey with trace waterfall

---

