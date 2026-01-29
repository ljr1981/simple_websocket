# WS-PROBE - Ecosystem Integration

## simple_* Dependencies

### Required Libraries

| Library | Purpose | Integration Point |
|---------|---------|-------------------|
| simple_websocket | Core WebSocket protocol | WS_FRAME, WS_HANDSHAKE, WS_FRAME_PARSER |
| simple_json | JSON parsing and output | Message inspection, structured output |
| simple_cli | Command-line parsing | Argument handling, help generation |
| simple_logger | Session and debug logging | Audit trail, troubleshooting |
| simple_config | Configuration management | Connection profiles, settings |

### Optional Libraries

| Library | Purpose | When Needed |
|---------|---------|-------------|
| simple_regex | Pattern matching | --expect PATTERN validation |
| simple_template | Message templating | Pro feature: message templates |
| simple_encryption | TLS handling | wss:// connections (if not EiffelNet) |
| simple_datetime | Timestamp formatting | Session recording, logs |

## Integration Patterns

### simple_websocket Integration

**Purpose:** Core WebSocket protocol handling

**Usage:**
```eiffel
class PROBE_SESSION

feature -- Connection

    connect (a_url: STRING)
            -- Establish WebSocket connection.
        local
            handshake: WS_HANDSHAKE
            request: STRING
        do
            create handshake.make
            request := handshake.create_client_request (host, path)
            -- Send via socket...

            if handshake.validate_server_response (response) then
                is_connected := True
            else
                last_error := handshake.last_error
            end
        ensure
            connected_or_error: is_connected or not last_error.is_empty
        end

feature -- Messaging

    send_text (a_message: STRING)
            -- Send text message.
        require
            connected: is_connected
        local
            frame: WS_FRAME
        do
            create frame.make_text (a_message, True)
            frame.set_mask (generate_mask_key)
            socket.put_bytes (frame.to_bytes)
        end

    receive: detachable WS_FRAME
            -- Receive next frame.
        require
            connected: is_connected
        do
            parser.add_bytes (socket.read_bytes)
            if parser.parse and parser.has_frame then
                Result := parser.last_frame
            end
        end

end
```

**Data flow:** URL -> PROBE_SESSION.connect -> WS_HANDSHAKE -> Socket -> WS_FRAME -> send/receive

### simple_json Integration

**Purpose:** JSON message parsing and structured output

**Usage:**
```eiffel
class PROBE_OUTPUT

feature -- JSON Output

    format_message_json (a_frame: WS_FRAME): STRING
            -- Format frame as JSON.
        local
            json: SIMPLE_JSON_OBJECT
        do
            create json.make
            json.put_string ("type", frame_type_name (a_frame))
            json.put_integer ("opcode", a_frame.opcode)
            json.put_boolean ("fin", a_frame.is_fin)
            json.put_integer ("length", a_frame.payload_length)

            if a_frame.is_text then
                json.put_string ("payload", a_frame.text_payload)
                -- Try to parse as JSON
                if is_valid_json (a_frame.text_payload) then
                    json.put_object ("payload_json", parse_json (a_frame.text_payload))
                end
            else
                json.put_string ("payload_base64", encode_base64 (a_frame.payload))
            end

            Result := json.to_string_pretty
        end

end
```

**Data flow:** WS_FRAME -> SIMPLE_JSON_OBJECT -> JSON string -> stdout

### simple_cli Integration

**Purpose:** Command-line argument parsing

**Usage:**
```eiffel
class PROBE_CLI

inherit
    SIMPLE_CLI_APPLICATION
        redefine
            application_name,
            configure_commands
        end

feature -- Configuration

    application_name: STRING = "ws-probe"

    configure_commands
            -- Define CLI commands.
        do
            add_command (create {PROBE_CONNECT_COMMAND}.make)
            add_command (create {PROBE_SEND_COMMAND}.make)
            add_command (create {PROBE_RECEIVE_COMMAND}.make)
            add_command (create {PROBE_INTERACTIVE_COMMAND}.make)
            add_command (create {PROBE_PROFILE_COMMAND}.make)

            add_global_option ("config", "Configuration file", True)
            add_global_option ("output", "Output format: text, json", True)
            add_global_option ("timeout", "Timeout in milliseconds", True)
            add_global_flag ("verbose", "Verbose output")
            add_global_flag ("quiet", "Suppress non-essential output")
        end

end
```

**Data flow:** argv -> SIMPLE_CLI -> Command object -> execute

### simple_logger Integration

**Purpose:** Debug logging and session recording

**Usage:**
```eiffel
class PROBE_SESSION

feature {NONE} -- Logging

    logger: SIMPLE_LOGGER

    log_frame_sent (a_frame: WS_FRAME)
            -- Log outgoing frame.
        do
            logger.debug ("SEND [" + frame_type_name (a_frame) + "] " +
                         a_frame.payload_length.out + " bytes")
            if verbose_mode and a_frame.is_text then
                logger.trace ("  Payload: " + a_frame.text_payload)
            end
        end

    log_frame_received (a_frame: WS_FRAME)
            -- Log incoming frame.
        do
            logger.debug ("RECV [" + frame_type_name (a_frame) + "] " +
                         a_frame.payload_length.out + " bytes")
            if verbose_mode and a_frame.is_text then
                logger.trace ("  Payload: " + a_frame.text_payload)
            end
        end

end
```

