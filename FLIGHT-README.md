# RollingGo Flight MCP Server

`RollingGo-Flight-MCP` 是一个基于 MCP的机票服务，面向 AI Agent 和 MCP 客户端提供机场检索和航班查询能力。（更新日期：2026-04-24）。

## 在线端点

- MCP URL: `https://mcp.rollinggo.cn/mcp/flight`
- Transport: `streamable_http`
- 当前在线配置：无需在客户端侧配置 Bearer Token。如私有部署或业务渠道需要 API Key，请以实际环境配置为准。

## 工具列表

当前机票 MCP 对外文档保留 2 个可用工具：

1. `searchAirports`：根据城市名、机场名、机场代码等关键字搜索机场/城市信息
2. `searchFlights`：查询指定日期、航线、人数、舱等的航班列表和价格

## 推荐调用流程

1. 如果用户只提供自然语言城市名，先调用 `searchAirports` 获取可用的 `cityCode` 或 `airportCode`。
2. 调用 `searchFlights` 查询航班列表、航段信息和价格。
3. 价格和库存都可能随供应商实时变化；展示给用户前或下单前建议重新查询。

## 1) searchAirports

根据关键字搜索机场、城市或相关交通节点信息。

### 输入参数

- `keyword` (string，必填)：搜索关键字，例如城市名、机场名、机场代码。示例：`杭州`、`Hangzhou`、`HGH`、`成都`、`CTU`。

### 输出结构

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

说明：

- `airportCode` 可作为 `searchFlights.fromAirport` / `searchFlights.toAirport` 使用。
- `cityCode` 可作为 `searchFlights.fromCity` / `searchFlights.toCity` 使用。
- 搜索结果由供应商返回，可能包含城市相关交通节点，客户端应结合 `cityCode`、`airportCode`、`countryCode` 做二次筛选。
- 当没有匹配结果时，`airPortInformationList` 可能为空数组。

## 2) searchFlights

查询指定日期、航线、乘客人数和舱等的航班方案。

### 输入参数

- `adultNumber` (integer，必填)：成人数量。
- `childNumber` (integer，必填)：儿童数量，没有儿童时传 `0`。
- `cabinGrade` (string，必填)：舱等偏好，支持：
  - `ECONOMY`：经济舱
  - `PREMIUM_ECONOMY`：超级经济舱
  - `BUSINESS`：商务舱
  - `FIRST`：头等舱
- `tripType` (string，必填)：行程类型，支持：
  - `ONE_WAY`：单程
  - `ROUND_TRIP`：往返
- `fromDate` (string，必填，格式：`YYYY-MM-DD`)：出发日期。
- `retDate` (string，往返必填，格式：`YYYY-MM-DD`)：返程日期。`tripType=ROUND_TRIP` 时使用。
- `fromCity` (string，选填)：出发城市代码，与 `fromAirport` 二选一。示例：`HGH`。
- `fromAirport` (string，选填)：出发机场代码，与 `fromCity` 二选一。示例：`HGH`。
- `toCity` (string，选填)：到达城市代码，与 `toAirport` 二选一。示例：`CTU`。
- `toAirport` (string，选填)：到达机场代码，与 `toCity` 二选一。示例：`TFU`。

### 输出结构

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

字段说明：

- `routingId`：航线唯一 ID，标识本次搜索返回的航班方案。
- `totalAdultPrice`：成人总价，币种见 `currency`。
- `totalChildPrice`：儿童总价，币种见 `currency`。
- `fromSmartValueScore`：供应商返回的综合推荐分，分数越高通常代表价格、时间、直飞等综合表现越好。
- `validatingCarrier`：出票/承运相关航司代码。
- `fromSegments`：去程航段列表。直飞通常只有 1 段，中转会有多段。
- `retSegments`：返程航段列表。单程通常为空数组。
- `duration`：航段飞行时长，单位为分钟，当前返回为字符串。
- `stopCities`：经停城市；无经停时通常为空字符串。

说明：

- 价格和库存具有实时性，展示给用户前建议标注“以实际下单为准”。
- 同一城市可能包含多个机场，使用 `fromAirport` / `toAirport` 可以限制到具体机场。
- 如果用户只说“成都”“上海”等城市，建议优先使用 `fromCity` / `toCity` 让服务返回覆盖该城市多机场的结果。

## 使用示例

### 示例 1：搜索机场/城市代码

```json
{
  "keyword": "CTU"
}
```

### 示例 2：杭州到成都，五一单程，1 位成人，经济舱

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

### 示例 3：杭州到成都往返，1 位成人，经济舱

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

## MCP 客户端配置示例

不同 MCP 客户端字段名可能略有差异，核心是配置 `streamable_http` 类型和线上 MCP URL。

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

如果你的 MCP 客户端要求显式 headers，可按业务环境补充：

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

## 接入建议

- LLM 参数抽取时，先判断用户要“单程”还是“往返”；往返必须补充 `retDate`。
- 用户只提供中文城市名时，可先用 `searchAirports`，但如果已经知道 IATA 城市代码，可以直接调用 `searchFlights`。
- 国内城市多机场场景下，城市代码适合泛查，机场代码适合精确筛选。
- 展示航班时建议至少显示航班号、起降时间、出发/到达机场、总价、币种、是否中转。

## 相关链接

- GitHub 仓库：https://github.com/RollingGo-AI/rollinggo-readme
- API Key 申请：https://rollinggo.store/apply
- MCP 标准：https://modelcontextprotocol.io/
