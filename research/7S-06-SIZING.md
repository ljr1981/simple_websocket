# 7S-06 SIZING - simple_websocket


**Date**: 2026-01-23

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## Codebase Metrics

### Source Files
| File | Lines | Purpose |
|------|-------|---------|
| ws_frame.e | ~360 | Frame encode/decode |
| ws_frame_parser.e | ~215 | Streaming parser |
| ws_message.e | ~145 | Complete message |
| ws_handshake.e | ~305 | HTTP upgrade |
| **Total** | ~1025 | |

### Class Count
- 4 production classes
- 2 test classes (test_app.e, lib_tests.e)

## Complexity Analysis

### WS_FRAME
- Features: ~25
- Cyclomatic complexity: Medium
- Encoding logic, multiple constructors

### WS_FRAME_PARSER
- Features: ~10
- Cyclomatic complexity: High
- State machine for parsing

### WS_MESSAGE
- Features: ~8
- Cyclomatic complexity: Low
- Simple wrapper

### WS_HANDSHAKE
- Features: ~15
- Cyclomatic complexity: Medium
- String parsing

## Memory Footprint

| Component | Typical Size |
|-----------|-------------|
| WS_FRAME | ~100 bytes + payload |
| WS_FRAME_PARSER | ~100 bytes + buffer |
| WS_MESSAGE | ~50 bytes + data |
| WS_HANDSHAKE | ~200 bytes |

## Performance Characteristics

| Operation | Expected Time |
|-----------|--------------|
| Frame encode | O(payload size) |
| Frame parse | O(data size) |
| Handshake create | O(1) |
| Handshake validate | O(response size) |
