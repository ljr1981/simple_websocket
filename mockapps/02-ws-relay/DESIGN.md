# WS-RELAY - Technical Design

## Architecture

### Component Overview

```
+----------------------------------------------------------+
|                       WS-RELAY                            |
+----------------------------------------------------------+
|  Server Interface Layer                                   |
|    - RELAY_SERVER: Accept client connections              |
|    - RELAY_CLI: Command-line startup/control              |
|    - RELAY_HEALTH: Health check endpoint                  |
+----------------------------------------------------------+
|  Connection Management Layer                              |
|    - RELAY_SESSION: Client-Upstream pair management       |
|    - RELAY_POOL: Upstream connection pooling              |
|    - RELAY_ROUTER: Path-based routing decisions           |
+----------------------------------------------------------+
|  Message Processing Layer                                 |
|    - RELAY_FILTER: Message allow/deny filtering           |
|    - RELAY_TRANSFORM: Message modification                |
|    - RELAY_LOGGER: Traffic logging                        |
+----------------------------------------------------------+
|  Protocol Layer                                           |
|    - WS_HANDSHAKE: HTTP upgrade (client & server side)    |
|    - WS_FRAME: Frame encoding/decoding                    |
|    - WS_FRAME_PARSER: Streaming frame parser              |
+----------------------------------------------------------+
|  Transport Layer (External)                               |
|    - EiffelNet SOCKET: TCP server and client sockets      |
+----------------------------------------------------------+
```

### Class Design

| Class | Responsibility | Key Features |
|-------|----------------|--------------|
| RELAY_CLI | Server startup and control | parse_args, start_server, stop_server |
| RELAY_SERVER | Accept and manage connections | bind, accept, shutdown |
| RELAY_SESSION | Client-upstream pair | relay_client_to_upstream, relay_upstream_to_client |
| RELAY_ROUTER | Routing decisions | route_by_path, route_by_header, select_upstream |
| RELAY_POOL | Upstream connection pool | get_connection, release_connection, health_check |
| RELAY_FILTER | Message filtering | should_allow, match_pattern, log_filtered |
| RELAY_TRANSFORM | Message modification | transform_json, inject_field, mask_field |
| RELAY_LOGGER | Traffic logging | log_message, log_connection, flush |
| RELAY_CONFIG | Configuration management | load_config, reload_config, validate |
| RELAY_METRICS | Metrics collection | increment_counter, record_latency, expose |
| RELAY_HEALTH | Health endpoint | check_upstreams, report_status |

### Command Structure

```bash
ws-relay [options]

Options:
  --config FILE         Configuration file (required)
  --bind HOST:PORT      Override bind address (default from config)
  --log-level LEVEL     Logging level: debug, info, warn, error
  --log-file FILE       Log file path (default: stdout)
  --pid-file FILE       PID file for process management
  --test-config         Validate configuration and exit
  --reload              Send reload signal to running server
  --stop                Send stop signal to running server
  --help                Show help message
  --version             Show version information

Signals:
  SIGHUP                Reload configuration
  SIGTERM               Graceful shutdown
  SIGINT                Immediate shutdown
```

### Data Flow

```
Client WebSocket Connect
        |
        v
+------------------+
| RELAY_SERVER     |  Accept connection
+------------------+
        |
        v
+------------------+
| WS_HANDSHAKE     |  Parse client handshake
+------------------+
        |
        v
+------------------+
| RELAY_ROUTER     |  Determine upstream based on path/headers
+------------------+
        |
        v
+------------------+
| RELAY_POOL       |  Get/create upstream connection
+------------------+
        |
        v
+------------------+
| WS_HANDSHAKE     |  Perform upstream handshake
+------------------+
        |
        v
+------------------+
| RELAY_SESSION    |  Begin bidirectional relay
+------------------+

Message Flow (client to upstream):
Client -> RELAY_SESSION -> RELAY_FILTER -> RELAY_TRANSFORM -> RELAY_LOGGER -> Upstream

Message Flow (upstream to client):
Upstream -> RELAY_SESSION -> RELAY_FILTER -> RELAY_TRANSFORM -> RELAY_LOGGER -> Client
```

### Configuration Schema

