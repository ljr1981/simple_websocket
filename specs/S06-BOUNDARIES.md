# S06 BOUNDARIES - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## System Boundaries

```
+--------------------------------------------------+
|                    Client Code                    |
|  (Socket layer, TLS handling, reconnection)       |
+--------------------------------------------------+
                        |
                        v
+--------------------------------------------------+
|                simple_websocket                   |
|  +------------+  +--------+  +-----------------+ |
|  | WS_FRAME   |  | PARSER |  | WS_HANDSHAKE    | |
|  +------------+  +--------+  +-----------------+ |
|  | WS_MESSAGE |                                  |
|  +------------+                                  |
+--------------------------------------------------+
                        |
                        v
+--------------------------------------------------+
|              simple_base64, simple_hash           |
|              (Key encoding, SHA-1)                |
+--------------------------------------------------+
```

## Interface Boundaries

### Public Interface
- WS_FRAME: Frame encode/decode
- WS_FRAME_PARSER: Streaming parser
- WS_MESSAGE: Message wrapper
- WS_HANDSHAKE: Upgrade handshake

### Internal (Not Exposed)
- Mask application (private feature)
- Length encoding (to_bytes internal)
- Header parsing (parse internal)

## Data Boundaries

### Input Data
| Data | Source | Validation |
|------|--------|------------|
| Payload bytes | Client | None |
| Mask key | Client | Length = 4 |
| Close code | Client | Range 1000-4999 |
| Received bytes | Network | Parser validates |

### Output Data
| Data | Destination | Format |
|------|-------------|--------|
| Encoded frame | Network | ARRAY [NATURAL_8] |
| Parsed frame | Client | WS_FRAME |
| Handshake | Network | STRING (HTTP) |

## Trust Boundaries

| Zone | Trust Level | Validation |
|------|-------------|------------|
| Frame construction | Trusted | Preconditions |
| Incoming bytes | Untrusted | Parser validates |
| Handshake headers | Semi-trusted | Header checks |
| Payload content | Untrusted | Client validates |
