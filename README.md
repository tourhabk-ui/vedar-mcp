# Vedar MCP — Kamchatka travel for AI agents

Remote [Model Context Protocol](https://modelcontextprotocol.io) server of **[Vedar](https://vedarai.ru)** — a Kamchatka (Russia) travel platform with tours from local operators, transfers, car rental, lodging, a trip planner and live volcano / weather / safety data.

- **Endpoint:** `https://vedarai.ru/api/mcp`
- **Transport:** Streamable HTTP
- **Auth:** none
- **Docs:** https://vedarai.ru/mcp

Lookup tools are read-only. The request tools (`create_*`, `request_charter`) only send a request that a human manager/operator confirms by phone — there is no payment through the platform, and personal data is sent only with the traveller's explicit consent (`consent: true`).

## Tools

Tool list as returned by `tools/list` on the live endpoint:

- `get_tours` — Use when the user asks about Kamchatka (Russia) tours: operator tours with prices and dates; live availability via get_tour_availability.
- `get_tour_details` — One Kamchatka tour in depth: program, what is included, what to bring, safety notes.
- `get_guardian_context` — Kamchatka safety context for a place or route: hazards, active alerts, advice.
- `get_place_info` — Kamchatka place facts only: what it is, type, coordinates.
- `get_weather` — Daily weather forecast (Open-Meteo) for a Kamchatka place by name, or for any point by latitude/longitude.
- `get_volcano_status` — Use when the user asks about Kamchatka volcano activity: KVERT aviation code and KB GS RAS seismicity.
- `search_accommodations` — Use when the user asks about hotels or accommodation in Kamchatka, Russia: stays from platform partners.
- `search_transfers` — Use when the user asks about Kamchatka transfers: whole-vehicle charters (vakhtovka, minibus, jeep), price per vehicle by destination or hour/day, not per seat.
- `search_car_rental` — Use when the user wants to rent a car, SUV, 4x4 or minibus in Kamchatka (car rental, car hire): self-drive, price per day, deposit.
- `search_gear` — Gear rental in Kamchatka from platform partners.
- `safety_status` — Kamchatka regional safety status: active alerts (seismic, volcanic, weather, MChS) with their source and dates, plus elevated volcanoes (KVERT, KB GS RAS).
- `make_trip_plan` — Day-by-day Kamchatka trip plan with weather and live availability; returns a plan ID for edit_trip_plan.
- `edit_trip_plan` — Edit a Kamchatka trip plan from make_trip_plan by its ID: add, remove or move a day, change lodging level, or apply a flight delay the traveller reports; untouched days stay as they were.
- `get_tour_availability` — Use when the user asks if a Kamchatka tour has free seats: live availability by date, free seats and nearest open dates.
- `search` — Keyword search across Vedar Kamchatka: operator tours, places, stays, routes; returns up to 10 results with id for fetch, title and canonical url.
- `fetch` — Full text of one Vedar Kamchatka record by id from search (tour, place, stay, route): description, price, program, safety facts, canonical url.
- `get_request_status` — Status of a Kamchatka request by the key from create_lead, create_booking_request, create_stay_request, request_charter or create_car_rental_request: operator answer or who got it.
- `create_lead` — Tour-selection request for Kamchatka when no tour or date is chosen yet; human-confirmed by a manager, no payment.
- `create_booking_request` — Use when the user wants to book a Kamchatka tour on a date: live availability checked first, human-confirmed by the operator, no payment.
- `create_stay_request` — Kamchatka stay request: dates and guests go to the owner, human-confirmed by the owner, no payment.
- `request_charter` — Whole-vehicle (vakhtovka, minibus) request to a Kamchatka charter carrier: destination, dates, group size; human-confirmed by a manager, no payment.
- `create_car_rental_request` — Two-step Kamchatka self-drive car rental request: step 1 returns a draft and confirmation code; step 2 only after the person explicitly agrees.

## Connect

### Claude Desktop / Cursor / any client with remote MCP

```json
{
  "mcpServers": {
    "vedar": {
      "url": "https://vedarai.ru/api/mcp"
    }
  }
}
```

### Clients that only support stdio

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

### npm config helper

[`vedar-mcp`](https://www.npmjs.com/package/vedar-mcp) on npm — a config helper for this server.

### Quick test

```bash
curl -s -X POST https://vedarai.ru/api/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Links

- Website: https://vedarai.ru
- MCP page: https://vedarai.ru/mcp
- Glama: https://glama.ai/mcp/connectors/ru.vedarai/mcp
- Article: [Why AI assistants don't know small businesses — and how we opened Kamchatka to agents with MCP](https://dev.to/vedarai/why-ai-assistants-dont-know-small-businesses-and-how-we-opened-kamchatka-to-agents-with-mcp-5gio)
- Article: [A 7-Day Kamchatka Trip Plan That Leaves a Day Empty on Purpose](https://dev.to/vedarai/a-7-day-kamchatka-trip-plan-that-leaves-a-day-empty-on-purpose-cke)

## По-русски

**Ведар** — платформа путешествий по Камчатке. MCP-сервер `https://vedarai.ru/api/mcp` (Streamable HTTP, без авторизации) даёт ИИ-ассистентам туры местных операторов с реальными ценами и свободными местами, трансферы, прокат авто, жильё, планировщик поездки по дням, погоду, состояние вулканов (KVERT, КФ ЕГС РАН) и обстановку по безопасности. Заявки подтверждает человек по телефону, оплаты через платформу нет. Подробнее: https://vedarai.ru/mcp

## License

MIT
