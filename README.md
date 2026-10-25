# PCL-CE 定制构建

本仓库**不含上游源码**，只放「补丁 + 构建配方」。每次构建时现场拉取 [PCL-Community/PCL-CE](https://github.com/PCL-Community/PCL-CE) 的**正式发版**（最新 `v*` tag），打上补丁后编译发布。

## 补丁做了什么

| 改动 | 效果 |
|---|---|
| `RestrictedFeaturePolicy.Assess()` 恒定返回非受限 | 解除中国大陆地区对 Minecraft 下载、安装、补全、整合包的封锁 |

## 为什么不用 fork

fork 意味着本地背着上游的完整历史和几万个文件，每次上游更新都要 merge。这里反过来做：仓库只有三个文件，源码在构建时现拉。

- 永远不 merge，永远不冲突
- 补丁失效时构建直接红，不会静默产出一个还被限制着的包

## 什么时候构建

- 定时每 8 小时检查一次上游的**最新正式发版**（GitHub Releases 的 `latest`，自动跳过 beta）
- 判定依据是仓库里的 `.github/upstream-lock.txt`（上次成功构建的上游 tag），构建成功后由 `record` job 自动写回并提交，同一个版本不会重复构建
- 判定不看 release 列表，所以清理旧版本不会影响「是否需要构建」
- 手动：Actions → Custom build → Run workflow；可在 `ref` 里指定上游 tag 或分支（例如临时要 dev 的最新修复就填 `dev`）

发版与上游保持一致：tag 直接用上游的 `v2.15.1`，标题用上游的 `Release 2.15.1`，更新说明取上游那一版的原文，前面加三行说明本版是自行编译、已解除区域限制。

## 版本保留策略

- 最近 30 天内：全部保留
- 超过 30 天：每个月只保留当月最后一版

每次构建结束时自动清理，不需要手动删。

## 启动器内更新

构建产物自己带更新源，指向本仓库：

- 补丁把 `UpdateManager.remoteServer` 换成本仓库的 raw 地址，不再查上游（自动检查更新是开着的，检查的就是本仓库）
- 每次构建自动生成 `apiv2/cache.json`、`apiv2/updates/updates-srx64.json`、`apiv2/announcement.json` 并提交
- 更新包 `PCL-CE-Custom-<tag>-x64.zip` 随 Release 一起发布（资产名带版本号，例如 `PCL-CE-Custom-v2.15.1-x64.exe`），`sha256` 字段填的是**解压后 exe** 的哈希（上游更新器就是这么校验的）

所以启动器里点「检查更新」查的是本仓库的最新版本；上游发新版后，本仓库生成对应版本，启动器就能直接更新到新的定制版。

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