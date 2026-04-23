# ChatGPT Mirror

一个按 `dairoot/ChatGPT-Mirror` 路由形态整理的三层项目：

- `gateway/`：Rust 网关，负责镜像站流量承接、ChatGPT 上游反代、mirror token 登录态下发。
- `frontend/`：Vue 管理后台，构建后挂载到 `/admin/`。
- `backend/`：已内置的 Django 后台，负责 `/0x/*`。

当前前端已按 `dairoot/ChatGPT-Mirror` 的 Django 路由做兼容调整：

- 使用 `/0x/chatgpt/car-enum`
- 不再依赖本地扩展出来的 `/0x/models/*`
- 用户模型限制直接按字符串列表提交给 Django

## 站点层路由

部署层至少需要这三段路由：

- `/admin/*` -> 前端后台
- `/0x/*` -> Django 后台
- `/*` -> Rust `gateway`

当前前端构建基路径已经固定为 `/admin/`，静态资源也会落在 `/admin/assets/*`。

## Gateway 职责

Rust `gateway` 对外提供两类能力：

1. 直接暴露的网关接口
2. 其余 ChatGPT 站点路径与上游 API 的透明反代

### 网关接口

默认接口如下：

- `/v1/chat/completions`
- `/api/livekit`
- `/v1/audio/speech`
- `/api/login`
- `/api/logout`
- `/api/get-mirror-token`
- `/api/get-user-info`
- `/api/get-user-use-count`
- `/api/get-chatgpt-use-count`
- `/api/not-login?user_gateway_token=<mirror_token>`

代码中还包含一个 Django 后台主动调用、但原始文档常漏写的接口：

- `/api/close-chatgpt-memory`

如果设置了 `MIRROR_API_PREFIX`，以上接口会整体带前缀。例如 `MIRROR_API_PREFIX=/mirror` 时：

- `/mirror/api/login`
- `/mirror/api/not-login?user_gateway_token=<mirror_token>`
- `/mirror/v1/chat/completions`

## Django 后台接口

站点层把 `/0x/*` 转给 Django。当前文档约束的接口集合如下。

### `/0x/user/*`

- `/0x/user`
- `/0x/user/version-cfg`
- `/0x/user/get-mirror-token`
- `/0x/user/register`
- `/0x/user/login-free`
- `/0x/user/chatgpt-list`
- `/0x/user/batch-model-limit`
- `/0x/user/relat-gptcar`
- `/0x/user/login`
- `/0x/user/visit-log`

### `/0x/chatgpt/*`

- `/0x/chatgpt`
- `/0x/chatgpt/enum`
- `/0x/chatgpt/login`
- `/0x/chatgpt/car`
- `/0x/chatgpt/car-enum`

## ChatGPT 上游说明

Mirror 文档里明确引用的取 token 路径：

- `https://chatgpt.com/api/auth/session`

仓库当前 Rust 网关也按这个路径处理 `session_token -> access_token`。

### 当前公开可观察到的匿名/页面路径

- `https://chatgpt.com/`
- `https://chatgpt.com/public-api/conversation_limit`
- `https://chatgpt.com/backend-anon/me`
- `https://chatgpt.com/backend-anon/models`
- `https://chatgpt.com/backend-anon/conversation/init`
- `https://chatgpt.com/backend-anon/prompt_library/?limit=4&offset=0`
- `https://chatgpt.com/backend-anon/prompt_library/?limit=8&offset=0&use_v2=true&model_slug=auto`
- `https://chatgpt.com/backend-anon/prompt_library/?limit=4&offset=0&model_slug=auto`

### 配置/实验/埋点

- `https://chatgpt.com/ces/v1/projects/oai/settings`
- `https://chatgpt.com/ces/v1/p`
- `https://chatgpt.com/ces/v1/t`
- `https://ab.chatgpt.com/v1/initialize`
- `https://ab.chatgpt.com/v1/rgstr`

### 已登录 `backend-api` 常见业务路径

- `https://chatgpt.com/backend-api/models`
- `https://chatgpt.com/backend-api/me`
- `https://chatgpt.com/backend-api/conversation/init`
- `https://chatgpt.com/backend-api/conversations?offset=0&limit=28`
- `https://chatgpt.com/backend-api/conversation/<conversation_id>`
- `https://chatgpt.com/backend-api/conversation`
- `https://chatgpt.com/backend-api/moderations`
- `https://chatgpt.com/backend-api/conversation/gen_title/<conversation_id>`
- `https://chatgpt.com/backend-api/conversation/message_feedback`
- `https://chatgpt.com/backend-api/conversations`
- `https://chatgpt.com/backend-api/share/create`
- `https://chatgpt.com/backend-api/share/<share_id>`
- `https://chatgpt.com/backend-api/shared_conversations?order=created`
- `https://chatgpt.com/backend-api/user_system_messages`
- `https://chatgpt.com/backend-api/accounts/check/v4-2023-04-27`
- `https://chatgpt.com/backend-api/accounts/data_export`

