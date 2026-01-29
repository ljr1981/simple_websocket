# WS-BENCH - Technical Design

## Architecture

### Component Overview

```
+----------------------------------------------------------+
|                       WS-BENCH                            |
+----------------------------------------------------------+
|  CLI Interface Layer                                      |
|    - BENCH_CLI: Argument parsing, benchmark control       |
|    - BENCH_PROGRESS: Real-time progress display           |
|    - BENCH_REPORTER: Output formatting (text/JSON/CSV)    |
+----------------------------------------------------------+
|  Orchestration Layer                                      |
|    - BENCH_COORDINATOR: Manage worker pool                |
|    - BENCH_SCENARIO: Define test scenarios                |
|    - BENCH_RAMP: Connection ramp-up control               |
+----------------------------------------------------------+
|  Worker Layer                                             |
|    - BENCH_WORKER: Individual connection handler          |
|    - BENCH_TIMER: High-precision timing                   |
|    - BENCH_SAMPLER: Latency sample collection             |
+----------------------------------------------------------+
|  Statistics Layer                                         |
|    - BENCH_STATS: Aggregate statistics                    |
|    - BENCH_HISTOGRAM: Latency distribution                |
|    - BENCH_PERCENTILE: Percentile calculations            |
+----------------------------------------------------------+
|  Protocol Layer                                           |
|    - WS_FRAME: Frame encoding/decoding                    |
|    - WS_HANDSHAKE: HTTP upgrade handshake                 |
|    - WS_FRAME_PARSER: Streaming frame parser              |
+----------------------------------------------------------+
```

### Class Design

| Class | Responsibility | Key Features |
|-------|----------------|--------------|
| BENCH_CLI | Command-line interface | parse_args, run_benchmark, show_help |
| BENCH_COORDINATOR | Orchestrate workers | create_workers, start_all, collect_stats |
| BENCH_WORKER | Individual connection | connect, send_message, receive_message, measure_latency |
| BENCH_SCENARIO | Test definition | load_scenario, get_next_action |
| BENCH_RAMP | Ramp-up control | calculate_delay, connections_at_time |
| BENCH_TIMER | High-precision timing | start, stop, elapsed_microseconds |
| BENCH_SAMPLER | Sample collection | add_sample, get_samples |
| BENCH_STATS | Aggregate statistics | add_measurement, calculate_stats |
| BENCH_HISTOGRAM | Latency distribution | add_value, get_bucket, get_percentile |
| BENCH_REPORTER | Output formatting | format_text, format_json, format_csv |
| BENCH_PROGRESS | Progress display | update, show_rate, show_eta |

### Command Structure

```bash
ws-bench <url> [options]

Arguments:
  url                   WebSocket URL to benchmark (ws:// or wss://)

Connection Options:
  -c, --connections N   Number of concurrent connections (default: 1)
  --ramp-up SECONDS     Time to ramp up to full connections (default: 0)
  --connect-timeout MS  Connection timeout in milliseconds (default: 5000)

Message Options:
  -m, --messages N      Messages per connection (default: 100)
  -r, --rate N          Max messages per second (0 = unlimited)
  -d, --data TEXT       Message payload (default: "benchmark")
  --data-file FILE      Load payload from file
  --binary              Send binary messages

Duration Options:
  -t, --time SECONDS    Run for specified duration (overrides -m)
  --warmup SECONDS      Warmup period before measurement (default: 0)

Output Options:
  -o, --output FORMAT   Output format: text (default), json, csv
  --histogram           Show latency histogram
  --percentiles LIST    Percentiles to calculate (default: 50,90,95,99)
  --interval SECONDS    Progress update interval (default: 1)
  --quiet               Suppress progress output
  --report FILE         Write report to file

Other Options:
  --scenario FILE       Load test scenario from file
  --header KEY:VALUE    Add custom HTTP header (repeatable)
  --subprotocol NAME    Request WebSocket subprotocol
  --echo                Measure echo round-trip time
  --help                Show help message
  --version             Show version information
```

### Data Flow

```
CLI Arguments
     |
     v
+------------------+
| BENCH_CLI        |  Parse arguments, validate
+------------------+
     |
     v
+------------------+
| BENCH_COORDINATOR|  Create worker pool
+------------------+
     |
     +----> BENCH_RAMP: Calculate ramp-up schedule
     |
     v
+------------------+
| BENCH_WORKER(s)  |  Concurrent execution
+------------------+
     |
     +---> WS_HANDSHAKE: Connect
     |
     +---> BENCH_TIMER: Start timing
     |
     +---> WS_FRAME: Send message
     |
     +---> WS_FRAME_PARSER: Receive response
     |
     +---> BENCH_TIMER: Stop timing
     |
     +---> BENCH_SAMPLER: Record sample
     |
     v
+------------------+
| BENCH_STATS      |  Aggregate results
| BENCH_HISTOGRAM  |  Build distribution
+------------------+
     |
     v
+------------------+
| BENCH_REPORTER   |  Format output
+------------------+
     |
     v
stdout / file
```

### Scenario Schema

