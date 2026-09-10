# 酷狗签到

GitHub Actions 实现「酷狗概念版 VIP」自动领取：每天自动听歌领 VIP + 领取 8 次广告 VIP，总计两天酷狗概念版 VIP。

> 登录方式仅保留**手机号 + 验证码**（一个手机号绑定多个酷狗账号时无法用此方式登录，见 [多账号登录问题](https://github.com/MakcRe/KuGouMusicApi/issues/51)）。

## 免责声明

> [!important]
> 1. 本项目仅供学习使用，请尊重版权，请勿利用此项目从事商业行为及非法用途!
> 2. 使用本项目的过程中可能会产生版权数据。对于这些版权数据，本项目不拥有它们的所有权。为了避免侵权，使用者务必在 24小时内清除使用本项目的过程中所产生的版权数据。
> 3. 由于使用本项目产生的包括由于本协议或由于使用或无法使用本项目而引起的任何性质的任何直接、间接、特殊、偶然或结果性损害（包括但不限于因商誉损失、停工、计算机故障或故障引起的损害赔偿，或任何及所有其他商业损害或损失）由使用者负责。
> 4. **禁止在违反当地法律法规的情况下使用本项目。**
> 5. 音乐平台不易，请尊重版权，支持正版。

## 快速开始

### 1. Fork 本仓库

### 2. 创建 PAT（Personal Access Token）

打开 <https://github.com/settings/personal-access-tokens/new>：

- **Token name**：随意
- **Expiration**：按需选择（不建议过长）
- **Repository access**：只勾选你 fork 的这个仓库
- **Repository permissions**：`Metadata` 只读，`Secrets` 读写

生成后复制 token，到本仓库 Settings → Secrets and variables → Actions → New repository secret，变量名 `PAT`。

### 3. 手机号登录（两步）

1. 添加 Secret `PHONE`（你的酷狗绑定手机号）
2. 运行 Actions「**手机号登录**」→ 操作步骤选「**发送验证码**」→ 手机收到验证码
3. 把验证码填入 Secret `CODE`
4. 再次运行 Actions「**手机号登录**」→ 操作步骤选「**登录**」→ 登录信息自动写入 Secret `USERINFO`

多账号：重复上述流程，登录那一步把 `append_user` 选择「是」即可追加而不覆盖。

### 4. 启用 Actions

到 Actions 标签页启用「**签到**」工作流（每天北京时间 01:10 自动执行）。

**必须同时启用「仓库保活」**：GitHub 对 60 天无提交的仓库会自动停用定时 Actions，「仓库保活」每月 1 号自动提交一次保活记录，保证签到长期执行。

## 工作原理

```
签到.yml (每天 01:10)
 └─ main.js
     ├─ 启动本地 KuGouMusicApi 服务（api/，精简为 7 个接口）
     ├─ 读取 Secret USERINFO（userid + token）
     ├─ /youth/listen/song      听歌领取 VIP
     ├─ /youth/vip × 8          广告领取 VIP（每次间隔 30s）
     ├─ /user/vip/detail        查询 VIP 到期时间
     └─ 周日: /login/token 刷新 token → 用 PAT 写回 Secret USERINFO
```

## Secrets 一览

| Secret | 用途 | 必填 |
| --- | --- | --- |
| `PAT` | 精细化令牌（Secrets 读写权限），用于自动写入 USERINFO / 刷新 token | 是 |
| `PHONE` | 酷狗绑定手机号 | 登录时 |
| `CODE` | 短信验证码 | 登录时 |
| `USERINFO` | 登录信息数组（登录成功后自动生成/更新） | 自动维护 |

## 常见问题

- **提示 token 过期**：重新走一遍「手机号登录」流程即可；配置了 PAT 时每个周日会自动刷新 token，正常情况下无需手动处理
- **听歌领取失败**：到 APP 活动中心 → 天天签到领 VIP，确认当天名额状态（该活动新用户可能没有）
- **定时没跑**：检查仓库 Actions 是否被手动禁用；查看「仓库保活」是否正常
- **改签到时间**：编辑 `.github/workflows/签到.yml` 的 cron（UTC 时间，北京时间 -8）

## 致谢

- [MakcRe/KuGouMusicApi](https://github.com/MakcRe/KuGouMusicApi) 提供 API 源码（本仓库已精简）
- [develop202/kgcheckin](https://github.com/develop202/kgcheckin) 原项目