```yaml
# ws-relay.yaml
server:
  bind: "0.0.0.0:8080"
  max_connections: 1000
  read_timeout: 30000
  write_timeout: 30000

upstreams:
  default:
    url: "ws://backend.internal:9000/ws"
    timeout: 5000

  api-v2:
    url: "ws://backend-v2.internal:9001/ws"
    timeout: 10000

  fallback:
    url: "ws://backup.internal:9000/ws"
    timeout: 5000

routing:
  - path: "/api/v2/*"
    upstream: api-v2

  - path: "/api/*"
    upstream: default

  - path: "/*"
    upstream: default

filters:
  - name: "block-sensitive"
    action: drop
    direction: upstream  # client-to-upstream
    pattern:
      json_path: "$.type"
      equals: "admin_command"
    log: true

  - name: "allow-subscriptions"
    action: allow
    direction: both
    pattern:
      text_contains: "subscribe"

transforms:
  - name: "add-relay-header"
    direction: downstream  # upstream-to-client
    action: json_insert
    path: "$.metadata.relay"
    value: "ws-relay-v1"

  - name: "mask-tokens"
    direction: both
    action: json_mask
    path: "$.auth.token"
    mask_char: "*"

logging:
  level: info
  format: json
  file: "/var/log/ws-relay/traffic.log"
  include_payload: false
  max_payload_log: 1024

metrics:
  enabled: true
  endpoint: "/metrics"
  port: 9090

health:
  endpoint: "/health"
  port: 8081
  check_upstreams: true
```

### Error Handling

| Error Type | Handling | Client Message | Log Level |
|------------|----------|----------------|-----------|
| Upstream unavailable | Try fallback if configured | Close 1011 "Server error" | ERROR |
| Upstream timeout | Close session | Close 1011 "Backend timeout" | WARN |
| Client disconnect | Close upstream | N/A | INFO |
| Invalid frame | Close client | Close 1002 "Protocol error" | WARN |
| Config error | Refuse reload | N/A | ERROR |
| Memory pressure | Reject new connections | Close 1013 "Try again later" | WARN |

### State Diagram

```
                    +----------+
                    |  INIT    |
                    +----------+
                         |
                         | load_config()
                         v
                    +----------+
                    | STARTING |
                    +----------+
                         |
                         | bind() success
                         v
                    +----------+
                    | RUNNING  |<----+
                    +----------+     |
                         |           | reload_config()
           +-------------+           |
           |             |           |
           v             v           |
      +---------+   +----------+     |
      | SESSION |   | RELOAD   |-----+
      |  (many) |   +----------+
      +---------+
           |
           | SIGTERM
           v
                    +----------+
                    | DRAINING |  (finish active sessions)
                    +----------+
                         |
                         v
                    +----------+
                    | STOPPED  |
                    +----------+
```

## GUI/TUI Future Path

**CLI foundation enables:**

1. **TUI Dashboard:** Use simple_tui for:
   - Live connection count and message rates
   - Active session list with status
   - Recent log messages
   - Upstream health status

2. **Shared Components:**
   - `RELAY_SERVER`: Core server logic unchanged
   - `RELAY_METRICS`: Feed data to TUI displays
   - `RELAY_CONFIG`: Config editing in TUI

3. **GUI Future:**
   - Visual routing rule editor
   - Real-time message flow diagram
   - Filter/transform rule builder
   - Connection debugger

## Sample Usage

```bash
# Start relay with config file
ws-relay --config /etc/ws-relay/config.yaml

# Test configuration
ws-relay --config config.yaml --test-config

# Run with debug logging
ws-relay --config config.yaml --log-level debug

# Run in foreground with stdout logging
ws-relay --config config.yaml --log-file -

# Reload configuration (running server)
ws-relay --config /etc/ws-relay/config.yaml --reload

# Graceful shutdown
ws-relay --pid-file /var/run/ws-relay.pid --stop

# Docker-style run
ws-relay --config /config/relay.yaml --bind 0.0.0.0:8080
```

## Protocol Compatibility

| Feature | Client-Side | Upstream-Side |
|---------|-------------|---------------|
| Text frames | Full support | Full support |
| Binary frames | Full support | Full support |
| Ping/Pong | Relay transparent | Relay transparent |
| Close frames | Code/reason preserved | Code/reason preserved |
| Fragmentation | Reassemble optional | Reassemble optional |
| Masking | Unmask client, mask upstream | Unmask upstream, no mask to client |
| Extensions | Pass through | Pass through |
| Subprotocols | Configurable translation | Configurable translation |
