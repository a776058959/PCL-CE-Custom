# PCL-CE 定制构建

本仓库**不含上游源码**，只放「补丁 + 构建配方」。每次构建时现场拉取 [PCL-Community/PCL-CE](https://github.com/PCL-Community/PCL-CE) 的最新 `dev`，打上补丁后编译发布。

## 补丁做了什么

| 改动 | 效果 |
|---|---|
| `RestrictedFeaturePolicy.Assess()` 恒定返回非受限 | 解除中国大陆地区对 Minecraft 下载、安装、补全、整合包的封锁 |
| `Config.Update.UpdateMode` 默认值改为 `Disable` | 新装版本不自动检查更新，避免被官方受限版覆盖 |

## 为什么不用 fork

fork 意味着本地背着上游的完整历史和几万个文件，每次上游更新都要 merge。这里反过来做：仓库只有几个文件，源码在构建时现拉。

- 永远不 merge，永远不冲突
- 补丁失效时构建直接红，不会静默产出一个还被限制着的包

## 什么时候构建

- 定时任务每 6 小时检查一次上游 `dev`，**只有上游有新提交时才真正编译**（比对上一次发版记录里的提交号）
- 手动：Actions → Custom build → Run workflow
- 改动了 `patches/` 或工作流本身并推送时会构建

## 版本保留策略

每次构建产出一个独立 release，标签形如 `build-20261025-1006`。

- 最近 30 天内：全部保留
- 超过 30 天：每个月只保留当月最后一版

清理在每次构建结束时自动执行，所以不需要手动删。

## 构建门禁

编译前逐条校验，任一不成立直接失败：

1. 两处补丁确实落到了源码里
2. 打过补丁后，测试目录之外不存在任何 `RegionPolicy.Evaluate` 调用（上游新增调用点即失败）
3. 下载入口 `McDownloadClient` 仍然存在
4. 配置了 `CLIENT_ID` 时，校验它真的被编进了 exe

## 密钥

仓库 Settings → Secrets and variables → Actions 里配：

| 名字 | 作用 | 必需 |
|---|---|---|
| `CLIENT_ID` | 微软登录的 OAuth 应用 ID | 想用正版登录才需要 |
| `CURSEFORGE_API_KEY` | CurseForge 资源源 | 可选，**仓库公开时不要填**，它会被编进 exe |

没有 `CLIENT_ID` 也能正常构建，只是启动器里微软登录不可用（离线登录不受影响）。

## 使用要求

按 `Plain Craft Launcher 2/LICENCE` 的重度使用条款：标明为第三方二次创作、名称以 `Plain Craft Launcher (PCL)` 开头并带第三方后缀、关于页保留龙腾猫跃署名与赞助链接、公开源代码。个人自用不分发时不受分发义务约束。