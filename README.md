# PropertyScoop MCP Server

A remote [Model Context Protocol (MCP)](https://modelcontextprotocol.io) server that provides comprehensive US property intelligence data. Query any US address and get 16+ data points back instantly.

## Tools

### `get_property_info`

Returns comprehensive property data for a given US address:

- **Noise levels** — environmental noise scores
- **Wetlands** — wetland areas on or near the property
- **Terrain slope** — slope percentage of the lot
- **Solar exposure** — natural light and solar potential
- **Powerline proximity** — nearby power lines
- **Crime rates** — local crime statistics
- **Property facing direction** — which direction the front of the property faces
- **Zoning** — zoning code and land use regulations
- **Property details** — parcel and building information
- **Radon risk** — radon gas exposure levels
- **Natural hazards** — earthquake, flood, wildfire, tornado, and other FEMA risk data
- **Neighborhood demographics** — census and socioeconomic data
- **Falling tree risk** — risk of tree fall damage
- **RF/cell tower exposure** — nearby radio frequency sources
- **School info** — nearby schools and ratings
- **Environmental hazards** — toxic release sites, underground storage tanks, cleanup sites
- **Water quality** — local water system violations and contaminants
- **Privacy rating** — based on surrounding density and visibility factors

**Input:**

```json
{
  "address": "123 Main St, Seattle, WA 98101"
}
```

**Usage metadata** is appended to each response:

```json
{
  "_usage": {
    "get_property_info_calls_this_month": 3,
    "free_calls_remaining": 7,
    "free_tier_limit": 10,
    "note": "Free tier: 7 of 10 free call(s) remaining this month."
  }
}
```

### `get_facing`

Returns the cardinal/intercardinal direction a property faces (N, S, E, W, NE, NW, SE, SW).

**Input:**

```json
{
  "address": "123 Main St, Seattle, WA 98101"
}
```

**Response:**

```json
{
  "facing": "Northwest",
  "address": "123 Main St, Seattle, WA 98101"
}
```

## Setup

### 1. Get an API key

Sign up at [propertyscoop.us/APIAccess](https://www.propertyscoop.us/APIAccess) to get your API key. The first 10 calls each month are free.

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

| Tier | Calls/Month | Price |
|------|-------------|-------|
| Free | First 10 | $0.00 |
| Pay-as-you-go | 11+ | $0.37/call |

Volume discounts available for high-usage customers. Manage billing at [propertyscoop.us/APIAccess](https://www.propertyscoop.us/APIAccess).

## Rate Limits

| Limit | Value |
|-------|-------|
| Burst | 100 concurrent requests |
| Sustained rate | 50 requests/sec |
| Daily quota | 10,000 requests |

Exceeding these limits returns HTTP `429 Too Many Requests`. Limits are per API key. Contact us if you need higher limits.

## Error Handling

The server uses standard [JSON-RPC 2.0](https://www.jsonrpc.org/specification) error codes:

| Code | Meaning | Description |
|------|---------|-------------|
| -32600 | Invalid Request | Malformed JSON-RPC request |
| -32601 | Method Not Found | Unknown MCP method |
| -32602 | Invalid Params | Missing or invalid parameters (e.g., empty address, address exceeds 500 characters) |
| -32603 | Internal Error | Server-side failure |

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

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `API key validation failed` | Invalid or deactivated key | Verify your key at [propertyscoop.us/APIAccess](https://www.propertyscoop.us/APIAccess) |
| `429 Too Many Requests` | Rate limit exceeded | Reduce request frequency or contact us for higher limits |
| `Address is required` | Empty address parameter | Provide a complete US street address |
| No response / timeout | Network issue | Verify connectivity to `https://mcp.propertyscoop.us` |

## Links

- [Website](https://www.propertyscoop.us)
- [API Access & Keys](https://www.propertyscoop.us/APIAccess)
- [MCP Registry](https://registry.modelcontextprotocol.io/v0.1/servers?search=propertyscoop)

## License

Proprietary. See [propertyscoop.us](https://www.propertyscoop.us) for terms of service.
