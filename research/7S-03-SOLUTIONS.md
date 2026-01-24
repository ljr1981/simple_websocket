# 7S-03 SOLUTIONS - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## Alternative Approaches Considered

### 1. External WebSocket Library
**Description**: Use existing C/C++ WebSocket library
**Pros**: Mature, tested
**Cons**: Integration complexity, dependency
**Decision**: Rejected - prefer pure Eiffel

### 2. Socket.IO Protocol
**Description**: Implement Socket.IO instead of raw WebSocket
**Pros**: Higher-level, more features
**Cons**: Complex, non-standard
**Decision**: Rejected - WebSocket is standard

### 3. Pure Eiffel Implementation (Selected)
**Description**: Implement RFC 6455 in pure Eiffel
**Pros**: No dependencies, full control, educational
**Cons**: More work, needs thorough testing
**Decision**: Selected - best for simple_* ecosystem

## Implementation Strategy

1. **WS_FRAME**: Single frame encoding/decoding
2. **WS_FRAME_PARSER**: Streaming frame parser
3. **WS_MESSAGE**: Complete message (may span frames)
4. **WS_HANDSHAKE**: HTTP upgrade handshake

## Technology Stack

- **simple_base64**: Base64 encoding for keys
- **simple_hash**: SHA-1 for Sec-WebSocket-Accept
- **RANDOM**: Generate masking keys
- **ARRAY [NATURAL_8]**: Binary data handling
