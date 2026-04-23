# ChatGPT Mirror

一个按 `dairoot/ChatGPT-Mirror` 路由形态整理的三层项目：

- `gateway/`：Rust 网关，负责镜像站流量承接、ChatGPT 上游反代、mirror token 登录态下发。
- `frontend/`：Vue 管理后台，构建后挂载到 `/admin/`。
- `backend/`：已内置的 Django 后台，负责 `/0x/*`。


因此默认直接执行：

```bash
docker-compose up -d 
```

即可在同一个项目里把前端、gateway、Django、`CloudFlare5sBypass` 一起拉起。
