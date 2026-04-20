# RollingGo MCP Server

`RollingGo-MCP` 是一个基于 MCP (Model Context Protocol) 的酒店搜索服务。
本文档已按在线 `RollingGo-MCP` 工具的最新实测结果更新（更新日期：2026-02-26）。

## 工具列表

当前在线 MCP 提供 3 个工具：

1. `searchHotels`：按地点、日期、星级、人数、标签等条件搜索酒店
2. `getHotelDetail`：查询指定酒店的实时房型与价格详情
3. `getHotelSearchTags`：获取可用于 `searchHotels.hotelTags` 的标签元数据

## 1) searchHotels

### 输入参数

- `originQuery` (string，必填)：用户原始提问语句。
- `place` (string，必填)：地点名称，建议尽量详细（城市/机场/景点/详细地址等）。
- `placeType` (string，必填)：地点类型，支持 `城市`、`机场`、`景点`、`火车站`、`地铁站`、`酒店`、`区/县`、`详细地址`。
- `countryCode` (string，选填)：国家二字码（ISO 3166-1），如 `CN`、`US`。
- `size` (number，选填，默认：`5`)：返回酒店数量，最大 `20`。
- `checkInParam` (object，选填)：入住相关参数。
- `filterOptions` (object，选填)：筛选参数。
- `hotelTags` (object，选填)：标签/品牌/预算筛选。

`checkInParam` 子字段：

- `adultCount` (number，选填，默认：`2`)：每间房成人数。
- `checkInDate` (string，选填，格式：`YYYY-MM-DD`)：入住日期。未传或早于当天时，自动使用“明天”；格式错误会返回参数错误。
- `stayNights` (number，选填，默认：`1`)：入住晚数。

`filterOptions` 子字段：

- `distanceInMeter` (number，选填)：与 POI 的直线距离（米），POI 场景生效时默认 `5000`。
- `starRatings` (number[]，选填)：星级范围，默认 `[0.0, 5.0]`，步长 `0.5`。

`hotelTags` 子字段：

- `preferredTags` (string[]，选填)：偏好标签。
- `requiredTags` (string[]，选填)：必须命中标签（强约束）。
- `excludedTags` (string[]，选填)：排除标签。
- `preferredBrands` (string[]，选填)：偏好品牌。
- `maxPricePerNight` (number，选填)：每晚预算上限（人民币）。
- `minRoomSize` (number，选填)：最小房间面积（平方米）。

### 输出结构

```json
{
  "message": "酒店搜索成功",
  "hotelInformationList": [
    {
      "hotelId": 43615,
      "bookingUrl": "https://rollinggo.cn/pages/hotel/detail/index?...",
      "name": "北京天伦王朝酒店(Sunworld Dynasty Hotel Beijing)",
      "brand": null,
      "address": "王府井大街50号",
      "destinationId": "6140156",
      "latitude": 39.917748,
      "longitude": 116.412249,
      "distanceInMeters": 205,
      "starRating": 5.0,
      "price": {
        "message": "查价成功,最低价:626.0, 币种:CNY",
        "hasPrice": true,
        "currency": "CNY",
        "lowestPrice": 626.0
      },
      "areaCode": "CN",
      "description": "...",
      "imageUrl": "https://image-cdn.RollingGo.com/...",
      "hotelAmenities": ["24小时前台", "WIFI"],
      "score": 1.0,
      "tags": ["临近商场", "免费WiFi"]
    }
  ]
}
```

说明：

- `price` 为对象，不是数字。
- 不同城市/供应源下，字段可能缺失或为 `null`。

## 2) getHotelDetail

### 输入参数

- `hotelId` (number，选填)：酒店 ID。与 `name` 二选一，若同时传入优先使用 `hotelId`。
- `name` (string，选填)：酒店名称（模糊匹配）。
- `dateParam` (object，选填)：入住离店日期参数。
- `occupancyParam` (object，选填)：入住人数与房间数量参数。
- `localeParam` (object，选填)：国家与币种参数。

`dateParam` 子字段：

- `checkInDate` (string，选填，格式：`YYYY-MM-DD`)：入住日期。为空/格式错误/早于当天时自动使用“明天”。
- `checkOutDate` (string，选填，格式：`YYYY-MM-DD`)：离店日期。为空/格式错误/不晚于入住日期时自动使用 `checkInDate + 1` 天。

`occupancyParam` 子字段：

- `adultCount` (number，选填，默认：`2`)：每间房成人数。
- `childCount` (number，选填，默认：`0`)：每间房儿童数。
- `childAgeDetails` (number[]，选填)：儿童年龄列表，如 `[3,5]`。
- `roomCount` (number，选填，默认：`1`)：房间数量。

