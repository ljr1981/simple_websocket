# S02 CLASS CATALOG - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## Class Hierarchy

```
WS_FRAME
  - WebSocket frame encode/decode
  - No inheritance

WS_FRAME_PARSER
  - Streaming frame parser
  - No inheritance

WS_MESSAGE
  - Complete message wrapper
  - No inheritance

WS_HANDSHAKE
  - HTTP upgrade handshake
  - No inheritance
```

## Class Descriptions

### WS_FRAME

**Purpose**: Encode and decode WebSocket frames per RFC 6455

**Responsibilities**:
- Create frames of all types (text, binary, close, ping, pong)
- Encode frames to wire format
- Store and apply masking keys
- Extract payload data

**Key Features**:
- `make`, `make_text`, `make_binary`, `make_close`, `make_ping`, `make_pong`
- `to_bytes`: Encode to wire format
- `opcode`, `payload`, `is_fin`, `is_masked`, `mask_key`
- Type queries: `is_text`, `is_binary`, `is_close`, `is_ping`, `is_pong`
- `text_payload`, `close_code`, `close_reason`

### WS_FRAME_PARSER

**Purpose**: Parse WebSocket frames from byte stream

**Responsibilities**:
- Accumulate incoming bytes
- Parse complete frames
- Handle extended length formats
- Unmask payloads

**Key Features**:
- `add_bytes`: Feed data to parser
- `parse`: Attempt to parse frame
- `has_frame`, `last_frame`: Access parsed frame
- `has_error`, `last_error`: Error handling
- `reset`: Clear parser state

### WS_MESSAGE

**Purpose**: Represent a complete WebSocket message

**Responsibilities**:
- Store text or binary message
- Convert to/from frames
- Support message fragmentation

**Key Features**:
- `make_text`, `make_binary`
- `is_text`, `is_binary`, `text`, `data`, `size`
- `to_frame`: Single frame conversion
- `to_frames`: Fragmented conversion

### WS_HANDSHAKE

**Purpose**: Handle WebSocket HTTP upgrade handshake

**Responsibilities**:
- Generate client handshake request
- Validate server handshake response
- Parse client handshake request (server side)
- Generate server handshake response
- Compute Sec-WebSocket-Accept

**Key Features**:
- `create_client_request`: Client initiates
- `validate_server_response`: Client validates
- `parse_client_request`: Server parses
- `create_server_response`: Server responds
- `sec_websocket_key`, `sec_websocket_accept`
