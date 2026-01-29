# WS-BENCH - Build Plan

## Phase Overview

| Phase | Deliverable | Effort | Dependencies |
|-------|-------------|--------|--------------|
| Phase 1 | MVP Benchmark - Single connection, basic stats | 3 days | simple_websocket, simple_cli |
| Phase 2 | Full Benchmark - Multi-connection, JSON output | 5 days | Phase 1 + simple_json, simple_datetime, threading |
| Phase 3 | Polish - Scenarios, histograms, CSV export | 4 days | Phase 2 + simple_yaml, simple_chart, simple_csv |

---

## Phase 1: MVP

### Objective

Demonstrate basic WebSocket benchmarking. Single connection, configurable message count, measure latency for echo responses. Basic text statistics output.

### Deliverables

1. **BENCH_CLI** - Basic argument parsing (url, messages, timeout)
2. **BENCH_WORKER** - Single connection handler with timing
3. **BENCH_TIMER** - High-precision timing
4. **BENCH_STATS** - Basic statistics (min, max, avg, count)
5. **BENCH_REPORTER** - Text output format

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T1.1 | Create project structure | ECF compiles, TEST_APP runs |
| T1.2 | Implement BENCH_TIMER | Microsecond precision timing |
| T1.3 | Implement BENCH_WORKER.connect | WebSocket connection established |
| T1.4 | Implement BENCH_WORKER.send_and_measure | Message sent, response timed |
| T1.5 | Implement BENCH_STATS basic | Min, max, avg calculation |
| T1.6 | Implement BENCH_CLI argument parsing | URL and options parsed |
| T1.7 | Implement BENCH_REPORTER text | Human-readable output |
| T1.8 | Add --echo mode | Round-trip measurement |
| T1.9 | Add message count option | -m flag works |
| T1.10 | Write MVP tests | Core timing and stats tested |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Basic benchmark | `ws-bench ws://echo.websocket.org` | Stats printed |
| Custom messages | `ws-bench ws://... -m 1000` | 1000 messages sent |
| Echo timing | `ws-bench ws://echo.websocket.org --echo` | Latency in output |
| Connection failure | `ws-bench ws://invalid.host` | "Connection failed", exit 1 |
| Invalid URL | `ws-bench not-a-url` | "Invalid URL", exit 1 |
| Help | `ws-bench --help` | Usage information |

### Phase 1 Directory Structure

```
ws-bench/
├── ws_bench.ecf
├── src/
│   ├── bench_cli.e
│   ├── bench_worker.e
│   ├── bench_timer.e
│   ├── bench_stats.e
│   └── bench_reporter.e
└── tests/
    ├── test_app.e
    ├── test_timer.e
    └── test_stats.e
```

---

## Phase 2: Full Implementation

### Objective

Complete benchmark with concurrent connections, JSON output, and thread-safe statistics. Tool is CI/CD-ready with machine-parseable output.

### Deliverables

1. **BENCH_COORDINATOR** - Manage multiple workers
2. **BENCH_SAMPLER** - Thread-safe sample collection
3. **BENCH_PERCENTILE** - Percentile calculations
4. **JSON output** - Structured benchmark report
5. **Progress display** - Real-time progress during test

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T2.1 | Implement BENCH_COORDINATOR | Manages N workers |
| T2.2 | Add threading support | Workers run concurrently |
| T2.3 | Implement BENCH_SAMPLER | Thread-safe sample collection |
| T2.4 | Implement percentile calculation | p50, p90, p95, p99 accurate |
| T2.5 | Add standard deviation | Stddev in stats |
| T2.6 | Implement JSON output | --output json works |
| T2.7 | Implement BENCH_PROGRESS | Real-time progress bar |
| T2.8 | Add ramp-up support | --ramp-up delays connections |
| T2.9 | Add rate limiting | --rate caps throughput |
| T2.10 | Write full test suite | All features tested |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Multi-connection | `ws-bench ws://... -c 100` | 100 connections |
| JSON output | `ws-bench ws://... -o json` | Valid JSON |
| Percentiles | `ws-bench ws://... --echo` | p50, p90, p95, p99 in output |
| Ramp-up | `ws-bench ws://... -c 100 --ramp-up 10` | Gradual connection increase |
| Rate limit | `ws-bench ws://... -r 100` | ~100 msg/sec |
| Quiet mode | `ws-bench ws://... --quiet` | No progress, only final stats |
| CI integration | Run and check exit code | 0 on success, non-0 on failure |

### Phase 2 Threading Model

```
Main Thread
     |
     +-- Create BENCH_COORDINATOR
     |
     +-- Spawn N worker threads
     |         |
     |         +-- Worker 1: connect, send/receive, push samples
     |         +-- Worker 2: connect, send/receive, push samples
     |         +-- ...
     |         +-- Worker N: connect, send/receive, push samples
     |
     +-- Progress display loop (every 1 sec)
     |
     +-- Wait for workers to complete
     |
     +-- Aggregate statistics
     |
     +-- Generate report
```

---

## Phase 3: Production Polish

### Objective

Add scenario support, histogram visualization, CSV export, and production hardening. Tool is ready for enterprise use.

### Deliverables

