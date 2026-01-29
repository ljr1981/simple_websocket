# WS-BENCH - Ecosystem Integration

## simple_* Dependencies

### Required Libraries

| Library | Purpose | Integration Point |
|---------|---------|-------------------|
| simple_websocket | Core WebSocket protocol | WS_FRAME, WS_HANDSHAKE, WS_FRAME_PARSER |
| simple_json | JSON output, config | Report generation, scenario parsing |
| simple_cli | Command-line parsing | Argument handling, help generation |
| simple_logger | Debug logging | Troubleshooting, verbose mode |
| simple_datetime | Timestamp handling | Timing, report timestamps |

### Optional Libraries

| Library | Purpose | When Needed |
|---------|---------|-------------|
| simple_csv | CSV export | --output csv option |
| simple_yaml | Scenario files | --scenario option |
| simple_chart | ASCII histograms | --histogram option |
| simple_config | Configuration | Pro feature: benchmark profiles |
| simple_math | Statistical calculations | Standard deviation, percentiles |

## Integration Patterns

### simple_websocket Integration

**Purpose:** WebSocket protocol for benchmark connections

**Usage:**
```eiffel
class BENCH_WORKER

feature -- Connection

    connect
            -- Establish WebSocket connection.
        local
            handshake: WS_HANDSHAKE
            timer: BENCH_TIMER
        do
            create timer.make
            timer.start

            -- TCP connect
            socket.connect

            -- WebSocket handshake
            create handshake.make
            socket.put_string (handshake.create_client_request (host, path))

            if handshake.validate_server_response (socket.read_line_until_crlf) then
                timer.stop
                connect_latency := timer.elapsed_microseconds
                is_connected := True
                stats.record_connection_success
            else
                stats.record_connection_failure (handshake.last_error)
            end
        end

feature -- Messaging

    send_and_measure (a_payload: STRING)
            -- Send message and measure response time (for echo test).
        require
            connected: is_connected
        local
            frame: WS_FRAME
            timer: BENCH_TIMER
        do
            create frame.make_text (a_payload, True)
            frame.set_mask (generate_mask_key)

            create timer.make
            timer.start

            socket.put_bytes (frame.to_bytes)
            stats.record_message_sent (a_payload.count)

            if echo_mode then
                -- Wait for echo response
                parser.add_bytes (socket.read_available)
                if parser.parse and parser.has_frame then
                    timer.stop
                    sampler.add_sample (timer.elapsed_microseconds)
                    stats.record_message_received (parser.last_frame.payload_length)
                end
            end
        end

end
```

**Data flow:** BENCH_WORKER -> WS_HANDSHAKE -> WS_FRAME -> Socket -> WS_FRAME_PARSER -> BENCH_SAMPLER

### simple_json Integration

**Purpose:** JSON output format and scenario configuration

**Report generation:**
```eiffel
class BENCH_REPORTER

feature -- JSON Output

    format_json (a_stats: BENCH_STATS): STRING
            -- Generate JSON benchmark report.
        local
            json: SIMPLE_JSON_OBJECT
            connections, messages, latency: SIMPLE_JSON_OBJECT
        do
            create json.make

            -- Benchmark info
            json.put_string ("url", config.url)
            json.put_integer ("connections", config.connection_count)
            json.put_integer ("duration_ms", a_stats.duration_ms)

            -- Connections section
            create connections.make
            connections.put_integer ("total", a_stats.total_connections)
            connections.put_integer ("successful", a_stats.successful_connections)
            connections.put_integer ("failed", a_stats.failed_connections)
            connections.put_real ("success_rate", a_stats.connection_success_rate)
            json.put_object ("connections", connections)

            -- Messages section
            create messages.make
            messages.put_integer ("sent", a_stats.messages_sent)
            messages.put_integer ("received", a_stats.messages_received)
            messages.put_real ("throughput_per_second", a_stats.messages_per_second)
            json.put_object ("messages", messages)

            -- Latency section
            create latency.make
            latency.put_integer ("min_us", a_stats.latency_min)
            latency.put_integer ("max_us", a_stats.latency_max)
            latency.put_real ("avg_us", a_stats.latency_avg)
            latency.put_real ("stddev_us", a_stats.latency_stddev)
            latency.put_integer ("p50_us", a_stats.percentile (50))
            latency.put_integer ("p90_us", a_stats.percentile (90))
            latency.put_integer ("p95_us", a_stats.percentile (95))
            latency.put_integer ("p99_us", a_stats.percentile (99))
            json.put_object ("latency_us", latency)

            -- Histogram
            if include_histogram then
                json.put_array ("histogram", format_histogram_array (a_stats.histogram))
            end

            Result := json.to_string_pretty
        end

end
```

