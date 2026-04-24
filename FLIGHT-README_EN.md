# RollingGo Flight MCP Server

`RollingGo-Flight-MCP` is a flight service based on MCP. It provides airport lookup and flight search capabilities for AI agents and MCP clients. (updated on 2026-04-24).

## Online Endpoint

- MCP URL: `https://mcp.rollinggo.cn/mcp/flight`
- Transport: `streamable_http`
- Current online configuration: no client-side Bearer Token is required. For private deployments or business channels that require API keys, follow the actual environment configuration.

## Tool List

The flight MCP documentation currently keeps 2 available tools:

1. `searchAirports`: search airport/city information by city name, airport name, airport code, or similar keywords
2. `searchFlights`: search flight options and prices by date, route, passenger count, and cabin class

## Recommended Flow

1. If the user provides only natural-language city names, call `searchAirports` first to get a usable `cityCode` or `airportCode`.
2. Call `searchFlights` to get flight options, segment details, and prices.
3. Prices and inventory may change with supplier availability; refresh before showing final results or checkout.

## 1) searchAirports

Searches airport, city, or related transport-node information by keyword.

### Input Parameters

- `keyword` (string, required): Search keyword, such as city name, airport name, or airport code. Examples: `杭州`, `Hangzhou`, `HGH`, `成都`, `CTU`.

### Output Structure

```json
{
  "message": "机场搜索成功",
  "airPortInformationList": [
    {
      "airportCode": "TFU",
      "airportName": "Tianfu Airport",
      "cityCode": "CTU",
      "cityName": "Chengdu",
      "countryCode": "CN",
      "countryName": "China",
      "timeZone": "+08:00"
    }
  ]
}
```

Notes:

- `airportCode` can be used as `searchFlights.fromAirport` / `searchFlights.toAirport`.
- `cityCode` can be used as `searchFlights.fromCity` / `searchFlights.toCity`.
- Results are supplier-provided and may include city-related transport nodes. Clients should filter by `cityCode`, `airportCode`, and `countryCode` when needed.
- If no result is matched, `airPortInformationList` may be an empty array.

## 2) searchFlights

Searches flight options by date, route, passenger count, and cabin class.

### Input Parameters

- `adultNumber` (integer, required): Number of adults.
- `childNumber` (integer, required): Number of children. Use `0` if there are no children.
- `cabinGrade` (string, required): Cabin preference. Supported values:
  - `ECONOMY`: Economy
  - `PREMIUM_ECONOMY`: Premium Economy
  - `BUSINESS`: Business
  - `FIRST`: First
- `tripType` (string, required): Trip type. Supported values:
  - `ONE_WAY`: One-way
  - `ROUND_TRIP`: Round-trip
- `fromDate` (string, required, format `YYYY-MM-DD`): Departure date.
- `retDate` (string, required for round-trip, format `YYYY-MM-DD`): Return date. Used when `tripType=ROUND_TRIP`.
- `fromCity` (string, optional): Departure city code. Use either `fromCity` or `fromAirport`. Example: `HGH`.
- `fromAirport` (string, optional): Departure airport code. Use either `fromAirport` or `fromCity`. Example: `HGH`.
- `toCity` (string, optional): Arrival city code. Use either `toCity` or `toAirport`. Example: `CTU`.
- `toAirport` (string, optional): Arrival airport code. Use either `toAirport` or `toCity`. Example: `TFU`.

### Output Structure

```json
{
  "message": "航班搜索成功",
  "flightInformationList": [
    {
      "routingId": "...",
      "totalAdultPrice": 1813.0,
      "totalChildPrice": 0.0,
      "currency": "CNY",
      "fromSmartValueScore": 95.0,
      "validatingCarrier": "3U",
      "fromSegments": [
        {
          "flightNumber": "3U8916",
          "depTime": "2026-05-01T17:50:00",
          "arrTime": "2026-05-01T21:00:00",
          "depAirport": "HGH",
          "arrAirport": "CTU",
          "duration": "190",
          "stopCities": ""
        }
      ],
      "retSegments": []
    }
  ]
}
```

Field notes:

- `routingId`: Route identifier for the flight option returned by the current search.
- `totalAdultPrice`: Total adult price in `currency`.
- `totalChildPrice`: Total child price in `currency`.
- `fromSmartValueScore`: Supplier-provided recommendation score. Higher scores usually indicate a better combined value across price, timing, and directness.
- `validatingCarrier`: Validating or carrier airline code.
- `fromSegments`: Outbound segment list. Direct flights usually have 1 segment; connecting flights have multiple segments.
- `retSegments`: Return segment list. It is usually an empty array for one-way trips.
- `duration`: Segment flight duration in minutes. The current response returns it as a string.
- `stopCities`: Stopover cities. It is usually an empty string when there is no stopover.

Notes:

- Prices and inventory are real-time data. Mark displayed prices as subject to the final checkout result.
- A city may have multiple airports. Use `fromAirport` / `toAirport` to restrict the search to specific airports.
- If the user only says a city name, using `fromCity` / `toCity` usually gives better city-level coverage.

## Usage Examples

### Example 1: Search airport/city code

```json
{
  "keyword": "CTU"
}
```

### Example 2: Hangzhou to Chengdu, May 1, one-way, 1 adult, Economy

```json
{
  "adultNumber": 1,
  "childNumber": 0,
  "cabinGrade": "ECONOMY",
  "fromCity": "HGH",
  "toCity": "CTU",
  "fromDate": "2026-05-01",
  "tripType": "ONE_WAY"
}
```

### Example 3: Hangzhou to Chengdu round-trip, 1 adult, Economy

```json
{
  "adultNumber": 1,
  "childNumber": 0,
  "cabinGrade": "ECONOMY",
  "fromCity": "HGH",
  "toCity": "CTU",
  "fromDate": "2026-05-01",
  "retDate": "2026-05-05",
  "tripType": "ROUND_TRIP"
}
```

## MCP Client Config Example

Different MCP clients may use slightly different field names. The key configuration is the `streamable_http` transport and the online MCP URL.

```json
{
  "mcpServers": {
    "RollingGo-Flight-MCP": {
      "url": "https://mcp.rollinggo.cn/mcp/flight",
      "type": "streamable_http"
    }
  }
}
```

If your MCP client requires explicit headers, add them according to your business environment:

```json
{
  "mcpServers": {
    "RollingGo-Flight-MCP": {
      "url": "https://mcp.rollinggo.cn/mcp/flight",
      "type": "streamable_http",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

## Integration Notes

- When extracting parameters from user intent, identify whether the trip is one-way or round-trip first. Round-trip requests require `retDate`.
- When the user provides Chinese city names only, `searchAirports` can help resolve codes. If IATA city codes are already known, `searchFlights` can be called directly.
- For multi-airport cities, city codes are better for broad search, while airport codes are better for precise filtering.
- When displaying results, show at least flight number, departure/arrival time, departure/arrival airport, total price, currency, and whether the route has a connection.

## Links

- GitHub repository: https://github.com/RollingGo-AI/rollinggo-readme
- API key application: https://rollinggo.store/apply
- MCP standard: https://modelcontextprotocol.io/
