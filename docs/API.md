# Stream Pulse API

Base URL（开发）：`/api`（由 Vite proxy 转到 mock `http://127.0.0.1:3456`）

约定：

- JSON，UTF-8
- 成功：HTTP 2xx，body 为业务数据
- 失败：HTTP 4xx/5xx，body 统一为 `{ "error": { "code": string, "message": string } }`
- 需登录的接口：`Authorization: Bearer <token>`
- 缺 token 或 token 无效：`401`，`code = UNAUTHORIZED`

Mock 延迟：每个接口随机 200–500ms，便于看到 loading。直播中房间的 `viewerCount` 每次请求可 ±20 波动，方便验证轮询。

---

## POST /api/login

无需登录。

请求：

```json
{ "username": "demo", "password": "demo123" }
```

成功 `200`：

```json
{
  "token": "mock-token-demo",
  "user": { "id": "u_1", "name": "Demo Operator" }
}
```

失败 `401`：

```json
{ "error": { "code": "INVALID_CREDENTIALS", "message": "用户名或密码错误" } }
```

---

## GET /api/rooms

需登录。

Query：

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| q | string | 空 | 匹配 `title` 或 `id`（包含即可） |
| status | `live` \| `offline` \| `banned` | 空 | 空表示全部 |
| page | number | 1 | 从 1 开始 |
| pageSize | number | 6 | 固定 6，服务端忽略其他值也可 |

成功 `200`：

```json
{
  "items": [
    {
      "id": "r_1001",
      "title": "星河杯淘汰赛",
      "cover": "/covers/1001.png",
      "host": "Nova",
      "status": "live",
      "viewerCount": 12800,
      "category": "游戏"
    }
  ],
  "total": 12,
  "page": 1,
  "pageSize": 6
}
```

`status` 枚举：`live` | `offline` | `banned`。

---

## GET /api/rooms/:id

需登录。

成功 `200`：

```json
{
  "id": "r_1001",
  "title": "星河杯淘汰赛",
  "cover": "/covers/1001.png",
  "host": "Nova",
  "status": "live",
  "viewerCount": 12840,
  "likeCount": 3560,
  "startedAt": "2026-09-15T10:00:00.000Z",
  "category": "游戏",
  "viewerTrend": [10200, 10800, 11100, 11500, 11800, 12000, 12200, 12400, 12500, 12600, 12700, 12840],
  "comments": [
    {
      "id": "c_1",
      "user": "alice",
      "content": "这波节奏好",
      "createdAt": "2026-09-15T12:01:00.000Z"
    }
  ]
}
```

说明：

- `viewerTrend` 长度固定 12，时间从旧到新
- `comments` 最多 10 条，新的在前
- `status !== live` 时 `startedAt` 为 `null`，`viewerCount` 为 0
- 不存在：`404`，`code = ROOM_NOT_FOUND`，`message = 房间不存在`

---

## 前端实现注意

1. 列表和详情的字段不要混用一个过于宽松的类型；列表项用 `RoomSummary`，详情用 `RoomDetail extends RoomSummary`。
2. 图表只吃 `viewerTrend`，不要在页面里再随机生成点数。
3. 生产环境若无法访问 mock，使用与本文件结构一致的静态数据，并在 `docs/ARCHITECTURE.md` 声明切换方式。
