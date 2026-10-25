# 用 MCP 把俄罗斯一个地区的小旅行社接入 AI 智能体：22 个工具的实践

标签：MCP、Model Context Protocol、AI Agent、大模型应用、Claude、Cursor、旅游科技

---

## 问题：AI 不认识小商家

问大模型“堪察加有什么火山徒步线路、下周有没有位置”，它一般会给你一段概括性的介绍，然后建议你“联系当地旅行社”。原因很简单：堪察加的旅行社大多是几个人的小团队，信息散在俄语网站、Telegram 频道和电话里，搜索引擎抓不全，模型训练数据里更没有实时的余位和价格。

我们做的 Vedar（vedarai.ru）是堪察加本地的旅行平台，汇总了当地旅行社的线路、包车、租车和住宿。网站本身对人是友好的，但我们发现越来越多的用户在用 AI 助手规划旅行，于是决定把平台直接开放给 AI 智能体——用的就是 MCP（Model Context Protocol）。

## 架构：一个远程 MCP 服务器

- 接入地址：`https://vedarai.ru/api/mcp`
- 传输：Streamable HTTP
- 鉴权：无（查询类全部只读）
- 文档：https://vedarai.ru/mcp
- 仓库：https://github.com/tourhabk-ui/vedar-mcp （有中文 README）

我们没有让 MCP 去爬自己的网页，而是让工具直接读平台的业务数据：线路目录、实时余位、合作方的住宿和车辆。这样 AI 拿到的价格和日期就是旅行社在后台维护的那一份，不会出现“网页改了、AI 还在说旧价格”的问题。

## 22 个工具，分三类

`tools/list` 当前返回 22 个工具：

**1. 查询（只读）**
- `get_tours`、`get_tour_details`、`get_tour_availability`：线路、详情、实时余位
- `search_accommodations`、`search_transfers`、`search_car_rental`、`search_gear`：住宿、包车、租车、装备
- `search` + `fetch`：通用检索，兼容只认这两个工具名的客户端
- `get_place_info`：地点名称、类型、坐标

**2. 安全与环境**
- `get_weather`：按天预报（Open-Meteo），支持地名或经纬度
- `get_volcano_status`：KVERT 航空颜色代码 + 俄罗斯科学院地球物理局地震数据
- `safety_status`、`get_guardian_context`：全区预警（地震、火山、天气、紧急情况部）和某条路线的风险提示

**3. 规划与申请**
- `make_trip_plan` / `edit_trip_plan`：结合天气和余位生成按天行程，返回计划 ID，后续可以只改某一天
- `create_lead`、`create_booking_request`、`create_stay_request`、`request_charter`、`create_car_rental_request`、`get_request_status`：提交申请、查询进度

## 几个设计取舍

**不收钱，只提交申请。** 所有 `create_*` 工具只创建一份申请，由旅行社或承运人人工电话确认，平台不在线收款。对 AI 来说，“替用户下单付款”风险太大；“替用户把需求整理好交给真人”刚好合适。

**个人信息必须显式同意。** 只有参数里带 `consent: true` 时才会发送联系方式。租车申请更严格，分两步：第一步返回草稿和确认码，第二步需要用户本人明确同意才提交。

**价格只来自目录。** 我们要求工具返回的价格必须对应目录里的具体线路并附链接，不让模型自己“估一个区间”。旅游场景里，一个编出来的价格比没有价格更糟。

**行程留空天。** `make_trip_plan` 会在一周行程里主动留一天备用。堪察加的直升机和出海经常因天气取消，这是当地向导的经验，我们把它写进了规划逻辑。

**安全数据和线路放在一起。** 火山预警升级时，模型在推荐相关线路的同一轮对话里就能看到预警，而不是让用户自己去别处查。

## 接入配置

支持远程 MCP 的客户端（Claude Desktop、Cursor 等）：

```json
{
  "mcpServers": {
    "vedar": {
      "url": "https://vedarai.ru/api/mcp"
    }
  }
}
```

只支持 stdio 的客户端，用 `mcp-remote` 转一下：

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

不装客户端，先用 curl 看看工具列表：

```bash
curl -s -X POST https://vedarai.ru/api/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## 一个实际场景

今年 10 月，伊尔航空开通了哈尔滨直飞堪察加的航线（每周六），中国公民入境俄罗斯又可以免签停留 30 天。用户在 AI 助手里说“周六从哈尔滨飞堪察加，玩一周，想看火山”，智能体可以依次调用 `safety_status` 看预警、`get_tours` 和 `get_tour_availability` 找有位置的线路、`make_trip_plan` 排出“周六到周六”的行程，最后用 `create_booking_request` 把申请交给旅行社。整个过程中，价格、日期、预警都来自实时数据。

## 写在最后

对小商家来说，MCP 的意义不在于技术多新，而在于门槛低：一个 HTTP 端点，就能让所有支持 MCP 的 AI 客户端“认识”你。如果你也在做本地生活、旅游或者其他长尾行业，欢迎参考我们的仓库，也欢迎提 Issue 交流。

- 仓库：https://github.com/tourhabk-ui/vedar-mcp
- 中文站：https://vedarai.ru/zh
