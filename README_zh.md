# Vedar MCP — 面向 AI 智能体的堪察加旅行服务

[English](README.md) | 简体中文

**[Vedar（Ведар）](https://vedarai.ru)** 的远程 [Model Context Protocol](https://modelcontextprotocol.io) 服务器。Vedar 是俄罗斯堪察加半岛的旅行平台，提供当地旅行社的线路、包车接送、租车、住宿、按天行程规划，以及实时的火山、天气和安全信息。

- **接入地址：** `https://vedarai.ru/api/mcp`
- **传输方式：** Streamable HTTP
- **鉴权：** 无需
- **文档：** https://vedarai.ru/mcp

查询类工具均为只读。申请类工具（`create_*`、`request_charter`）只会提交一份申请，由人工客服或旅行社通过电话确认——平台不收取任何款项；只有在旅行者明确同意（`consent: true`）时才会发送个人信息。

## 从哈尔滨出发

- **直飞航班：** IrAero（伊尔航空）IO-771/772，哈尔滨 ↔ 堪察加彼得罗巴甫洛夫斯克，**每周六**执飞，全年运营，飞行约 3.5 小时（首航 2026 年 10 月 24 日）。来源：[IrAero 新闻](https://iraero.ru/about/news/pryamoy-reys-kamchatka-kharbin-iraero-otkryla-prodazhu-biletov/)、[航线时刻](http://iraero.ru/routs/petropavlovsk-kamchatskiy_kharbin/)。
- **免签：** 中国公民可免签入境俄罗斯，单次停留不超过 30 天（俄罗斯总统令第 872 号，有效期延长至 2027 年 12 月 31 日）。来源：[克里姆林宫](http://kremlin.ru/acts/bank/52641/print)、[俄罗斯驻华使馆](https://newchina.mid.ru/ru/consular-services/vezd_i_vizy_v_rossiyskuyu_federatsiyu/temporary_visafree/)。
- 周六上午抵达，正好安排"周六到周六"的一周行程：可以用 `make_trip_plan` 生成按天计划，用 `get_tours` 和 `get_tour_availability` 查看线路与余位。

## 工具

以下为线上接入地址 `tools/list` 返回的 22 个工具：

- `get_tours` — 查询堪察加旅游线路：当地旅行社的线路、价格和日期；实时余位请用 get_tour_availability。
- `get_tour_details` — 单条线路详情：行程安排、费用包含项、装备清单、安全提示。
- `get_guardian_context` — 某地点或路线的安全信息：风险、当前预警和建议。
- `get_place_info` — 地点基本信息：名称、类型、坐标。
- `get_weather` — 按天天气预报（Open-Meteo），可按堪察加地名或经纬度查询。
- `get_volcano_status` — 火山活动：KVERT 航空颜色代码及俄罗斯科学院地球物理局（KB GS RAS）地震数据。
- `search_accommodations` — 查询平台合作方的酒店和住宿。
- `search_transfers` — 包车接送（越野卡车 vakhtovka、中巴、吉普），按目的地或按小时/天整车计价，不按座位。
- `search_car_rental` — 自驾租车（轿车、SUV、四驱、中巴）：日租价格和押金。
- `search_gear` — 平台合作方的装备租赁。
- `safety_status` — 堪察加全区安全状况：当前预警（地震、火山、天气、紧急情况部 MChS）及来源和日期，以及活跃火山列表。
- `make_trip_plan` — 结合天气和实时余位生成按天行程，返回计划 ID 供 edit_trip_plan 使用。
- `edit_trip_plan` — 按 ID 修改行程：增加、删除或调整某天，更改住宿档次，或处理航班延误；未改动的天数保持不变。
- `get_tour_availability` — 线路实时余位：按日期显示空位和最近可订日期。
- `search` — 关键词搜索线路、地点、住宿、路线，最多返回 10 条结果（含 id、标题和链接）。
- `fetch` — 按 search 返回的 id 获取完整内容：介绍、价格、行程、安全信息和链接。
- `get_request_status` — 按申请编号查询处理状态：旅行社答复或受理人。
- `create_lead` — 尚未选定线路或日期时提交选线申请；由客服人工确认，不付款。
- `create_booking_request` — 预订指定日期的线路：先检查实时余位，由旅行社人工确认，不付款。
- `create_stay_request` — 住宿申请：日期和人数发送给房东，由房东人工确认，不付款。
- `request_charter` — 向包车承运人提交整车申请：目的地、日期、人数；人工确认，不付款。
- `create_car_rental_request` — 两步式自驾租车申请：第一步返回草稿和确认码，第二步须本人明确同意后才提交。

## 接入

### Claude Desktop / Cursor / 任何支持远程 MCP 的客户端

```json
{
  "mcpServers": {
    "vedar": {
      "url": "https://vedarai.ru/api/mcp"
    }
  }
}
```

### 仅支持 stdio 的客户端

```json
{
  "mcpServers": {
    "vedar": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://vedarai.ru/api/mcp"]
    }
  }
}
```

### npm 配置助手

npm 上的 [`vedar-mcp`](https://www.npmjs.com/package/vedar-mcp)——本服务器的配置助手。

### 快速测试

```bash
curl -s -X POST https://vedarai.ru/api/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## 链接

- 网站（中文）：https://vedarai.ru/zh
- MCP 页面：https://vedarai.ru/mcp
- Glama：https://glama.ai/mcp/connectors/ru.vedarai/mcp
- 文章（英文）：[Why AI assistants don't know small businesses — and how we opened Kamchatka to agents with MCP](https://dev.to/vedarai/why-ai-assistants-dont-know-small-businesses-and-how-we-opened-kamchatka-to-agents-with-mcp-5gio)
- 文章（英文）：[A 7-Day Kamchatka Trip Plan That Leaves a Day Empty on Purpose](https://dev.to/vedarai/a-7-day-kamchatka-trip-plan-that-leaves-a-day-empty-on-purpose-cke)

## 许可证

MIT
