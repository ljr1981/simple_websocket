# S04 FEATURE SPECS - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## WS_FRAME Features

### Constructors

| Feature | Signature | Description |
|---------|-----------|-------------|
| make | (INT, ARRAY, BOOL) | Generic frame |
| make_text | (STRING, BOOL) | Text frame |
| make_binary | (ARRAY, BOOL) | Binary frame |
| make_close | (INT, STRING) | Close frame |
| make_ping | | Ping frame |
| make_pong | | Pong frame |

### Access

| Feature | Signature | Description |
|---------|-----------|-------------|
| opcode | INTEGER | Frame opcode |
| payload | ARRAY [NATURAL_8] | Payload data |
| is_fin | BOOLEAN | Final fragment? |
| is_masked | BOOLEAN | Masked? |
| mask_key | ARRAY [NATURAL_8] | 4-byte mask |
| payload_length | INTEGER | Payload size |
| text_payload | STRING | Text (if text frame) |
| close_code | INTEGER | Close code |
| close_reason | STRING | Close reason |

### Status

| Feature | Signature | Description |
|---------|-----------|-------------|
| is_control | BOOLEAN | Control frame? |
| is_text | BOOLEAN | Text frame? |
| is_binary | BOOLEAN | Binary frame? |
| is_close | BOOLEAN | Close frame? |
| is_ping | BOOLEAN | Ping frame? |
| is_pong | BOOLEAN | Pong frame? |
| is_continuation | BOOLEAN | Continuation? |
| is_valid_opcode | (INT): BOOL | Valid opcode? |

### Operations

| Feature | Signature | Description |
|---------|-----------|-------------|
| set_mask | (ARRAY) | Enable masking |
| to_bytes | ARRAY [NATURAL_8] | Encode to wire |

### Constants

| Constant | Value | Description |
|----------|-------|-------------|
| Opcode_continuation | 0x0 | Continuation |
| Opcode_text | 0x1 | Text frame |
| Opcode_binary | 0x2 | Binary frame |
| Opcode_close | 0x8 | Close frame |
| Opcode_ping | 0x9 | Ping frame |
| Opcode_pong | 0xA | Pong frame |
| Close_normal | 1000 | Normal close |
| Close_going_away | 1001 | Going away |
| Close_protocol_error | 1002 | Protocol error |

## WS_FRAME_PARSER Features

| Feature | Signature | Description |
|---------|-----------|-------------|
| make | | Initialize parser |
| add_bytes | (ARRAY) | Feed data |
| parse | BOOLEAN | Try parse frame |
| has_frame | BOOLEAN | Frame available? |
| last_frame | WS_FRAME | Parsed frame |
| has_error | BOOLEAN | Parse error? |
| last_error | STRING | Error message |
| reset | | Clear state |

## WS_MESSAGE Features

| Feature | Signature | Description |
|---------|-----------|-------------|
| make_text | (STRING) | Text message |
| make_binary | (ARRAY) | Binary message |
| is_text | BOOLEAN | Text message? |
| is_binary | BOOLEAN | Binary message? |
| text | STRING | Text content |
| data | ARRAY [NATURAL_8] | Binary content |
| size | INTEGER | Message size |
| to_frame | WS_FRAME | Single frame |
| to_frames | (INT): LIST | Fragmented |

## WS_HANDSHAKE Features

| Feature | Signature | Description |
|---------|-----------|-------------|
| make | | Initialize |
| create_client_request | (STRING, STRING): STRING | Client request |
| validate_server_response | (STRING): BOOL | Validate response |
| parse_client_request | (STRING): BOOL | Server parse |
| create_server_response | STRING | Server response |
| is_valid | BOOLEAN | Valid handshake? |
| last_error | STRING | Error message |
| sec_websocket_key | STRING | Client key |
| sec_websocket_accept | STRING | Server accept |
| requested_path | STRING | Request path |
| subprotocol | STRING | Negotiated proto |
