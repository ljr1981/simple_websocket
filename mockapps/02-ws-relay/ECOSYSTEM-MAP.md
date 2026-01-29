# WS-RELAY - Ecosystem Integration

## simple_* Dependencies

### Required Libraries

| Library | Purpose | Integration Point |
|---------|---------|-------------------|
| simple_websocket | Core WebSocket protocol | WS_FRAME, WS_HANDSHAKE, WS_FRAME_PARSER |
| simple_http | HTTP upgrade handling | Initial handshake HTTP parsing |
| simple_json | Message transformation, config | JSON path operations, config parsing |
| simple_yaml | Configuration files | YAML config loading |
| simple_logger | Traffic and event logging | Structured logging |
| simple_config | Configuration management | Hot reload, validation |

### Optional Libraries

| Library | Purpose | When Needed |
|---------|---------|-------------|
| simple_regex | Pattern matching | Filter rules with regex |
| simple_template | Message transformation | Complex transformations |
| simple_jwt | Token validation | Enterprise auth proxy |
| simple_datetime | Timestamp handling | Log timestamps |
| simple_csv | Metrics export | CSV metric dumps |

## Integration Patterns

### simple_websocket Integration

**Purpose:** WebSocket protocol handling for both server and client sides

**Server-side (accepting clients):**
```eiffel
class RELAY_SERVER

feature -- Accept

    handle_client_upgrade (a_socket: STREAM_SOCKET)
            -- Process WebSocket upgrade from client.
        local
            handshake: WS_HANDSHAKE
            request: STRING
        do
            request := read_http_request (a_socket)
            create handshake.make

            if handshake.parse_client_request (request) then
                -- Valid upgrade request
                a_socket.put_string (handshake.create_server_response)
                create_session (a_socket, handshake.requested_path)
            else
                -- Reject with HTTP 400
                send_http_error (a_socket, 400, handshake.last_error)
            end
        end

end
```

**Client-side (connecting to upstream):**
```eiffel
class RELAY_SESSION

feature -- Upstream Connection

    connect_upstream (a_url: STRING)
            -- Establish connection to upstream.
        local
            handshake: WS_HANDSHAKE
        do
            create handshake.make
            upstream_socket.put_string (
                handshake.create_client_request (upstream_host, upstream_path)
            )

            if handshake.validate_server_response (read_response (upstream_socket)) then
                upstream_connected := True
            else
                last_error := "Upstream handshake failed: " + handshake.last_error
            end
        end

end
```

**Data flow:** Client socket -> WS_HANDSHAKE (server) -> RELAY_SESSION -> WS_HANDSHAKE (client) -> Upstream socket

### simple_json Integration

**Purpose:** JSON message transformation and configuration

**Message transformation:**
```eiffel
class RELAY_TRANSFORM

feature -- JSON Transformation

    transform_json (a_frame: WS_FRAME; a_rule: TRANSFORM_RULE): WS_FRAME
            -- Apply JSON transformation to frame.
        require
            text_frame: a_frame.is_text
        local
            json: SIMPLE_JSON_OBJECT
            payload: STRING
        do
            payload := a_frame.text_payload
            json := json_parser.parse_object (payload)

            inspect a_rule.action
            when Action_insert then
                json.put_at_path (a_rule.path, a_rule.value)

            when Action_remove then
                json.remove_at_path (a_rule.path)

            when Action_mask then
                if attached json.string_at_path (a_rule.path) as val then
                    json.put_string_at_path (a_rule.path, mask_string (val))
                end

            when Action_rename then
                json.rename_key (a_rule.from_path, a_rule.to_path)
            end

            create Result.make_text (json.to_string, a_frame.is_fin)
        end

end
```