**Data flow:** BENCH_STATS -> BENCH_REPORTER -> SIMPLE_JSON_OBJECT -> JSON string

### simple_cli Integration

**Purpose:** Command-line argument parsing

**Usage:**
```eiffel
class BENCH_CLI

inherit
    SIMPLE_CLI_APPLICATION
        redefine
            application_name,
            configure_options
        end

feature -- Configuration

    application_name: STRING = "ws-bench"

    configure_options
            -- Define CLI options.
        do
            add_positional ("url", "WebSocket URL to benchmark", True)

            -- Connection options
            add_option ("connections", "c", "Number of concurrent connections", True)
            add_option ("ramp-up", Void, "Ramp-up time in seconds", True)
            add_option ("connect-timeout", Void, "Connection timeout in ms", True)

            -- Message options
            add_option ("messages", "m", "Messages per connection", True)
            add_option ("rate", "r", "Max messages per second", True)
            add_option ("data", "d", "Message payload", True)
            add_option ("data-file", Void, "Payload file path", True)
            add_flag ("binary", Void, "Send binary messages")

            -- Duration options
            add_option ("time", "t", "Test duration in seconds", True)
            add_option ("warmup", Void, "Warmup period in seconds", True)

            -- Output options
            add_option ("output", "o", "Output format: text, json, csv", True)
            add_flag ("histogram", Void, "Show latency histogram")
            add_option ("percentiles", Void, "Percentiles to show", True)
            add_option ("report", Void, "Report file path", True)
            add_flag ("quiet", Void, "Suppress progress output")

            -- Other options
            add_flag ("echo", Void, "Measure echo round-trip time")
        end

feature -- Execution

    execute
            -- Run benchmark.
        local
            config: BENCH_CONFIG
            coordinator: BENCH_COORDINATOR
            reporter: BENCH_REPORTER
        do
            config := build_config_from_args
            create coordinator.make (config)
            coordinator.run_benchmark
            create reporter.make (config.output_format)
            io.put_string (reporter.format (coordinator.stats))
        end

end
```

**Data flow:** argv -> SIMPLE_CLI -> BENCH_CONFIG -> BENCH_COORDINATOR

### simple_datetime Integration

**Purpose:** High-precision timing and timestamps

**Usage:**
```eiffel
class BENCH_TIMER

feature -- Timing

    start
            -- Start timer.
        do
            start_time := high_precision_time
        ensure
            running: is_running
        end

    stop
            -- Stop timer.
        require
            running: is_running
        do
            end_time := high_precision_time
        ensure
            stopped: not is_running
        end

    elapsed_microseconds: INTEGER_64
            -- Elapsed time in microseconds.
        require
            stopped: not is_running
        do
            Result := end_time - start_time
        ensure
            non_negative: Result >= 0
        end

feature {NONE} -- Implementation

    high_precision_time: INTEGER_64
            -- Current time in microseconds.
        local
            dt: SIMPLE_DATETIME
        do
            create dt.make_now_utc
            Result := dt.epoch_microseconds
        end

end
```

**Data flow:** SIMPLE_DATETIME -> BENCH_TIMER -> BENCH_SAMPLER

### simple_csv Integration

**Purpose:** CSV export for trend analysis

**Usage:**
```eiffel
class BENCH_REPORTER

feature -- CSV Output

    format_csv (a_stats: BENCH_STATS): STRING
            -- Generate CSV benchmark report.
        local
            csv: SIMPLE_CSV_WRITER
        do
            create csv.make

            -- Header row
            csv.add_header (<<"timestamp", "url", "connections",
                            "messages_sent", "throughput_per_sec",
                            "latency_min_us", "latency_max_us", "latency_avg_us",
                            "latency_p50_us", "latency_p90_us", "latency_p99_us">>)

            -- Data row
            csv.add_row (<<current_timestamp, config.url, a_stats.total_connections.out,
                          a_stats.messages_sent.out, a_stats.messages_per_second.out,
                          a_stats.latency_min.out, a_stats.latency_max.out,
                          a_stats.latency_avg.out, a_stats.percentile (50).out,
                          a_stats.percentile (90).out, a_stats.percentile (99).out>>)

            Result := csv.to_string
        end

end
```

**Data flow:** BENCH_STATS -> SIMPLE_CSV_WRITER -> CSV string

### simple_chart Integration

**Purpose:** ASCII histogram visualization

