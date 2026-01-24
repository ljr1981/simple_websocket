# S05 CONSTRAINTS - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## Technical Constraints

### Platform
- **Target OS**: Cross-platform (pure Eiffel)
- **Compiler**: EiffelStudio 25.02+
- **Void Safety**: Required

### Protocol
- RFC 6455 compliant
- No extensions (deflate-frame, etc.)
- Basic subprotocol support only

## Frame Constraints

### Opcodes
- Valid: 0x0, 0x1, 0x2, 0x8, 0x9, 0xA
- Reserved: 0x3-0x7, 0xB-0xF (rejected)

### RSV Bits
- Must be 0 (no extensions)
- Non-zero triggers parse error

### Payload Length
| Length | Encoding |
|--------|----------|
| 0-125 | 7-bit |
| 126-65535 | 16-bit extended |
| 65536+ | 64-bit extended |

### Masking
- Client-to-server MUST be masked
- Server-to-client MUST NOT be masked
- XOR with 4-byte key

## Close Frame Constraints

### Close Codes
| Range | Description |
|-------|-------------|
| 1000-1015 | RFC defined |
| 3000-3999 | IANA registered |
| 4000-4999 | Private use |

### Close Payload
- First 2 bytes: Status code (network order)
- Remainder: UTF-8 reason (optional)

## Handshake Constraints

### Required Headers (Client)
- Upgrade: websocket
- Connection: Upgrade
- Sec-WebSocket-Key: (16 random bytes, base64)
- Sec-WebSocket-Version: 13

### Required Headers (Server)
- Upgrade: websocket
- Connection: Upgrade
- Sec-WebSocket-Accept: (computed from key)

### Accept Key Computation
```
Base64(SHA-1(key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"))
```

## Performance Constraints

| Operation | Complexity |
|-----------|------------|
| Frame encode | O(payload) |
| Frame parse | O(data) |
| Handshake | O(header size) |
