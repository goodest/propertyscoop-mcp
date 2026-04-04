# PropertyScoop MCP Server

A remote [Model Context Protocol (MCP)](https://modelcontextprotocol.io) server that provides comprehensive US property intelligence data. Query any US address and get 16+ data points back instantly.

## Tools

### `get_property_info`

Returns comprehensive property data for a given US address:

- **Noise levels** - environmental noise scores
- **Wetlands** - wetland areas on or near the property
- **Terrain slope** - slope percentage of the lot
- **Solar exposure** - natural light and solar potential
- **Powerline proximity** - nearby power lines
- **Crime rates** - local crime statistics
- **Property facing direction** - which direction the front of the property faces
- **Zoning** - zoning code and land use regulations
- **Property details** - parcel and building information
- **Radon risk** - radon gas exposure levels
- **Natural hazards** - earthquake, flood, wildfire, tornado, and other FEMA risk data
- **Neighborhood demographics** - census and socioeconomic data
- **Falling tree risk** - risk of tree fall damage
- **RF/cell tower exposure** - nearby radio frequency sources
- **School info** - nearby schools and ratings
- **Environmental hazards** - toxic release sites, underground storage tanks

**Input:**

```json
{
  "address": "123 Main St, Seattle, WA 98101"
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

| Tier | Price |
|------|-------|
| First 10 calls/month | Free |
| Additional calls | $0.37/call (volume discounts available) |

Manage billing at [propertyscoop.us/APIAccess](https://www.propertyscoop.us/APIAccess).

## Links

- [Website](https://www.propertyscoop.us)
- [API Access & Keys](https://www.propertyscoop.us/APIAccess)
- [MCP Registry](https://registry.modelcontextprotocol.io/v0.1/servers?search=propertyscoop)

## License

Proprietary. See [propertyscoop.us](https://www.propertyscoop.us) for terms of service.
