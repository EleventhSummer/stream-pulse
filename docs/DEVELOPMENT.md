# 本地开发笔记

从 Day 1 起往这里补。只写你亲口验证过的事实。

## 环境

- Node：v24.21.0
- 包管理：npm

## 启动

```bash
npm install
# mock 是 Day2 的，现在跑会失败
npm run dev
```

（mock 脚本在 Day 2 加上。）

## 我搞懂的工具

| 工具                | 它解决什么               | 我验证过的现象                         |
| ----------------- | ------------------- | ------------------------------- |
| Vite | 把Vue/TS变成浏览器能打开的网页  | npm run dev 后打开网页能够看到欢迎页面 |
| TypeScript | 用类型当合同，尽量在运行前发现字段错误 | npm run type-check 能够跑完，没有报错 |
| Vue Router | 让网址和页面一一对应，刷新、分享才有意义 | 欢迎页面上有Home/About，点击会更换页面，地址栏会更改 |
| Pinia | 存放多个页面都能用的数据，避免每个组件自己存一份 | 官方示例有个counter store，业务状态暂未用上 |
| proxy / CORS | 开发时把前端接口请求转给后端，避开浏览器跨域限制 | 未做到 |
| ESLint / Prettier | ESLint查有没有明显写错，Prettier把代码格式统一，机器代劳 | lint和format都能通过 |
| Vitest | 自动跑小测试，避免每次改代码都靠手点页面 | 未做 |
| GitHub Actions    |  在GitHub上自动跑检查，不依赖我的电脑 | 未做 |

## 踩坑

- 标题：Issue模版未出现
- 现象：/issues/new是空白描述
- 原因：放成了.github/ISSUE_TEMPLATE.md文件，Github要的是文件夹
- 处理：改成.github/ISSUE_TEMPLATE/task.md，并加上name/about

## 关键文件

- package.json:
scripts是我在终端跑的命令，例如dev会调用Vite；dependencies是网页真正要使用的库（vue，vue-router，pinia）；devDependencies是开发时才使用的（vite，eslint）。
- vite.config.ts：
这里配了vue()插件，所以.vue能够被处理。@指向src。
- tsconfig.json
将检查分给旁边几个tsconfig，验证过npm run type-check（vue-tsc）能够通过。
- src/main.ts：
createApp(App)创建应用，app.use(router),app.use(pinia)挂上插件，最后mount('#app')挂到页面上。
- src/router/index.ts
/对应首页，/about对应About。点击欢迎页面链接时地址栏会变，靠的就是这份路由表。
- src/stores/counter.ts
用来存放跨组件能够共享的数据，现在只是示例计数器，请求不写在该处
- index.html
浏览器真正打开的入口，里面的<div id="app">还有指向src/main.ts的script，所以入口不是某个.vue。
