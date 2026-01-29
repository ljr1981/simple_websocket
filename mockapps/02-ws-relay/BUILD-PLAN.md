# WS-RELAY - Build Plan

## Phase Overview

| Phase | Deliverable | Effort | Dependencies |
|-------|-------------|--------|--------------|
| Phase 1 | MVP Relay - Basic pass-through | 4 days | simple_websocket, simple_http |
| Phase 2 | Full Relay - Routing, logging, filtering | 5 days | Phase 1 + simple_json, simple_yaml, simple_logger |
| Phase 3 | Polish - Transforms, metrics, hot reload | 4 days | Phase 2 + simple_config |

---

## Phase 1: MVP

### Objective

Demonstrate basic WebSocket relay functionality. Server accepts client connections, connects to a configured upstream, and relays messages bidirectionally. Single upstream, no transformation.

### Deliverables

1. **RELAY_CLI** - Basic command-line interface for server startup
2. **RELAY_SERVER** - Accept incoming WebSocket connections
3. **RELAY_SESSION** - Manage client-upstream pair, relay messages
4. **RELAY_CONFIG** - Load basic configuration (bind address, upstream URL)

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T1.1 | Create project structure | ECF compiles, TEST_APP runs |
| T1.2 | Implement RELAY_SERVER.bind | Server listens on port |
| T1.3 | Implement RELAY_SERVER.accept | Client connections accepted |
| T1.4 | Implement server-side handshake | HTTP upgrade processed correctly |
| T1.5 | Implement RELAY_SESSION.connect_upstream | Upstream connection established |
| T1.6 | Implement client-side handshake | Upstream upgrade completed |
| T1.7 | Implement message relay loop | Messages pass through both directions |
| T1.8 | Implement graceful close | Close frames relayed, connections cleaned |
| T1.9 | Implement basic config loading | Bind/upstream from config file |
| T1.10 | Write MVP tests | Core relay functionality tested |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Server starts | `ws-relay --config basic.yaml` | "Listening on 0.0.0.0:8080" |
| Client connects | wscat to relay | "Client connected from {ip}" |
| Upstream connects | (automatic on client connect) | "Upstream connected to {url}" |
| Message relay C->U | Client sends "hello" | Upstream receives "hello" |
| Message relay U->C | Upstream sends "world" | Client receives "world" |
| Client close | Client sends close frame | Both connections closed |
| Upstream close | Upstream sends close frame | Client notified, both closed |
| Config error | Missing upstream URL | "Configuration error: upstream required" |

### Phase 1 Configuration

```yaml
# basic.yaml
server:
  bind: "0.0.0.0:8080"

upstream:
  url: "ws://localhost:9000/ws"
```

### Phase 1 Directory Structure

```
ws-relay/
├── ws_relay.ecf
├── src/
│   ├── relay_cli.e
│   ├── relay_server.e
│   ├── relay_session.e
│   └── relay_config.e
└── tests/
    ├── test_app.e
    ├── test_server.e
    └── test_session.e
```

---

## Phase 2: Full Implementation

### Objective

Complete relay with routing, logging, and filtering. Multiple upstreams supported. Tool is production-ready for basic use cases.

### Deliverables

1. **RELAY_ROUTER** - Path-based routing to multiple upstreams
2. **RELAY_POOL** - Upstream connection pooling
3. **RELAY_FILTER** - Message allow/deny filtering
4. **RELAY_LOGGER** - Structured traffic logging
5. **RELAY_HEALTH** - Health check endpoint

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T2.1 | Implement multiple upstreams | Config supports multiple backends |
| T2.2 | Implement RELAY_ROUTER | Path-based routing works |
| T2.3 | Implement RELAY_POOL | Connection reuse functional |
| T2.4 | Implement RELAY_FILTER | Pattern-based filtering |
| T2.5 | Implement RELAY_LOGGER | JSON structured logs |
| T2.6 | Implement RELAY_HEALTH | /health endpoint responds |
| T2.7 | Add concurrent session support | Multiple clients handled |
| T2.8 | Add graceful shutdown | Connection draining works |
| T2.9 | Add YAML config support | Full YAML config loaded |
| T2.10 | Write full test suite | All features tested |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Route by path | Connect to /api/v2/... | Routes to api-v2 upstream |
| Route default | Connect to /other | Routes to default upstream |
| Filter drop | Message matches drop pattern | Message not relayed, logged |
| Filter allow | Message matches allow pattern | Message relayed |
| Logging | Message relayed | Log entry with session, size, direction |
| Health check | GET /health | JSON status with upstream health |
| Concurrent | 10 simultaneous clients | All handled correctly |
| Shutdown | SIGTERM | Existing sessions complete, no new accepted |

### Phase 2 Configuration