`localeParam` 子字段：

- `countryCode` (string，选填，默认：`CN`)：国家二字码。
- `currency` (string，选填，默认：`CNY`)：币种。

### 输出结构

```json
{
  "success": true,
  "errorMessage": null,
  "hotelId": 43615,
  "bookingUrl": "https://rollinggo.cn/pages/hotel/detail/index?...",
  "name": "北京天伦王朝酒店(Sunworld Dynasty Hotel Beijing)",
  "checkIn": "2026-03-05",
  "checkOut": "2026-03-06",
  "roomRatePlans": [
    {
      "roomTypeId": 4984714,
      "roomName": "Superior Room",
      "roomNameCn": "高级客房",
      "ratePlanId": "7012072001634754626",
      "ratePlanName": "Superior Room King Bed , 1 King Bed",
      "bedType": 73,
      "bedTypeDescription": "未知",
      "currency": "CNY",
      "totalPrice": 0,
      "totalSalesRate": null,
      "inventoryCount": null,
      "isOnRequest": null,
      "recommendIndex": null,
      "cancellationPolicies": [
        {
          "fromDate": "2026-03-02T10:00:00+08:00",
          "toDate": null,
          "amount": 634,
          "percent": null,
          "type": null,
          "description": null
        }
      ],
      "includedFees": null,
      "excludedFees": null,
      "metadata": null
    }
  ]
}
```

说明：

- 失败时可能返回错误文本（如“获取价格失败，请稍后重试”），也可能返回结构化字段。
- `roomRatePlans` 数组可能很长，建议客户端分页或限制展示数量。

## 3) getHotelSearchTags

用于获取 `searchHotels.hotelTags` 可用标签，适合本地缓存并在客户端做意图映射。

### 输出结构

```json
{
  "tags": [
    {
      "name": "免费WiFi",
      "category": "核心设施",
      "description": "提供免费WiFi"
    }
  ],
  "usageGuide": {
    "tagUsage": "将标签名称放入 hotelTags.preferredTags（偏好）、requiredTags（必须）或 excludedTags（排除）列表中",
    "exampleRequest": "{...}"
  }
}
```

常见标签分类包括：

- 品牌与评分
- 特色卖点
- 核心设施
- 亲子家庭
- 服务细节
- 服务与餐饮
- 交通与支付
- 景观与房型
- 酒店类型
- 价格相关

## 使用示例

### 示例 1：城市搜索

```json
{
  "originQuery": "帮我找北京 2 晚 4 星以上酒店",
  "place": "北京",
  "placeType": "城市",
  "checkInParam": {
    "checkInDate": "2026-03-01",
    "stayNights": 2
  },
  "filterOptions": {
    "starRatings": [4.0, 5.0]
  },
  "size": 5
}
```

### 示例 2：带标签与预算约束

```json
{
  "originQuery": "在北京找免费WiFi、每晚1000以内的品质酒店",
  "place": "北京",
  "placeType": "城市",
  "hotelTags": {
    "preferredTags": ["免费WiFi", "品质酒店"],
    "maxPricePerNight": 1000
  },
  "size": 5
}
```

### 示例 3：查询酒店房型与价格

```json
{
  "hotelId": 43615,
  "dateParam": {
    "checkInDate": "2026-03-05",
    "checkOutDate": "2026-03-06"
  },
  "occupancyParam": {
    "adultCount": 2,
    "roomCount": 1
  },
  "localeParam": {
    "currency": "CNY",
    "countryCode": "CN"
  }
}
```

## 部署与运行

### 1. 获取源码

```bash
git clone https://github.com/RollingGo-AI/rollinggo-hotel-mcp.git
cd rollinggo-hotel-mcp
```

### 2. 安装依赖

```bash
python -m venv venv
venv\Scripts\activate   # Windows
pip install -r requirements.txt
```

### 3. 启动服务

```bash
python server.py
```

默认 MCP 端点：`http://127.0.0.1:8000/mcp`

### 4. 在 MCP 客户端通过 headers 传 API Key

当前服务端不会从 `.env` 读取 API Key，请在 MCP 客户端配置请求头：

- 推荐：`Authorization: Bearer YOUR_API_KEY`
- 兼容：`X-Secret-Key: YOUR_API_KEY`

## MCP 客户端配置示例

```json
{
  "mcpServers": {
    "RollingGo-MCP": {
      "url": "http://localhost:8000/mcp",
      "type": "http",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

## 相关链接

- API Key 申请：https://mcp.agentichotel.cn/apply
- MCP 标准：https://modelcontextprotocol.io/
