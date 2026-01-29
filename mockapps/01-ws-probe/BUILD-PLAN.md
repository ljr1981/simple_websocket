# WS-PROBE - Build Plan

## Phase Overview

| Phase | Deliverable | Effort | Dependencies |
|-------|-------------|--------|--------------|
| Phase 1 | MVP CLI - Connect, Send, Receive | 3 days | simple_websocket, simple_cli |
| Phase 2 | Full CLI - All commands, JSON output | 4 days | Phase 1 + simple_json, simple_logger |
| Phase 3 | Polish - Profiles, validation, docs | 3 days | Phase 2 + simple_config |

---

## Phase 1: MVP

### Objective

Demonstrate core WebSocket connectivity via CLI. User can connect to a WebSocket endpoint, send a text message, receive a response, and close the connection. Basic interactive mode functional.

### Deliverables

1. **PROBE_CLI** - Basic argument parsing for connect, send, receive, close
2. **PROBE_SESSION** - Connection lifecycle (connect, disconnect, is_connected)
3. **PROBE_MESSENGER** - send_text, receive operations
4. **PROBE_OUTPUT** - Text output formatting
5. **PROBE_REPL** - Basic interactive mode

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T1.1 | Create project structure | ECF compiles, TEST_APP runs |
| T1.2 | Implement PROBE_SESSION.connect | Can establish WS connection |
| T1.3 | Implement PROBE_SESSION.disconnect | Clean connection close |
| T1.4 | Implement PROBE_MESSENGER.send_text | Text frame transmitted |
| T1.5 | Implement PROBE_MESSENGER.receive | Frame received and decoded |
| T1.6 | Implement PROBE_CLI argument parsing | Commands recognized |
| T1.7 | Implement PROBE_OUTPUT.format_text | Human-readable output |
| T1.8 | Implement PROBE_REPL basic loop | Interactive session works |
| T1.9 | Add timeout handling | Connect/read timeouts configurable |
| T1.10 | Write MVP tests | Core operations tested |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| Connect success | `ws-probe connect ws://echo.websocket.org` | "Connected to ws://echo.websocket.org" |
| Connect failure | `ws-probe connect ws://invalid.host` | "Connection refused: invalid.host:80", exit 1 |
| Send message | `ws-probe send "hello"` (connected) | "Sent: hello" |
| Send not connected | `ws-probe send "hello"` (not connected) | "Not connected", exit 1 |
| Receive message | `ws-probe receive` after server sends | "Received: {message}" |
| Receive timeout | `ws-probe receive --timeout 1000` | "Read timeout after 1000ms", exit 2 |
| Close connection | `ws-probe close` | "Connection closed" |
| Interactive mode | `ws-probe interactive ws://echo.websocket.org` | REPL prompt, echo works |

### Phase 1 Directory Structure

```
ws-probe/
├── ws_probe.ecf
├── src/
│   ├── probe_cli.e
│   ├── probe_session.e
│   ├── probe_messenger.e
│   ├── probe_output.e
│   └── probe_repl.e
└── tests/
    ├── test_app.e
    ├── test_session.e
    └── test_messenger.e
```

---

## Phase 2: Full Implementation

### Objective

Complete CLI with all commands, JSON output, logging, and binary message support. Tool is scriptable and CI/CD-ready.

### Deliverables

1. **PROBE_CLI** - All commands implemented (send-binary, ping, profile)
2. **PROBE_OUTPUT** - JSON output mode
3. **PROBE_VALIDATOR** - Response assertion (--expect)
4. **PROBE_RECORDER** - Session recording
5. **Logging integration** - Debug and audit logging

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T2.1 | Implement send-binary command | Binary files transmitted |
| T2.2 | Implement ping command | Ping/pong cycle works |
| T2.3 | Implement JSON output format | --output json produces valid JSON |
| T2.4 | Implement PROBE_VALIDATOR | --expect patterns validated |
| T2.5 | Add custom headers | --header option works |
| T2.6 | Add subprotocol support | --subprotocol negotiation |
| T2.7 | Implement session recording | --record captures all messages |
| T2.8 | Integrate simple_logger | Debug logging functional |
| T2.9 | Add script execution | --script runs command file |
| T2.10 | Write full test suite | All commands tested |

### Test Cases

| Test | Input | Expected Output |
|------|-------|-----------------|
| JSON output | `ws-probe receive --output json` | Valid JSON with type, payload |
| Binary send | `ws-probe send-binary image.png` | Binary frame sent |
| Ping test | `ws-probe ping` | "Pong received" |
| Expect match | `ws-probe receive --expect "ok"` | Exit 0 if matches |
| Expect fail | `ws-probe receive --expect "ok"` (got "error") | "Expected: ok, Got: error", exit 1 |
| Custom header | `ws-probe connect ws://... --header "Auth:token"` | Header sent in handshake |
| Recording | `ws-probe interactive ... --record log.txt` | Log file contains messages |