### Sentinel / Codex / 响应流

- `https://chatgpt.com/backend-api/sentinel/chat-requirements`
- `https://chatgpt.com/backend-api/sentinel/chat-requirements/prepare`
- `https://chatgpt.com/backend-api/sentinel/chat-requirements/finalize`
- `https://chatgpt.com/backend-api/sentinel/sdk.js`
- `https://chatgpt.com/backend-api/codex/responses`
- `https://chatgpt.com/backend-api/codex/responses/compact`
- `wss://chatgpt.com/backend-api/codex/responses`
- `https://chatgpt.com/backend-api/wham/usage`

### Next.js 页面数据与页面路由

- `https://chatgpt.com/_next/data/<build_id>/c/<conversation_id>.json?chatId=<conversation_id>`
- `https://chatgpt.com/_next/data/<build_id>/index.json`
- `https://chatgpt.com/_next/data/<build_id>/share/<share_id>/continue.json?shareParams=<share_id>&shareParams=continue`
- `https://chatgpt.com/c/<conversation_id>`

### Cloudflare challenge / 基础设施路径

这些路径会被网关原样反代，但它们不属于核心业务 API：

- `https://chatgpt.com/cdn-cgi/challenge-platform/scripts/jsd/main.js`
- `https://chatgpt.com/cdn-cgi/challenge-platform/h/g/scripts/jsd/f9063374b04d/main.js`
- `https://chatgpt.com/cdn-cgi/challenge-platform/h/g/jsd/r/8f082c7d9deb2d53`

### 静态资源样例

这些文件名带哈希，不能当成稳定全量集合：

- `https://cdn.oaistatic.com/assets/conversation-small-kq10986g.css`
- `https://cdn.oaistatic.com/assets/manifest-7d43a138.js`
- `https://cdn.oaistatic.com/assets/mxkyxjre6muko6z4.js`
- `https://cdn.oaistatic.com/assets/nqo5y2f0dorhrqsr.js`
- `https://cdn.oaistatic.com/assets/fpwmsu1awpj0g2ko.js`
- `https://cdn.oaistatic.com/assets/dgcxf4c1lo6y3h3a.js`
- `https://cdn.oaistatic.com/assets/nb34aa8izknzna97.js`
- `https://cdn.oaistatic.com/assets/l697z2ouob9b6hw7.js`
- `https://cdn.oaistatic.com/assets/k56enwh74zn4hbwt.js`
- `https://cdn.oaistatic.com/assets/gtbc1g1q4ztw05rv.js`
- `https://cdn.oaistatic.com/assets/dvl2tfqalthh42cv.js`
- `https://cdn.oaistatic.com/assets/cb0x1wlgm93n2hpu.js`
- `https://cdn.oaistatic.com/assets/buun9i8g5c97ea0e.js`

## 运行

### 前端开发

```bash
cd frontend
npm install
npm run dev
```

开发时访问 `http://localhost:40003/admin/`。

### Gateway 开发

```bash
cd gateway
cargo run
```

### Docker

```bash
docker-compose up -d
```

## 关键环境变量

### 网关自身

- `HOST`：监听地址，默认 `0.0.0.0`
- `PORT`：监听端口，默认 `40002`
- `DATABASE_PATH`：SQLite 路径
- `ADMIN_PASSWORD`：默认管理密钥
- `GATEWAY_ADMIN_SECRET`：网关后台调用密钥，未设置时回退到 `ADMIN_PASSWORD`
- `MIRROR_API_PREFIX`：为网关接口整体加前缀

### 外部上游 / 内部编排

- `DJANGO_UPSTREAM`：`/0x/*` 反代目标；当前 `docker-compose.yml` 默认接到内置 `django` 服务
- `ADMIN_UPSTREAM`：如需把 `/admin/*` 交给独立前端服务，可配置此项；未配置时网关直接托管本地 `static/`
- `CHATGPT_BASE_URL`：默认 `https://chatgpt.com`
- `CHATGPT_CDN_BASE_URL`：默认 `https://cdn.oaistatic.com`
- `CHATGPT_AB_BASE_URL`：默认 `https://ab.chatgpt.com`
- `CF_BYPASS_URL`：`CloudFlare5sBypass` 服务地址，默认 `http://cfbypass:8000`
- `CF_BYPASS_PROXY_SERVER`：转发给 `CloudFlare5sBypass` 的代理地址
- `REQUEST_TIMEOUT_SECS`：网关请求超时

更多网关实现说明见 [`gateway/README.md`](./gateway/README.md)。

## Docker 编排

当前项目内置三类服务：

- `chatgpt-mirror`：Rust gateway + `/admin/` 静态后台
- `django`：本地 Django 后台，处理 `/0x/*`
- `cfbypass`：`CloudFlare5sBypass`

因此默认直接执行：

```bash
docker-compose up -d --build
```

即可在同一个项目里把前端、gateway、Django、`CloudFlare5sBypass` 一起拉起。
