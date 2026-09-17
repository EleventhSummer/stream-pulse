# 贡献约定

一个人练习也按这个走，目的是形成肌肉记忆。

## 分支

- `main`：始终可运行，禁止直接堆功能
- `feat/<topic>`：新功能，例如 `feat/room-list`
- `fix/<topic>`：缺陷
- `docs/<topic>`：只改文档
- `chore/<topic>`：脚手架、CI、依赖

每天从最新 `main` 拉分支。一天一个 PR，不要把列表和登录塞进同一次合并。

## Commit

[Conventional Commits](https://www.conventionalcommits.org/)：

```
feat(detail): 直播中房间每 8 秒刷新
fix(http): 401 时清除 token
docs(api): 补充 404 错误码
chore(ci): 增加 typecheck 步骤
```

禁止 `update`、`fix bug`、`改了一下` 这种信息。

## 开始一项工作

1. 开 Issue，写验收标准
2. 拉分支
3. 对照 `docs/PRD.md` 和 `docs/API.md` 实现
4. 本地跑过：`npm run lint`、`npm run typecheck`、`npm run test`（有测试之后）
5. 开 PR，填模板，贴截图或 Network 证据
6. CI 绿再合并

## 代码边界

- 业务请求只放 `src/api/`
- 纯展示组件不发请求
- 不在组件里使用 `any`
- 新增接口先改 `docs/API.md`，再写代码
