# PropertyScoop MCP Server

A remote [Model Context Protocol (MCP)](https://modelcontextprotocol.io) server that provides comprehensive US property intelligence data. Query any US address and get targeted property data back instantly.

## Tools

### `get_facing`

Returns the cardinal or intercardinal direction a property's front faces, derived from satellite imagery and machine learning. Useful for evaluating sun exposure, solar panel potential, or prevailing wind exposure. South-facing properties receive more winter sunlight in the Northern Hemisphere.

**Input:**

```json
{ "address": "123 Main St, Seattle, WA 98101" }
```

**Response:**

```json
{ "facing": "Southwest" }
```

Possible values: `North`, `South`, `East`, `West`, `Northeast`, `Northwest`, `Southeast`, `Southwest`

---

### `get_power_substation_location`

Finds electrical power substations within a 1-mile radius of a property. Useful for assessing proximity to high-voltage infrastructure or potential EMF exposure. Returns `count: 0` and an empty array if none are found within 1 mile.

**Input:**

```json
{ "address": "123 Main St, Seattle, WA 98101" }
```

**Response:**

```json
{
  "powerSubstationLocationsNearby": [
    {
      "type": "Feature",
      "geometry": { "type": "Point", "coordinates": [-122.2328941, 47.6192593] },
      "properties": {
        "name": "",
        "operator": "Puget Sound Energy",
        "substation_type": "distribution",
        "voltage": "",
        "city": "",
        "state": "WA",
        "distance_meters": 959.95,
        "distance_miles": 0.6
      }
    },
    {
      "type": "Feature",
      "geometry": { "type": "Point", "coordinates": [-122.208704, 47.6135383] },
      "properties": {
        "name": "",
        "operator": "Puget Sound Energy",
        "substation_type": "distribution",
        "voltage": "",
        "city": "",
        "state": "WA",
        "distance_meters": 1288.59,
        "distance_miles": 0.8
      }
    }
  ],
  "count": 2
}
```

---

### `get_water_quality`

Retrieves drinking water quality and EPA compliance history for the public water system serving a given address. Data is sourced from the EPA Safe Drinking Water Information System (SDWIS). Properties on private wells will not have EPA water system records.

**Input:**

```json
{ "address": "123 Main St, Seattle, WA 98101" }
```

**Response fields** (all nested under `drinking_water`):

- `system_name` — name of the serving water utility
- `pwsid` — EPA public water system ID
- `population_served` — number of people on the system
- `grade` — overall compliance grade (A–F)
- `violations_3yr` / `health_violations_3yr` / `monitoring_violations_3yr` — violation counts over 3 years
- `violations[]` — detailed violation records (if any): contaminant, type, date, severity
- `source_water_type` — e.g. surface water, groundwater
- `primary_sources[]` — named water source facilities

---

### `get_rf_exposure`

Measures radiofrequency (RF) electromagnetic radiation exposure at a property from FCC-licensed transmitters within a 2-mile radius, including cell towers and broadcast antennas.

**Input:**

```json
{ "address": "123 Main St, Seattle, WA 98101" }
```

**Response:**

```json
{
  "rfExposureInMilliwatts": 0.42,
  "emittersNearby": 3,
  "searchDistanceInMiles": 2
}
```

- `rfExposureInMilliwatts` — aggregate RF power density in mW from all nearby transmitters
- `emittersNearby` — count of licensed RF transmitters within 2 miles

---

### `get_noise_score`

Calculates an environmental noise score using national transportation noise models. Noise levels are reported in decibels (dB) across road traffic, railways, and aviation. Values above 65 dB are considered loud by EPA standards.

**Input:**

```json
{ "address": "123 Main St, Seattle, WA 98101" }
```

**Response:**

```json
{
  "combinedNoise": 58.3,
  "roadNoise": 57.1,
  "railNoise": 42.0,
  "aviationNoise": 35.5,
  "mainSources": ["Road"]
}
```

---

### `get_falling_tree_risk`

Assesses the risk of falling trees damaging a property, combining nearby tree density, estimated tree heights, distance of trees to the structure, and historical storm data (wind speed and precipitation intensity).

**Input:**

```json
{ "address": "123 Main St, Seattle, WA 98101" }
```

**Response:**

```json
{
  "risk_level": "Medium",
  "property_risk_annual_low": 0.002995654057260544,
  "property_risk_annual_high": 0.014891755791630046,
  "max_wind_speed_kmh": 48.2,
  "max_precipitation_24h_mm": 29.2
}
```

- `risk_level` — `Low`, `Medium`, `High`, or `Very High`
- `property_risk_annual_low` — lower bound of annual probability a tree strikes the property
- `property_risk_annual_high` — upper bound of annual probability a tree strikes the property
- `max_wind_speed_kmh` — maximum recorded wind speed in km/h near the property
- `max_precipitation_24h_mm` — maximum recorded 24-hour precipitation in mm

---

## Setup

### 1. Get an API key

Sign up at [propertyscoop.us/APIAccess](https://www.propertyscoop.us/APIAccess) to get your API key. Each tool has a free tier of 10 calls per month.

### 2. Configure your MCP client

#### Claude Desktop / Claude Code

Add to your MCP configuration:

```json
{
  "mcpServers": {
    "propertyscoop": {
      "type": "streamable-http",
      "url": "https://mcp.propertyscoop.us",
      "headers": {
        "X-API-Key": "YOUR_API_KEY"
      }
    }
  }
}
```

#### Cursor / Other MCP clients

Use the remote server URL `https://mcp.propertyscoop.us` with your API key in the `X-API-Key` header.

## Pricing

Each tool can be individually enabled or disabled from the API Access dashboard.

| Tool                            | Free Tier      | Paid Price    |
|---------------------------------|----------------|---------------|
| `get_facing`                    | 1–2,000 calls  | $0.02/call    |
|                                 | 2,001–5,000    | $0.015/call   |
|                                 | 5,001+         | $0.01/call    |
| `get_power_substation_location` | First 10/month | $0.01/call    |
| `get_water_quality`             | First 10/month | $0.01/call    |
| `get_rf_exposure`               | First 10/month | $0.03/call    |
| `get_noise_score`               | First 10/month | $0.01/call    |
| `get_falling_tree_risk`         | First 10/month | $0.20/call    |

Only `tools/call` requests that successfully retrieve data are billed. Protocol messages (`tools/list`, `initialize`) are free.

## Rate Limits

| Limit          | Value                  |
|----------------|------------------------|
| Burst          | 100 concurrent requests |
| Sustained rate | 50 requests/sec        |
| Daily quota    | 10,000 requests        |

Exceeding these limits returns HTTP `429 Too Many Requests`. Limits are per API key. Contact us if you need higher limits.

## Error Handling

The server uses standard [JSON-RPC 2.0](https://www.jsonrpc.org/specification) error codes:

| Code    | Meaning          | Description                                                              |
|---------|------------------|--------------------------------------------------------------------------|
| -32600  | Invalid Request  | Malformed JSON-RPC request                                               |
| -32601  | Method Not Found | Unknown MCP method                                                       |
| -32602  | Invalid Params   | Missing or invalid parameters (e.g., empty address, exceeds 500 chars)  |
| -32603  | Internal Error   | Server-side failure or tool not enabled for your key                    |

Example error response:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32602,
    "message": "Address is required"
  }
}
```

If a tool is not enabled for your API key, you will receive a `-32603` error directing you to the API Access dashboard to enable it.

## Troubleshooting

| Symptom                       | Cause                         | Fix                                                              |
|-------------------------------|-------------------------------|------------------------------------------------------------------|
| `API key validation failed`   | Invalid or deactivated key    | Verify your key at [propertyscoop.us/APIAccess](https://www.propertyscoop.us/APIAccess) |
| `Method 'X' is not enabled`   | Tool not enabled for this key | Enable it at [propertyscoop.us/APIAccess](https://www.propertyscoop.us/APIAccess) |
| `429 Too Many Requests`       | Rate limit exceeded           | Reduce request frequency or contact us for higher limits         |
| `Address is required`         | Empty address parameter       | Provide a complete US street address                             |
| No response / timeout         | Network issue                 | Verify connectivity to `https://mcp.propertyscoop.us`            |

## Links

- [Website](https://www.propertyscoop.us)
- [API Access & Keys](https://www.propertyscoop.us/APIAccess)
- [MCP Registry](https://registry.modelcontextprotocol.io/v0.1/servers?search=propertyscoop)

## License

Proprietary. See [propertyscoop.us](https://www.propertyscoop.us) for terms of service.