**Data flow:** Events -> SIMPLE_LOGGER -> Console/File

### simple_config Integration

**Purpose:** Connection profiles and settings

**Usage:**
```eiffel
class PROBE_CONFIG

feature -- Profile Management

    load_profile (a_name: STRING): detachable PROBE_PROFILE
            -- Load named connection profile.
        local
            config: SIMPLE_CONFIG
        do
            create config.make_from_file (config_file_path)
            if attached config.object_at ("profiles." + a_name) as profile_obj then
                create Result.make
                Result.set_url (profile_obj.string_at ("url"))
                Result.set_timeout (profile_obj.integer_at ("timeout"))
                if attached profile_obj.object_at ("headers") as headers then
                    across headers as h loop
                        Result.add_header (h.key, h.item.as_string)
                    end
                end
            end
        end

    save_profile (a_name: STRING; a_profile: PROBE_PROFILE)
            -- Save connection profile.
        require
            valid_name: not a_name.is_empty
        local
            config: SIMPLE_CONFIG
        do
            create config.make_from_file (config_file_path)
            config.put_string ("profiles." + a_name + ".url", a_profile.url)
            config.put_integer ("profiles." + a_name + ".timeout", a_profile.timeout)
            config.save
        end

end
```

**Data flow:** JSON config file -> SIMPLE_CONFIG -> PROBE_PROFILE -> PROBE_SESSION

## Dependency Graph

```
ws-probe
    |
    +-- simple_websocket (required)
    |       +-- simple_base64
    |       +-- simple_hash
    |
    +-- simple_json (required)
    |
    +-- simple_cli (required)
    |
    +-- simple_logger (required)
    |
    +-- simple_config (required)
    |       +-- simple_json
    |
    +-- simple_regex (optional)
    |
    +-- simple_datetime (optional)
    |
    +-- ISE base (required)
    +-- ISE net (required for sockets)
```

## ECF Configuration

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<system xmlns="http://www.eiffel.com/developers/xml/configuration-1-23-0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.eiffel.com/developers/xml/configuration-1-23-0
        http://www.eiffel.com/developers/xml/configuration-1-23-0.xsd"
        name="ws_probe" uuid="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX">

    <description>WS-PROBE: WebSocket Testing Probe</description>

    <target name="ws_probe">
        <root class="PROBE_CLI" feature="make"/>

        <option warning="warning" syntax="standard" manifest_array_type="mismatch_warning">
            <assertions precondition="true" postcondition="true" check="true" invariant="true"/>
        </option>

        <setting name="console_application" value="true"/>
        <setting name="dead_code_removal" value="feature"/>

        <capability>
            <concurrency support="none"/>
            <void_safety support="all"/>
        </capability>

        <!-- Application source -->
        <cluster name="src" location=".\src\" recursive="true"/>

        <!-- simple_* dependencies -->
        <library name="simple_websocket" location="$SIMPLE_EIFFEL\simple_websocket\simple_websocket.ecf"/>
        <library name="simple_json" location="$SIMPLE_EIFFEL\simple_json\simple_json.ecf"/>
        <library name="simple_cli" location="$SIMPLE_EIFFEL\simple_cli\simple_cli.ecf"/>
        <library name="simple_logger" location="$SIMPLE_EIFFEL\simple_logger\simple_logger.ecf"/>
        <library name="simple_config" location="$SIMPLE_EIFFEL\simple_config\simple_config.ecf"/>

        <!-- Optional dependencies -->
        <library name="simple_regex" location="$SIMPLE_EIFFEL\simple_regex\simple_regex.ecf"/>
        <library name="simple_datetime" location="$SIMPLE_EIFFEL\simple_datetime\simple_datetime.ecf"/>

        <!-- ISE dependencies (only when no simple_* alternative) -->
        <library name="base" location="$ISE_LIBRARY\library\base\base.ecf"/>
        <library name="net" location="$ISE_LIBRARY\library\net\net.ecf"/>
    </target>

    <target name="ws_probe_tests" extends="ws_probe">
        <root class="TEST_APP" feature="make"/>
        <library name="simple_testing" location="$SIMPLE_EIFFEL\simple_testing\simple_testing.ecf"/>
        <cluster name="tests" location=".\tests\" recursive="true"/>
    </target>

</system>
```

## Integration Notes

### Connection Lifecycle

1. **PROBE_CLI** parses arguments, loads config
2. **PROBE_CONFIG** resolves profile if specified
3. **PROBE_SESSION** manages socket + handshake via simple_websocket
4. **PROBE_MESSENGER** wraps high-level send/receive
5. **PROBE_OUTPUT** formats results via simple_json
6. **PROBE_CLI** returns exit code

### Error Propagation

All errors flow through the standard simple_* pattern:
- `has_error`: BOOLEAN query
- `last_error`: STRING with description
- Preconditions prevent invalid operations
- Postconditions guarantee state consistency
