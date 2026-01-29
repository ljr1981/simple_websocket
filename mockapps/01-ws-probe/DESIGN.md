# WS-PROBE - Technical Design

## Architecture

### Component Overview

```
+----------------------------------------------------------+
|                       WS-PROBE                            |
+----------------------------------------------------------+
|  CLI Interface Layer                                      |
|    - PROBE_CLI: Argument parsing, command routing         |
|    - PROBE_REPL: Interactive session handler              |
|    - PROBE_OUTPUT: Formatted output (text/JSON)           |
+----------------------------------------------------------+
|  Business Logic Layer                                     |
|    - PROBE_SESSION: Connection lifecycle management       |
|    - PROBE_MESSENGER: Message send/receive operations     |
|    - PROBE_VALIDATOR: Response assertion/validation       |
|    - PROBE_RECORDER: Session recording to file            |
+----------------------------------------------------------+
|  Protocol Layer                                           |
|    - WS_HANDSHAKE: HTTP upgrade handshake                 |
|    - WS_FRAME: Frame encoding/decoding                    |
|    - WS_FRAME_PARSER: Streaming frame parser              |
|    - WS_MESSAGE: Message fragmentation                    |
+----------------------------------------------------------+
|  Transport Layer (External)                               |
|    - EiffelNet SOCKET: TCP connection                     |
+----------------------------------------------------------+
```

### Class Design

| Class | Responsibility | Key Features |
|-------|----------------|--------------|
| PROBE_CLI | Command-line interface | parse_args, route_command, show_help |
| PROBE_SESSION | Connection lifecycle | connect, disconnect, is_connected, send, receive |
| PROBE_MESSENGER | High-level messaging | send_text, send_binary, send_file, receive_with_timeout |
| PROBE_VALIDATOR | Response validation | expect_text, expect_json, assert_contains |
| PROBE_RECORDER | Session recording | start_recording, stop_recording, write_message |
| PROBE_OUTPUT | Output formatting | format_text, format_json, format_message |
| PROBE_REPL | Interactive mode | run_loop, handle_input, display_message |
| PROBE_CONFIG | Connection profiles | load_profile, save_profile, list_profiles |

### Command Structure

```bash
ws-probe <command> [options] [arguments]

Commands:
  connect <url>         Connect to WebSocket endpoint
  send <message>        Send text message (requires connection)
  send-binary <file>    Send binary file contents
  receive               Wait for and display next message
  ping                  Send ping frame
  close [code] [reason] Close connection gracefully
  interactive <url>     Start interactive REPL session
  profile <subcommand>  Manage connection profiles

Global Options:
  --config FILE         Configuration file path
  --profile NAME        Use named connection profile
  --output FORMAT       Output format: text (default), json
  --timeout MS          Operation timeout in milliseconds
  --verbose             Verbose output (show frames)
  --quiet               Suppress non-essential output
  --help                Show help message
  --version             Show version information

Connect Options:
  --header KEY:VALUE    Add custom HTTP header (repeatable)
  --subprotocol NAME    Request WebSocket subprotocol
  --origin URL          Set Origin header

Send Options:
  --binary              Treat message as binary
  --no-mask             Disable client masking (non-compliant)
  --fragment SIZE       Fragment message into SIZE-byte frames

Receive Options:
  --expect PATTERN      Assert response matches pattern
  --expect-json PATH    Assert JSON path exists/matches
  --count N             Receive N messages then exit

Interactive Options:
  --record FILE         Record session to file
  --script FILE         Execute commands from script file
```

### Data Flow

```
User Input
    |
    v
+-------------------+
| PROBE_CLI         |  Parse arguments, validate
+-------------------+
    |
    v
+-------------------+
| PROBE_SESSION     |  Manage connection state
+-------------------+
    |
    v
+-------------------+
| PROBE_MESSENGER   |  High-level send/receive
+-------------------+
    |
    v
+-------------------+
| WS_FRAME          |  Encode/decode frames
| WS_HANDSHAKE      |  HTTP upgrade
| WS_FRAME_PARSER   |  Parse byte stream
+-------------------+
    |
    v
+-------------------+
| SOCKET (EiffelNet)|  TCP I/O
+-------------------+
    |
    v
Output (text/JSON) -> stdout
Errors -> stderr
Exit code -> 0 (success), 1 (error), 2 (timeout)
```

### Configuration Schema

```json
{
  "ws-probe": {
    "default_timeout": 5000,
    "output_format": "text",
    "profiles": {
      "local-dev": {
        "url": "ws://localhost:8080/ws",
        "headers": {
          "Authorization": "Bearer dev-token"
        }
      },
      "production": {
        "url": "wss://api.example.com/ws",
        "subprotocol": "graphql-ws",
        "timeout": 10000
      }
    }
  }
}
```

### Error Handling

| Error Type | Handling | User Message | Exit Code |
|------------|----------|--------------|-----------|
| Connection refused | Immediate fail | "Connection refused: {host}:{port}" | 1 |
| Handshake failure | Report reason | "Handshake failed: {reason}" | 1 |
| Connection timeout | Configurable | "Connection timeout after {ms}ms" | 2 |
| Read timeout | Configurable | "Read timeout after {ms}ms" | 2 |
| Invalid URL | Immediate fail | "Invalid WebSocket URL: {url}" | 1 |
| Protocol error | Close connection | "Protocol error: {details}" | 1 |
| Expect failure | Report mismatch | "Expected: {pattern}, Got: {actual}" | 1 |
| Server close | Report code | "Server closed: {code} {reason}" | 0 |

### State Machine

```
            +--------+
            | IDLE   |
            +--------+
                |
                | connect()
                v
         +-------------+
         | CONNECTING  |
         +-------------+
                |
    +-----------+-----------+
    |                       |
    v                       v
+--------+            +--------+
| OPEN   |            | FAILED |
+--------+            +--------+
    |
    +---> send() / receive()
    |
    | close()
    v
+----------+
| CLOSING  |
+----------+
    |
    v
+--------+
| CLOSED |
+--------+
```

## GUI/TUI Future Path

**CLI foundation enables:**

1. **TUI Migration:** The `PROBE_REPL` class can be extended to use simple_tui for:
   - Split-pane view (sent/received messages)
   - Real-time message highlighting
   - Connection status bar
   - Command history navigation

2. **Shared Components:**
   - `PROBE_SESSION`: Connection management unchanged
   - `PROBE_MESSENGER`: Send/receive logic reusable
   - `PROBE_OUTPUT`: Add TUI-specific formatters

3. **GUI Future:**
   - Message list with filtering
   - Syntax-highlighted JSON viewer
   - Connection tree (multiple sessions)
   - Request builder with templates

## Sample Usage

```bash
# Basic connection and message
ws-probe connect ws://localhost:8080/chat
ws-probe send "Hello, WebSocket!"
ws-probe receive
ws-probe close

# One-liner with expect
ws-probe connect ws://api.example.com/ws && \
ws-probe send '{"type":"ping"}' && \
ws-probe receive --expect-json "$.type=pong"

# CI/CD integration with JSON output
ws-probe interactive wss://api.example.com/ws \
  --script test-script.txt \
  --output json \
  --timeout 10000 > results.json

# Interactive session with recording
ws-probe interactive ws://localhost:8080/chat \
  --record session.log

# Using connection profile
ws-probe profile use production
ws-probe connect
ws-probe send '{"subscribe":"ticker"}'
```