```yaml
# scenario.yaml
name: "Chat Application Load Test"
description: "Simulate chat room with mixed message sizes"

setup:
  url: "ws://chat.example.com/room/1"
  headers:
    Authorization: "Bearer test-token"
  subprotocol: "chat-v1"

phases:
  - name: "warmup"
    duration: 10
    connections: 10
    rate: 10

  - name: "ramp"
    duration: 30
    connections: 100
    ramp_up: true

  - name: "steady"
    duration: 60
    connections: 100
    rate: 1000

  - name: "spike"
    duration: 10
    connections: 500
    rate: 5000

actions:
  - send: '{"type":"join","room":"test"}'
    wait: 1000

  - send_random:
      - '{"type":"message","text":"Hello"}'
      - '{"type":"message","text":"How are you?"}'
      - '{"type":"message","text":"This is a longer message for testing"}'
    rate: 10

  - receive:
      expect_json: "$.type=ack"
      timeout: 5000

success_criteria:
  - latency_p99_ms: 100
  - error_rate_percent: 1
  - throughput_min: 900
```

### Statistics Model

```
BENCH_STATS:
  - total_connections: INTEGER
  - successful_connections: INTEGER
  - failed_connections: INTEGER
  - total_messages_sent: INTEGER
  - total_messages_received: INTEGER
  - total_bytes_sent: INTEGER_64
  - total_bytes_received: INTEGER_64
  - test_duration_ms: INTEGER_64
  - latency_samples: ARRAYED_LIST [INTEGER_64]  (microseconds)

Derived Metrics:
  - connection_success_rate: REAL = successful / total * 100
  - messages_per_second: REAL = total_sent / (duration / 1000)
  - bytes_per_second: REAL = total_bytes / (duration / 1000)
  - latency_min_us: INTEGER_64
  - latency_max_us: INTEGER_64
  - latency_avg_us: REAL
  - latency_stddev_us: REAL
  - latency_p50_us: INTEGER_64
  - latency_p90_us: INTEGER_64
  - latency_p95_us: INTEGER_64
  - latency_p99_us: INTEGER_64
```

### Error Handling

| Error Type | Handling | User Message | Impact |
|------------|----------|--------------|--------|
| Connection refused | Count as failed | Stats include failed count | Continue other workers |
| Connection timeout | Count as failed | Stats include timeout count | Continue other workers |
| Send error | Count as error | Stats include error count | Reconnect or stop worker |
| Receive timeout | Count as timeout | Stats include timeout count | Continue next message |
| Protocol error | Close connection | Log error details | Reconnect or stop worker |
| All workers failed | Stop benchmark | "All connections failed" | Exit with error code |

### Output Formats

**Text Output (default):**
```
WebSocket Benchmark Results
===========================
URL: ws://localhost:8080/ws
Connections: 100
Duration: 30.05s

Connections:
  Successful: 100 (100.0%)
  Failed: 0

Messages:
  Sent: 10,000
  Received: 10,000
  Throughput: 332.8 msg/sec

Latency (ms):
  Min: 0.42
  Max: 45.21
  Avg: 2.15
  Std Dev: 3.41

Percentiles (ms):
  p50: 1.23
  p90: 4.56
  p95: 7.89
  p99: 15.34
```

**JSON Output:**
```json
{
  "benchmark": {
    "url": "ws://localhost:8080/ws",
    "connections": 100,
    "messages_per_connection": 100,
    "duration_ms": 30050
  },
  "connections": {
    "total": 100,
    "successful": 100,
    "failed": 0,
    "success_rate": 100.0
  },
  "messages": {
    "sent": 10000,
    "received": 10000,
    "bytes_sent": 150000,
    "bytes_received": 150000,
    "throughput_per_second": 332.8
  },
  "latency_us": {
    "min": 420,
    "max": 45210,
    "avg": 2150.5,
    "stddev": 3410.2,
    "p50": 1230,
    "p90": 4560,
    "p95": 7890,
    "p99": 15340
  },
  "histogram": [
    {"bucket_ms": "0-1", "count": 2345},
    {"bucket_ms": "1-2", "count": 3456},
    {"bucket_ms": "2-5", "count": 2890},
    {"bucket_ms": "5-10", "count": 987},
    {"bucket_ms": "10-50", "count": 322}
  ]
}
```

## GUI/TUI Future Path

**CLI foundation enables:**

1. **TUI Dashboard:** Use simple_tui for:
   - Real-time throughput graph
   - Live latency histogram
   - Connection status matrix
   - Progress bar with ETA

2. **Shared Components:**
   - `BENCH_COORDINATOR`: Worker management unchanged
   - `BENCH_STATS`: Feed data to TUI displays
   - `BENCH_HISTOGRAM`: Render as ASCII chart

3. **GUI Future:**
   - Visual scenario builder
   - Interactive latency charts
   - Connection timeline view
   - Comparative benchmark analysis

## Sample Usage

```bash
# Basic benchmark: 1 connection, 100 messages
ws-bench ws://localhost:8080/ws

# Concurrent connections
ws-bench ws://localhost:8080/ws -c 100 -m 1000

# Timed test with ramp-up
ws-bench ws://localhost:8080/ws -c 500 -t 60 --ramp-up 10

# Echo latency test with JSON output
ws-bench ws://echo.websocket.org -c 10 -m 100 --echo -o json

# CI/CD integration
ws-bench ws://staging.example.com/ws -c 50 -m 500 -o json > benchmark.json

# Scenario-based test
ws-bench --scenario load-test.yaml

# Custom payload from file
ws-bench ws://localhost:8080/ws -c 100 --data-file payload.json

# Rate-limited test
ws-bench ws://api.example.com/ws -c 100 -r 1000 -t 60
```
