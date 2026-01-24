# S01 PROJECT INVENTORY - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## Project Structure

```
simple_websocket/
  simple_websocket.ecf      # Eiffel configuration file
  src/
    ws_frame.e              # WebSocket frame
    ws_frame_parser.e       # Streaming frame parser
    ws_message.e            # Complete message
    ws_handshake.e          # HTTP upgrade handshake
  testing/
    test_app.e              # Test application entry
    lib_tests.e             # Test cases
  research/                 # 7S research documents
  specs/                    # Specification documents
```

## Source Files

| File | Type | Lines | Description |
|------|------|-------|-------------|
| ws_frame.e | Class | ~360 | Frame encode/decode |
| ws_frame_parser.e | Class | ~215 | Streaming parser |
| ws_message.e | Class | ~145 | Complete message |
| ws_handshake.e | Class | ~305 | Handshake handling |

## Test Files

| File | Type | Tests | Description |
|------|------|-------|-------------|
| test_app.e | Root | - | Test application entry point |
| lib_tests.e | Tests | TBD | Library test cases |

## Dependencies

### Internal (simple_* ecosystem)
- simple_base64 (Key encoding)
- simple_hash (SHA-1 for Accept)

### Standard Library
- base (ARRAY, ARRAYED_LIST, RANDOM, STRING)

## Build Targets

| Target | Type | Description |
|--------|------|-------------|
| simple_websocket | Library | Main library target |
| simple_websocket_tests | Executable | Test runner |
