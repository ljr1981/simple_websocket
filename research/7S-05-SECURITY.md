# 7S-05 SECURITY - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## Security Considerations

### Handshake Security

#### Sec-WebSocket-Key
- **Purpose**: Prevent caching proxies from replaying
- **Implementation**: Random 16-byte base64 key
- **Status**: Correctly implemented

#### Sec-WebSocket-Accept
- **Purpose**: Prove server understands WebSocket
- **Implementation**: SHA-1(key + GUID), base64
- **Status**: Correctly implemented

### Frame Security

#### Masking (Client to Server)
- **Purpose**: Prevent cache poisoning attacks
- **Requirement**: All client frames MUST be masked
- **Status**: Supported, caller responsibility to mask

#### Unmasking (Server receives)
- **Parser**: Automatically unmasks if MASK bit set
- **Status**: Correctly implemented

### Payload Validation

#### Close Frame
- **Risk**: Invalid close codes
- **Current**: Accepts any code in range
- **Recommendation**: Validate known codes

#### Text Frames
- **Risk**: Invalid UTF-8
- **Current**: No UTF-8 validation
- **Recommendation**: Add UTF-8 validation option

## Security Boundaries

| Boundary | Protection |
|----------|------------|
| Handshake | Key validation |
| Masking | Parser handles |
| Payload | None (caller validates) |
| Close codes | Range check only |

## Threat Model

1. **Replay attacks**: Sec-WebSocket-Key prevents
2. **Cache poisoning**: Masking prevents
3. **Invalid frames**: Parser validates structure
4. **Malicious payloads**: Caller responsibility
