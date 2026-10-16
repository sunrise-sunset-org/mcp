<p align="center"><img src="assets/logo.png" width="96" height="96" alt="Sunrise-Sunset.org"></p>

# Sunrise & Sunset Times MCP server

A free, remote [MCP](https://modelcontextprotocol.io) server that gives AI
assistants and agents accurate sun and moon times for any place on Earth and
any date: sunrise, sunset, twilight, golden hour, blue hour, solar position,
moonrise, moonset and moon phase. By [Sunrise-Sunset.org](https://sunrise-sunset.org).

```
https://api.sunrise-sunset.org/mcp
```

- **No API key, no sign up.** Add the URL to your client and ask.
- **Remote.** Nothing to install or run: Streamable HTTP, no authentication.
- **Local time.** Times come back in ISO 8601 in the time zone of the place.
- **Polar day and night handled.** Each day says whether it is normal, midnight
  sun or polar night, and events that do not happen are `null`, never a made-up time.

## Contents

- [Connect it](#connect-it)
- [Tools](#tools)
- [Things to ask](#things-to-ask)
- [Example result](#example-result)
- [Try it with curl](#try-it-with-curl)
- [Limits and attribution](#limits-and-attribution)
- [Questions and problems](#questions-and-problems)
- [License](#license)

## Connect it

**Claude** (claude.ai, desktop and mobile apps): Settings → Connectors → Add
custom connector, and paste the URL above.

**Claude Code:**

```sh
claude mcp add --transport http sunrise-sunset https://api.sunrise-sunset.org/mcp
```

**ChatGPT:** Settings → Apps & Connectors → Advanced settings → Developer mode,
then add the URL as a custom connector.

**Codex:**

```sh
codex mcp add sunrise-sunset --url https://api.sunrise-sunset.org/mcp
```

**Cursor:** add this to `~/.cursor/mcp.json`, or to `.cursor/mcp.json` inside a
project (same as [mcp.json](mcp.json)):

```json
{ "mcpServers": { "sunrise-sunset": { "url": "https://api.sunrise-sunset.org/mcp" } } }
```

**VS Code** (GitHub Copilot agent mode): add this to `.vscode/mcp.json` (same as
[configs/vscode-mcp.json](configs/vscode-mcp.json)):

```json
{ "servers": { "sunrise-sunset": { "type": "http", "url": "https://api.sunrise-sunset.org/mcp" } } }
```

**Any other MCP client** works too: it is a standard remote server over
Streamable HTTP with no authentication. More details in the
[setup guide](https://sunrise-sunset.org/mcp).

## Tools

Both tools are read-only and return the same per-day object.

### `get_sun_times`

One place, one day.

| Parameter | Required | Description |
|---|---|---|
| `lat` | yes | Latitude in decimal degrees, -90 to 90. |
| `lng` | yes | Longitude in decimal degrees, -180 to 180. |
| `date` | no | `today` (default), `tomorrow` or `YYYY-MM-DD`. Relative dates use the local date of the place. |
| `tz` | no | IANA time zone such as `Europe/Madrid`. Leave it out and the time zone of the coordinates is used (recommended). |

### `get_sun_times_range`

One place, up to 366 consecutive days in a single call: calendars, trips,
or comparing seasons without one call per day.

| Parameter | Required | Description |
|---|---|---|
| `lat`, `lng`, `tz` | | As above. |
| `date_start` | yes | First day, `YYYY-MM-DD`. |
| `date_end` | yes | Last day, inclusive, `YYYY-MM-DD`. |

A range can return fewer days than requested: calendar days that never existed
in that time zone are left out (Samoa skipped 30 December 2011 when it moved
across the date line). Match each entry by its `date` field.

### What each day contains

`sunrise`, `sunset`, `solar_noon`, `day_length` (seconds), `sun_status`
(`normal`, `midnight_sun` or `polar_night`), civil, nautical and astronomical
twilight, `dawn`, `dusk`, `first_light`, `last_light`, `golden_hour` and
`blue_hour` (morning and evening), the sun's azimuth at sunrise and sunset and
its altitude at noon, `moonrise`, `moonset`, `moon_phase` and
`moon_illumination`. Both tools declare an `outputSchema`, so clients and
agents know every field and type in advance.

## Things to ask

- "When is golden hour in San Francisco tomorrow?"
- "How much daylight does Seattle lose during November?"
- "Give me sunrise and sunset in Tromsø for every day of December 2026."
- "Which nights this month will be darkest for stargazing in Moab, Utah?"

## Example result

`get_sun_times` for Lisbon on 24 December 2026 (shortened):

```json
{
  "date": "2026-12-24",
  "tzid": "Europe/Lisbon",
  "utc_offset": "+00:00",
  "lat": 38.72,
  "lng": -9.14,
  "sunrise": "2026-12-24T07:52:21+00:00",
  "sunset": "2026-12-24T17:19:57+00:00",
  "solar_noon": "2026-12-24T12:36:08+00:00",
  "day_length": 34056,
  "sun_status": "normal",
  "dawn": "2026-12-24T07:22:27+00:00",
  "dusk": "2026-12-24T17:49:51+00:00",
  "golden_hour": {
    "morning": { "begin": "2026-12-24T07:33:53+00:00", "end": "2026-12-24T08:34:15+00:00" },
    "evening": { "begin": "2026-12-24T16:38:04+00:00", "end": "2026-12-24T17:38:25+00:00" }
  },
  "moonrise": "2026-12-24T17:48:33+00:00",
  "moonset": "2026-12-24T08:29:49+00:00",
  "moon_phase": "Full Moon",
  "moon_illumination": 99.61
}
```

## Try it with curl

The server speaks JSON-RPC 2.0 over POST:

```sh
curl -s https://api.sunrise-sunset.org/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_sun_times","arguments":{"lat":38.72,"lng":-9.14,"date":"2026-12-24"}}}'
```

Prefer plain HTTP? The same data is available as a REST API: see
[sunrise-sunset-org/api](https://github.com/sunrise-sunset-org/api).

## Limits and attribution

- Free for reasonable use. For many days at one place, use
  `get_sun_times_range` instead of one call per day. A client that calls the
  server in a tight loop may get HTTP 429 for a short while.
- **Attribution is required** wherever people see the data: a link to
  [sunrise-sunset.org](https://sunrise-sunset.org), or a mention of
  Sunrise-Sunset.org when there is no screen. Details in the
  [API repository](https://github.com/sunrise-sunset-org/api#usage-limits-and-attribution).
- Is the server down? Check the [status page](https://apistatus.sunrise-sunset.org/).

## Questions and problems

Open an [issue](https://github.com/sunrise-sunset-org/mcp/issues). Include the
client you use, what you asked and what came back.

Changes to the server are announced as
[releases of the API repository](https://github.com/sunrise-sunset-org/api/releases):
click *Watch* → *Custom* → *Releases* there to get them.

## License

The files in this repository (configuration and examples) are under the
[MIT License](LICENSE). The **data** the server returns has its own terms: free
to use with the attribution described above. See the
[API documentation](https://sunrise-sunset.org/api#rate-limits).
