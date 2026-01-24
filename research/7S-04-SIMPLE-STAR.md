# 7S-04 SIMPLE-STAR - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## Ecosystem Dependencies

### Required Libraries
| Library | Purpose | Usage |
|---------|---------|-------|
| simple_base64 | Base64 encoding | Handshake keys |
| simple_hash | SHA-1 hashing | Sec-WebSocket-Accept |

### Standard Library
| Component | Purpose |
|-----------|---------|
| ARRAY [NATURAL_8] | Binary data |
| ARRAYED_LIST | Frame buffer |
| RANDOM | Mask key generation |
| STRING | Text payloads |

## Integration Points

### Client Handshake
```eiffel
create handshake.make
request := handshake.create_client_request ("example.com", "/chat")
-- Send request, receive response
if handshake.validate_server_response (response) then
    -- WebSocket connection established
end
```

### Server Handshake
```eiffel
create handshake.make
if handshake.parse_client_request (request) then
    response := handshake.create_server_response
    -- Send response, connection established
end
```

### Frame Handling
```eiffel
-- Send text frame
create frame.make_text ("Hello", True)
frame.set_mask (random_key)
bytes := frame.to_bytes

-- Parse incoming frame
create parser.make
parser.add_bytes (received_data)
if parser.parse then
    if attached parser.last_frame as f then
        if f.is_text then
            print (f.text_payload)
        end
    end
end
```

## Ecosystem Position

simple_websocket provides protocol layer for:
- Real-time chat applications
- Live notification systems
- Streaming data applications
- simple_web WebSocket upgrade