1. **BENCH_SCENARIO** - YAML scenario loading
2. **BENCH_HISTOGRAM** - Latency distribution
3. **CSV export** - Trend analysis output
4. **Duration mode** - Time-based tests
5. **Warmup period** - Exclude initial samples

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T3.1 | Implement YAML scenario loading | --scenario works |
| T3.2 | Implement BENCH_HISTOGRAM | Latency buckets |
| T3.3 | Add ASCII histogram output | --histogram visualizes |
| T3.4 | Implement CSV export | --output csv works |
| T3.5 | Add duration mode | -t for timed tests |
| T3.6 | Add warmup period | --warmup excludes samples |
| T3.7 | Add data-file option | --data-file loads payload |
| T3.8 | Harden error handling | No unhandled exceptions |
| T3.9 | Final documentation | README + examples |
| T3.10 | Performance optimization | Minimal overhead |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Scenario test | `ws-bench --scenario load.yaml` | Phases executed |
| Histogram | `ws-bench ws://... --histogram` | ASCII histogram |
| CSV output | `ws-bench ws://... -o csv` | Valid CSV |
| Duration test | `ws-bench ws://... -t 60` | Runs for 60 seconds |
| Warmup | `ws-bench ws://... --warmup 5` | First 5s excluded |
| Data file | `ws-bench ws://... --data-file msg.json` | File contents sent |

---

## ECF Target Structure

```xml
<!-- Library target (reusable benchmark components) -->
<target name="ws_bench_lib">
    <option warning="warning" syntax="standard">
        <assertions precondition="true" postcondition="true"/>
    </option>
    <setting name="concurrency" value="thread"/>
    <capability>
        <concurrency support="thread"/>
        <void_safety support="all"/>
    </capability>
    <cluster name="src" location=".\src\" recursive="true">
        <file_rule>
            <exclude>/bench_cli\.e$</exclude>
        </file_rule>
    </cluster>
    <!-- Dependencies -->
    <library name="simple_websocket" location="$SIMPLE_EIFFEL\simple_websocket\simple_websocket.ecf"/>
    <library name="simple_json" location="$SIMPLE_EIFFEL\simple_json\simple_json.ecf"/>
    <library name="simple_cli" location="$SIMPLE_EIFFEL\simple_cli\simple_cli.ecf"/>
    <library name="simple_logger" location="$SIMPLE_EIFFEL\simple_logger\simple_logger.ecf"/>
    <library name="simple_datetime" location="$SIMPLE_EIFFEL\simple_datetime\simple_datetime.ecf"/>
    <library name="simple_csv" location="$SIMPLE_EIFFEL\simple_csv\simple_csv.ecf"/>
    <library name="simple_yaml" location="$SIMPLE_EIFFEL\simple_yaml\simple_yaml.ecf"/>
    <library name="simple_chart" location="$SIMPLE_EIFFEL\simple_chart\simple_chart.ecf"/>
    <library name="base" location="$ISE_LIBRARY\library\base\base.ecf"/>
    <library name="net" location="$ISE_LIBRARY\library\net\net.ecf"/>
    <library name="thread" location="$ISE_LIBRARY\library\thread\thread.ecf"/>
</target>

<!-- CLI executable target -->
<target name="ws_bench" extends="ws_bench_lib">
    <root class="BENCH_CLI" feature="make"/>
    <setting name="console_application" value="true"/>
</target>

<!-- Test target -->
<target name="ws_bench_tests" extends="ws_bench_lib">
    <root class="TEST_APP" feature="make"/>
    <library name="simple_testing" location="$SIMPLE_EIFFEL\simple_testing\simple_testing.ecf"/>
    <cluster name="tests" location=".\tests\" recursive="true"/>
</target>
```

---

## Build Commands

```bash
# Compile CLI (development)
/d/prod/ec.sh -batch -config ws_bench.ecf -target ws_bench -c_compile

# Compile CLI (finalized/optimized)
/d/prod/ec.sh -batch -config ws_bench.ecf -target ws_bench -finalize -c_compile

# Run tests
/d/prod/ec.sh -batch -config ws_bench.ecf -target ws_bench_tests -c_compile
./EIFGENs/ws_bench_tests/W_code/ws_bench.exe

# Quick benchmark
./EIFGENs/ws_bench/W_code/ws_bench.exe ws://echo.websocket.org -m 100 --echo
```

---

## Success Criteria

| Criterion | Measure | Target |
|-----------|---------|--------|
| Compiles | Zero errors | 100% |
| Tests pass | All test cases | 100% |
| Connection capacity | Max concurrent connections | 10,000+ |
| Throughput | Messages per second | 100,000+ |
| Timing accuracy | Measurement precision | < 1ms |
| Memory | Per-connection overhead | < 50KB |
| CPU | Benchmark overhead | < 10% |
| JSON valid | All JSON output parseable | 100% |
| Exit codes | Correct for all scenarios | 100% |
| Documentation | README + examples | Complete |

---

## Benchmark Validation

### Reference Tests

Before release, validate against known tools:

```bash
# Compare with wscat for connection
wscat -c ws://echo.websocket.org -x "hello"

# Compare message timing
# (Manual verification with stopwatch or external tool)

# Compare with Artillery for load
# Run equivalent test and compare throughput numbers
```

### Performance Baseline

Establish baseline on reference hardware:
- 1,000 connections: < 5 seconds to establish
- 10,000 messages/sec: achievable on single core
- Latency measurement: < 100 microsecond overhead

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Thread contention | Use lock-free queues for samples |
| Socket exhaustion | Test with ulimit adjustments |
| Memory growth | Profile under load |
| Timing inaccuracy | Validate against hardware timer |
| Percentile accuracy | Use appropriate algorithm (T-Digest or sorted array) |
| GC pauses | Pre-allocate, reuse objects |