**Configuration loading:**
```eiffel
class RELAY_CONFIG

feature -- Loading

    load_yaml (a_path: STRING)
            -- Load configuration from YAML file.
        local
            yaml: SIMPLE_YAML
            json: SIMPLE_JSON_OBJECT
        do
            create yaml.make
            json := yaml.parse_file_as_json (a_path)

            -- Parse server section
            if attached json.object_at ("server") as server then
                bind_address := server.string_at ("bind")
                max_connections := server.integer_at ("max_connections")
            end

            -- Parse upstreams
            if attached json.object_at ("upstreams") as upstreams then
                across upstreams.keys as k loop
                    parse_upstream (k.item, upstreams.object_at (k.item))
                end
            end

            -- Parse routing rules
            if attached json.array_at ("routing") as routes then
                across routes as r loop
                    parse_route (r.item.as_object)
                end
            end
        end

end
```

**Data flow:** YAML file -> SIMPLE_YAML -> JSON object -> RELAY_CONFIG

### simple_http Integration

**Purpose:** HTTP parsing for WebSocket upgrade handling

**Usage:**
```eiffel
class RELAY_SERVER

feature {NONE} -- HTTP Handling

    parse_upgrade_request (a_data: STRING): detachable HTTP_REQUEST
            -- Parse HTTP upgrade request.
        local
            http: SIMPLE_HTTP_PARSER
        do
            create http.make
            if http.parse_request (a_data) then
                Result := http.last_request
                -- Verify it's an upgrade request
                if not Result.header_value ("Upgrade").same_string_general ("websocket") then
                    Result := Void
                end
            end
        end

    build_error_response (a_code: INTEGER; a_message: STRING): STRING
            -- Build HTTP error response.
        local
            http: SIMPLE_HTTP_RESPONSE
        do
            create http.make (a_code)
            http.set_body (a_message)
            Result := http.to_string
        end

end
```

**Data flow:** Raw HTTP bytes -> SIMPLE_HTTP_PARSER -> HTTP_REQUEST -> WS_HANDSHAKE

### simple_logger Integration

**Purpose:** Structured traffic and event logging

**Usage:**
```eiffel
class RELAY_LOGGER

feature -- Traffic Logging

    log_message (a_session: RELAY_SESSION; a_frame: WS_FRAME; a_direction: INTEGER)
            -- Log relayed message.
        local
            entry: SIMPLE_JSON_OBJECT
        do
            create entry.make
            entry.put_string ("timestamp", current_iso_timestamp)
            entry.put_string ("session_id", a_session.id)
            entry.put_string ("direction", direction_name (a_direction))
            entry.put_string ("type", frame_type_name (a_frame))
            entry.put_integer ("size", a_frame.payload_length)

            if include_payload and a_frame.is_text then
                entry.put_string ("payload", truncate (a_frame.text_payload, max_payload_log))
            end

            logger.info (entry.to_string)
        end

    log_connection (a_session: RELAY_SESSION; a_event: STRING)
            -- Log connection event.
        do
            logger.info ("session=" + a_session.id + " event=" + a_event +
                        " client=" + a_session.client_address +
                        " upstream=" + a_session.upstream_url)
        end

end
```

**Data flow:** Events -> RELAY_LOGGER -> SIMPLE_LOGGER -> File/Console

### simple_config Integration

**Purpose:** Hot-reload configuration management

**Usage:**
```eiffel
class RELAY_CONFIG

inherit
    SIMPLE_CONFIG_RELOADABLE
        redefine
            on_reload
        end

feature -- Hot Reload

    on_reload
            -- Handle configuration reload.
        do
            -- Validate new config before applying
            if validate_config then
                apply_routing_rules
                apply_filter_rules
                apply_transform_rules
                logger.info ("Configuration reloaded successfully")
            else
                logger.error ("Configuration reload failed: " + validation_error)
            end
        end

feature {NONE} -- Validation

    validate_config: BOOLEAN
            -- Validate loaded configuration.
        do
            Result := True

            -- Check required fields
            if bind_address.is_empty then
                validation_error := "bind address required"
                Result := False
            end

            -- Validate upstreams
            across upstreams as u loop
                if not is_valid_websocket_url (u.item.url) then
                    validation_error := "Invalid upstream URL: " + u.item.url
                    Result := False
                end
            end

            -- Validate routing rules reference valid upstreams
            across routes as r loop
                if not upstreams.has (r.item.upstream_name) then
                    validation_error := "Route references unknown upstream: " + r.item.upstream_name
                    Result := False
                end
            end
        end

end
```

