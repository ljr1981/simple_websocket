# 7S-01 SCOPE - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket
**Purpose**: WebSocket protocol implementation per RFC 6455

## Problem Domain

simple_websocket addresses the need for real-time, bidirectional communication in Eiffel applications. WebSocket enables persistent connections for live updates, chat, notifications, and streaming data.

## Scope Boundaries

### In Scope
- WebSocket frame encoding/decoding (RFC 6455 Section 5)
- Frame types: text, binary, close, ping, pong, continuation
- Frame masking (client-to-server)
- Extended payload lengths (7-bit, 16-bit, 64-bit)
- WebSocket handshake (client and server)
- Sec-WebSocket-Key/Accept validation
- Message fragmentation
- Close codes per RFC 6455 Section 7.4.1

### Out of Scope
- WebSocket extensions (e.g., compression)
- Subprotocol negotiation (basic support only)
- Full WebSocket client/server (transport layer)
- TLS/WSS handling (handled by transport)
- Reconnection logic

## Target Users

- Eiffel developers implementing WebSocket clients
- Eiffel developers building WebSocket servers
- Real-time application developers

## Success Criteria

1. Correctly encode/decode all frame types
2. Proper handshake key computation
3. Handle message fragmentation
4. Validate per RFC 6455
