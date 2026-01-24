# S07 SPEC SUMMARY - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## Executive Summary

simple_websocket provides RFC 6455 compliant WebSocket protocol implementation for Eiffel. The library handles framing, masking, and handshake, providing building blocks for WebSocket clients and servers.

## Key Specifications

### Classes
| Class | Purpose | Complexity |
|-------|---------|------------|
| WS_FRAME | Frame encode/decode | Medium |
| WS_FRAME_PARSER | Streaming parser | High |
| WS_MESSAGE | Message wrapper | Low |
| WS_HANDSHAKE | HTTP upgrade | Medium |

### Features by Category
| Category | Count | Key Features |
|----------|-------|--------------|
| Frame constructors | 6 | make, make_text, make_close, etc. |
| Frame access | 12 | opcode, payload, text_payload, etc. |
| Parser | 7 | add_bytes, parse, last_frame |
| Message | 6 | make_text, to_frame, to_frames |
| Handshake | 6 | create_client_request, validate, etc. |

### Contract Coverage
| Contract Type | Count |
|--------------|-------|
| Preconditions | 20+ |
| Postconditions | 15+ |
| Invariants | 7 |

## Dependencies

### Required
- simple_base64 (Key encoding)
- simple_hash (SHA-1)

## Quality Metrics

| Metric | Value |
|--------|-------|
| Source lines | ~1025 |
| Classes | 4 |
| Features | ~50 |
| Test coverage | Present |

## API Summary

```eiffel
-- Create and encode text frame
create frame.make_text ("Hello", True)
frame.set_mask (random_key)
bytes := frame.to_bytes

-- Parse incoming frame
create parser.make
parser.add_bytes (network_data)
if parser.parse and attached parser.last_frame as f then
    if f.is_text then
        print (f.text_payload)
    end
end

-- Client handshake
create handshake.make
request := handshake.create_client_request ("host", "/path")
if handshake.validate_server_response (response) then
    -- Connected
end
```

## Status

**Phase**: 2 (Expanded Features)
**Stability**: Stable
**Production Ready**: Yes (protocol layer)
