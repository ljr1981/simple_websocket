# S03 CONTRACTS - simple_websocket

**BACKWASH** - Generated: 2026-01-23
**Library**: simple_websocket

## WS_FRAME Contracts

### make
```eiffel
make (a_opcode: INTEGER; a_payload: ARRAY [NATURAL_8]; a_fin: BOOLEAN)
    require
        valid_opcode: is_valid_opcode (a_opcode)
        payload_not_void: a_payload /= Void
    ensure
        opcode_set: opcode = a_opcode
        payload_set: payload = a_payload
        fin_set: is_fin = a_fin
```

### make_close
```eiffel
make_close (a_code: INTEGER; a_reason: STRING)
    require
        valid_code: a_code >= 1000 and a_code <= 4999
```

### text_payload
```eiffel
text_payload: STRING
    require
        is_text: opcode = Opcode_text or opcode = Opcode_continuation
```

### close_code
```eiffel
close_code: INTEGER
    require
        is_close: opcode = Opcode_close
        has_code: payload.count >= 2
```

### set_mask
```eiffel
set_mask (a_key: ARRAY [NATURAL_8])
    require
        key_not_void: a_key /= Void
        key_length: a_key.count = 4
    ensure
        masked: is_masked
        key_set: mask_key = a_key
```

## WS_FRAME_PARSER Contracts

### add_bytes
```eiffel
add_bytes (a_bytes: ARRAY [NATURAL_8])
    require
        bytes_not_void: a_bytes /= Void
    ensure
        bytes_added: buffer.count >= old buffer.count
```

### reset
```eiffel
reset
    ensure
        empty_buffer: buffer.is_empty
        no_frame: not has_frame
        no_error: not has_error
```

## WS_MESSAGE Contracts

### make_text
```eiffel
make_text (a_text: STRING)
    require
        text_not_void: a_text /= Void
    ensure
        is_text: is_text
        not_binary: not is_binary
        text_set: text.same_string (a_text)
```

### to_frames
```eiffel
to_frames (a_max_size: INTEGER): ARRAYED_LIST [WS_FRAME]
    require
        positive_size: a_max_size > 0
    ensure
        has_frames: Result.count > 0
        last_is_fin: Result.last.is_fin
```

## WS_HANDSHAKE Contracts

### create_client_request
```eiffel
create_client_request (a_host: STRING; a_path: STRING): STRING
    require
        host_not_void: a_host /= Void
        host_not_empty: not a_host.is_empty
        path_not_void: a_path /= Void
    ensure
        key_set: sec_websocket_key /= Void
```

### validate_server_response
```eiffel
validate_server_response (a_response: STRING): BOOLEAN
    require
        response_not_void: a_response /= Void
        key_set: sec_websocket_key /= Void
```

### create_server_response
```eiffel
create_server_response: STRING
    require
        valid_request: is_valid
        key_set: sec_websocket_key /= Void
    ensure
        accept_set: sec_websocket_accept /= Void
```

## Class Invariants

### WS_FRAME
```eiffel
invariant
    payload_not_void: payload /= Void
    mask_key_not_void: mask_key /= Void
    mask_key_length: mask_key.count = 4
```

### WS_FRAME_PARSER
```eiffel
invariant
    buffer_not_void: buffer /= Void
    last_error_not_void: last_error /= Void
```

### WS_MESSAGE
```eiffel
invariant
    exclusive_type: is_text xor is_binary
    text_not_void: text /= Void
    data_not_void: data /= Void
```

### WS_HANDSHAKE
```eiffel
invariant
    last_error_not_void: last_error /= Void
```