```yaml
server:
  bind: "0.0.0.0:8080"
  max_connections: 1000

upstreams:
  default:
    url: "ws://backend:9000/ws"
  api-v2:
    url: "ws://backend-v2:9001/ws"

routing:
  - path: "/api/v2/*"
    upstream: api-v2
  - path: "/*"
    upstream: default

filters:
  - name: "block-admin"
    action: drop
    pattern:
      text_contains: "admin_command"

logging:
  level: info
  format: json

health:
  endpoint: "/health"
  port: 8081
```

---

## Phase 3: Production Polish

### Objective

Add message transformation, metrics endpoint, configuration hot reload, and production hardening. Tool is enterprise-ready.

### Deliverables

1. **RELAY_TRANSFORM** - Message modification rules
2. **RELAY_METRICS** - Prometheus-compatible metrics
3. **Config hot reload** - SIGHUP-triggered reload
4. **Error handling hardening** - All edge cases covered
5. **Performance optimization** - Efficient memory and CPU use

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T3.1 | Implement JSON transforms | Insert, remove, mask operations |
| T3.2 | Implement RELAY_METRICS | Counter and histogram metrics |
| T3.3 | Add Prometheus endpoint | /metrics returns valid format |
| T3.4 | Implement config hot reload | SIGHUP reloads without restart |
| T3.5 | Add config validation | Invalid config rejected with message |
| T3.6 | Harden error handling | No unhandled exceptions |
| T3.7 | Add PID file support | Process management enabled |
| T3.8 | Memory optimization | No leaks on long runs |
| T3.9 | Performance profiling | Latency overhead acceptable |
| T3.10 | Final documentation | README + config reference |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Transform insert | JSON message | Field added at path |
| Transform mask | JSON with token | Token field masked |
| Metrics endpoint | GET /metrics | Prometheus format metrics |
| Hot reload | SIGHUP | "Configuration reloaded" |
| Invalid reload | Bad config + SIGHUP | "Reload failed", old config kept |
| Memory test | 1 hour, 1000 connections | Memory stable |
| Latency test | 10000 msg/sec | < 5ms added latency |

---

## ECF Target Structure

```xml
<!-- Library target (reusable relay components) -->
<target name="ws_relay_lib">
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
            <exclude>/relay_cli\.e$</exclude>
        </file_rule>
    </cluster>
    <!-- Dependencies -->
    <library name="simple_websocket" location="$SIMPLE_EIFFEL\simple_websocket\simple_websocket.ecf"/>
    <library name="simple_http" location="$SIMPLE_EIFFEL\simple_http\simple_http.ecf"/>
    <library name="simple_json" location="$SIMPLE_EIFFEL\simple_json\simple_json.ecf"/>
    <library name="simple_yaml" location="$SIMPLE_EIFFEL\simple_yaml\simple_yaml.ecf"/>
    <library name="simple_logger" location="$SIMPLE_EIFFEL\simple_logger\simple_logger.ecf"/>
    <library name="simple_config" location="$SIMPLE_EIFFEL\simple_config\simple_config.ecf"/>
    <library name="base" location="$ISE_LIBRARY\library\base\base.ecf"/>
    <library name="net" location="$ISE_LIBRARY\library\net\net.ecf"/>
    <library name="thread" location="$ISE_LIBRARY\library\thread\thread.ecf"/>
</target>

<!-- Server executable target -->
<target name="ws_relay" extends="ws_relay_lib">
    <root class="RELAY_CLI" feature="make"/>
    <setting name="console_application" value="true"/>
</target>

<!-- Test target -->
<target name="ws_relay_tests" extends="ws_relay_lib">
    <root class="TEST_APP" feature="make"/>
    <library name="simple_testing" location="$SIMPLE_EIFFEL\simple_testing\simple_testing.ecf"/>
    <cluster name="tests" location=".\tests\" recursive="true"/>
</target>
```

---

## Build Commands

```bash
# Compile server (development)
/d/prod/ec.sh -batch -config ws_relay.ecf -target ws_relay -c_compile

# Compile server (finalized/optimized)
/d/prod/ec.sh -batch -config ws_relay.ecf -target ws_relay -finalize -c_compile

# Run tests
/d/prod/ec.sh -batch -config ws_relay.ecf -target ws_relay_tests -c_compile
./EIFGENs/ws_relay_tests/W_code/ws_relay.exe

# Run server
./EIFGENs/ws_relay/W_code/ws_relay.exe --config config.yaml
```

---

## Success Criteria

| Criterion | Measure | Target |
|-----------|---------|--------|
| Compiles | Zero errors | 100% |
| Tests pass | All test cases | 100% |
| Latency | Added latency per message | < 5ms |
| Throughput | Messages per second | > 10,000 |
| Memory | Per-connection overhead | < 100KB |
| Stability | Continuous operation | 7 days |
| Config reload | Zero downtime | 100% |
| Documentation | README + config reference | Complete |

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Thread complexity | Start single-threaded, add threading carefully |
| Memory leaks | Profile early and often |
| Config complexity | Start simple, add features incrementally |
| Upstream failures | Implement circuit breaker pattern |
| High connection count | Test with realistic loads early |
