# 7S-02 STANDARDS - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## Applicable Standards

### WebSocket Protocol
- **RFC 6455**: The WebSocket Protocol (primary reference)
- **RFC 7936**: Clarifications for RFC 6455

### Related Standards
- **RFC 4648**: Base64 encoding
- **RFC 3174**: SHA-1 hash (for Sec-WebSocket-Accept)

## Frame Format (RFC 6455 Section 5.2)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
|N|V|V|V|       |S|             |   (if payload len==126/127)   |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
|     Extended payload length continued, if payload len == 127  |
+ - - - - - - - - - - - - - - - +-------------------------------+
|                               |Masking-key, if MASK set to 1  |
+-------------------------------+-------------------------------+
| Masking-key (continued)       |          Payload Data         |
+-------------------------------- - - - - - - - - - - - - - - - +
:                     Payload Data continued ...                :
+ - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
|                     Payload Data continued ...                |
+---------------------------------------------------------------+
```

## Opcodes (RFC 6455 Section 5.2)

| Opcode | Value | Description |
|--------|-------|-------------|
| Continuation | 0x0 | Continuation frame |
| Text | 0x1 | Text frame (UTF-8) |
| Binary | 0x2 | Binary frame |
| Close | 0x8 | Connection close |
| Ping | 0x9 | Ping frame |
| Pong | 0xA | Pong frame |

## Close Codes (RFC 6455 Section 7.4.1)

| Code | Constant | Description |
|------|----------|-------------|
| 1000 | Close_normal | Normal closure |
| 1001 | Close_going_away | Going away |
| 1002 | Close_protocol_error | Protocol error |
| 1003 | Close_unsupported_data | Unsupported data |
| 1007 | Close_invalid_payload | Invalid payload |
| 1008 | Close_policy_violation | Policy violation |
| 1009 | Close_message_too_big | Message too big |
| 1011 | Close_server_error | Server error |

## Handshake Magic String

```
258EAFA5-E914-47DA-95CA-C5AB0DC85B11
```