**Usage:**
```eiffel
class BENCH_REPORTER

feature -- Histogram

    format_histogram (a_histogram: BENCH_HISTOGRAM): STRING
            -- Generate ASCII histogram of latency distribution.
        local
            chart: SIMPLE_ASCII_CHART
        do
            create chart.make_histogram
            chart.set_title ("Latency Distribution")
            chart.set_x_label ("Latency (ms)")
            chart.set_y_label ("Count")

            across a_histogram.buckets as b loop
                chart.add_bar (b.item.label, b.item.count)
            end

            Result := chart.render
        end

end
```

**Output example:**
```
Latency Distribution (ms)
|
|    ########
|    ########  ######
|    ########  ######  ####
|    ########  ######  ####  ##
|    ########  ######  ####  ##   #
+----+-------+-------+------+-----+----+----+----+
     0-1      1-2     2-5    5-10  10-20 20-50 50+
```

## Dependency Graph

```
ws-bench
    |
    +-- simple_websocket (required)
    |       +-- simple_base64
    |       +-- simple_hash
    |
    +-- simple_json (required)
    |
    +-- simple_cli (required)
    |
    +-- simple_logger (required)
    |
    +-- simple_datetime (required)
    |
    +-- simple_csv (optional)
    |
    +-- simple_yaml (optional)
    |
    +-- simple_chart (optional)
    |
    +-- simple_math (optional)
    |
    +-- ISE base (required)
    +-- ISE net (required for sockets)
    +-- ISE thread (required for concurrent workers)
```

## ECF Configuration

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<system xmlns="http://www.eiffel.com/developers/xml/configuration-1-23-0"
        name="ws_bench" uuid="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX">

    <description>WS-BENCH: WebSocket Benchmark Tool</description>

    <target name="ws_bench">
        <root class="BENCH_CLI" feature="make"/>

        <option warning="warning" syntax="standard">
            <assertions precondition="true" postcondition="true" check="true"/>
        </option>

        <setting name="console_application" value="true"/>
        <setting name="concurrency" value="thread"/>

        <capability>
            <concurrency support="thread"/>
            <void_safety support="all"/>
        </capability>

        <!-- Application source -->
        <cluster name="src" location=".\src\" recursive="true"/>

        <!-- simple_* dependencies (required) -->
        <library name="simple_websocket" location="$SIMPLE_EIFFEL\simple_websocket\simple_websocket.ecf"/>
        <library name="simple_json" location="$SIMPLE_EIFFEL\simple_json\simple_json.ecf"/>
        <library name="simple_cli" location="$SIMPLE_EIFFEL\simple_cli\simple_cli.ecf"/>
        <library name="simple_logger" location="$SIMPLE_EIFFEL\simple_logger\simple_logger.ecf"/>
        <library name="simple_datetime" location="$SIMPLE_EIFFEL\simple_datetime\simple_datetime.ecf"/>

        <!-- simple_* dependencies (optional) -->
        <library name="simple_csv" location="$SIMPLE_EIFFEL\simple_csv\simple_csv.ecf"/>
        <library name="simple_yaml" location="$SIMPLE_EIFFEL\simple_yaml\simple_yaml.ecf"/>
        <library name="simple_chart" location="$SIMPLE_EIFFEL\simple_chart\simple_chart.ecf"/>

        <!-- ISE dependencies -->
        <library name="base" location="$ISE_LIBRARY\library\base\base.ecf"/>
        <library name="net" location="$ISE_LIBRARY\library\net\net.ecf"/>
        <library name="thread" location="$ISE_LIBRARY\library\thread\thread.ecf"/>
    </target>

    <target name="ws_bench_tests" extends="ws_bench">
        <root class="TEST_APP" feature="make"/>
        <library name="simple_testing" location="$SIMPLE_EIFFEL\simple_testing\simple_testing.ecf"/>
        <cluster name="tests" location=".\tests\" recursive="true"/>
    </target>

</system>
```

## Integration Notes

### Thread Model

WS-BENCH uses a thread-per-connection model:
- Main thread: CLI parsing, coordination, reporting
- Worker threads: One per WebSocket connection (configurable pool)
- Stats thread: Aggregate samples from workers

Thread-safe patterns:
- Workers push samples to thread-safe queue
- Stats aggregator pulls from queue periodically
- Coordinator signals workers via atomic flags

### Timing Precision

For accurate latency measurement:
- Use `simple_datetime.epoch_microseconds` for sub-millisecond precision
- Avoid system calls in hot path
- Pre-allocate message buffers
- Minimize garbage collection during measurement

### Memory Efficiency

For high connection counts:
- Reuse frame buffers within workers
- Stream samples to aggregator (don't store all)
- Use reservoir sampling for percentiles (Pro feature)
- Configurable sample rate for very high throughput