---

## Phase 3: Production Polish

### Objective

Add connection profiles, comprehensive error handling, documentation, and performance optimization. Tool is production-ready.

### Deliverables

1. **PROBE_CONFIG** - Connection profile management
2. **Error handling hardening** - All edge cases covered
3. **Help documentation** - All commands documented
4. **Configuration validation** - Invalid config detected early
5. **Performance optimization** - Efficient memory use

### Tasks

| Task | Description | Acceptance Criteria |
|------|-------------|---------------------|
| T3.1 | Implement profile save/load | Profiles persist and load |
| T3.2 | Add profile list/show commands | Users can manage profiles |
| T3.3 | Harden error handling | No unhandled exceptions |
| T3.4 | Add --help for all commands | Help text complete |
| T3.5 | Add man page generation | Man page produced |
| T3.6 | Configuration validation | Invalid config fails early |
| T3.7 | Memory optimization | No leaks on long sessions |
| T3.8 | Performance profiling | Response time acceptable |
| T3.9 | Final documentation | README complete |
| T3.10 | Release preparation | Version tagged, changelog |

---

## ECF Target Structure

```xml
<!-- Library target (reusable components) -->
<target name="ws_probe_lib">
    <option warning="warning" syntax="standard">
        <assertions precondition="true" postcondition="true" check="true"/>
    </option>
    <capability>
        <void_safety support="all"/>
    </capability>
    <cluster name="src" location=".\src\" recursive="true">
        <file_rule>
            <exclude>/probe_cli\.e$</exclude>
        </file_rule>
    </cluster>
    <!-- Dependencies -->
    <library name="simple_websocket" location="$SIMPLE_EIFFEL\simple_websocket\simple_websocket.ecf"/>
    <library name="simple_json" location="$SIMPLE_EIFFEL\simple_json\simple_json.ecf"/>
    <library name="simple_cli" location="$SIMPLE_EIFFEL\simple_cli\simple_cli.ecf"/>
    <library name="simple_logger" location="$SIMPLE_EIFFEL\simple_logger\simple_logger.ecf"/>
    <library name="simple_config" location="$SIMPLE_EIFFEL\simple_config\simple_config.ecf"/>
    <library name="base" location="$ISE_LIBRARY\library\base\base.ecf"/>
    <library name="net" location="$ISE_LIBRARY\library\net\net.ecf"/>
</target>

<!-- CLI executable target -->
<target name="ws_probe" extends="ws_probe_lib">
    <root class="PROBE_CLI" feature="make"/>
    <setting name="console_application" value="true"/>
    <cluster name="cli" location=".\src\">
        <file_rule>
            <include>/probe_cli\.e$</include>
        </file_rule>
    </cluster>
</target>

<!-- Test target -->
<target name="ws_probe_tests" extends="ws_probe_lib">
    <root class="TEST_APP" feature="make"/>
    <library name="simple_testing" location="$SIMPLE_EIFFEL\simple_testing\simple_testing.ecf"/>
    <cluster name="tests" location=".\tests\" recursive="true"/>
</target>
```

---

## Build Commands

```bash
# Compile CLI (development)
/d/prod/ec.sh -batch -config ws_probe.ecf -target ws_probe -c_compile

# Compile CLI (finalized/optimized)
/d/prod/ec.sh -batch -config ws_probe.ecf -target ws_probe -finalize -c_compile

# Run tests
/d/prod/ec.sh -batch -config ws_probe.ecf -target ws_probe_tests -c_compile
./EIFGENs/ws_probe_tests/W_code/ws_probe.exe

# Run finalized tests
/d/prod/ec.sh -batch -config ws_probe.ecf -target ws_probe_tests -finalize -c_compile
./EIFGENs/ws_probe_tests/F_code/ws_probe.exe
```

---

## Success Criteria

| Criterion | Measure | Target |
|-----------|---------|--------|
| Compiles | Zero errors | 100% |
| Tests pass | All test cases | 100% |
| CLI works | All commands functional | 100% |
| Exit codes | Correct for all scenarios | 100% |
| JSON valid | All JSON output parseable | 100% |
| Documentation | README + help complete | 100% |
| Performance | Connect < 500ms local | Pass |
| Memory | No leaks on 1h session | Pass |

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| EiffelNet socket limitations | Test early, have fallback plan |
| TLS complexity | Start with ws://, add wss:// incrementally |
| Large message handling | Test fragmentation early |
| Platform differences | Test on Windows primary |