**Data flow:** File change -> SIMPLE_CONFIG -> on_reload -> Apply or reject

## Dependency Graph

```
ws-relay
    |
    +-- simple_websocket (required)
    |       +-- simple_base64
    |       +-- simple_hash
    |
    +-- simple_http (required)
    |
    +-- simple_json (required)
    |
    +-- simple_yaml (required)
    |       +-- simple_json
    |
    +-- simple_logger (required)
    |
    +-- simple_config (required)
    |       +-- simple_json
    |
    +-- simple_regex (optional)
    |
    +-- simple_template (optional)
    |
    +-- simple_jwt (optional, enterprise)
    |
    +-- ISE base (required)
    +-- ISE net (required for sockets)
    +-- ISE thread (required for concurrent sessions)
```

## ECF Configuration

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<system xmlns="http://www.eiffel.com/developers/xml/configuration-1-23-0"
        name="ws_relay" uuid="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX">

    <description>WS-RELAY: WebSocket Message Relay</description>

    <target name="ws_relay">
        <root class="RELAY_CLI" feature="make"/>

        <option warning="warning" syntax="standard">
            <assertions precondition="true" postcondition="true" check="true"/>
        </option>

        <setting name="console_application" value="true"/>
        <setting name="concurrency" value="thread"/>

        <capability>
            <concurrency support="thread"/>
            <void_safety support="all"/>
        </capability>

        <!-- Application source -->
        <cluster name="src" location=".\src\" recursive="true"/>

        <!-- simple_* dependencies (required) -->
        <library name="simple_websocket" location="$SIMPLE_EIFFEL\simple_websocket\simple_websocket.ecf"/>
        <library name="simple_http" location="$SIMPLE_EIFFEL\simple_http\simple_http.ecf"/>
        <library name="simple_json" location="$SIMPLE_EIFFEL\simple_json\simple_json.ecf"/>
        <library name="simple_yaml" location="$SIMPLE_EIFFEL\simple_yaml\simple_yaml.ecf"/>
        <library name="simple_logger" location="$SIMPLE_EIFFEL\simple_logger\simple_logger.ecf"/>
        <library name="simple_config" location="$SIMPLE_EIFFEL\simple_config\simple_config.ecf"/>

        <!-- simple_* dependencies (optional) -->
        <library name="simple_regex" location="$SIMPLE_EIFFEL\simple_regex\simple_regex.ecf"/>
        <library name="simple_datetime" location="$SIMPLE_EIFFEL\simple_datetime\simple_datetime.ecf"/>

        <!-- ISE dependencies -->
        <library name="base" location="$ISE_LIBRARY\library\base\base.ecf"/>
        <library name="net" location="$ISE_LIBRARY\library\net\net.ecf"/>
        <library name="thread" location="$ISE_LIBRARY\library\thread\thread.ecf"/>
    </target>

    <target name="ws_relay_tests" extends="ws_relay">
        <root class="TEST_APP" feature="make"/>
        <library name="simple_testing" location="$SIMPLE_EIFFEL\simple_testing\simple_testing.ecf"/>
        <cluster name="tests" location=".\tests\" recursive="true"/>
    </target>

</system>
```

## Integration Notes

### Thread Safety

WS-RELAY uses threads for concurrent session handling:
- Main thread: Accept connections, manage lifecycle
- Session threads: Handle individual client-upstream pairs
- Config thread: Watch for config file changes

All shared state (metrics, connection pool) uses thread-safe access patterns from simple_* conventions.

### Protocol Bridge Pattern

WS-RELAY acts as a protocol bridge:
1. Accept client connection (server-side WebSocket)
2. Establish upstream connection (client-side WebSocket)
3. Relay frames between them with optional transformation

This requires correct implementation of both sides of the WebSocket protocol, which simple_websocket provides through WS_HANDSHAKE (both create_client_request and parse_client_request paths).
