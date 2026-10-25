# PCL-CE 定制构建

这个仓库**不含上游源码**，只放「补丁 + 构建配方」。每次构建时现场拉取上游最新代码，打上补丁，编译并发版。

## 结构

```
patches/0001-custom-build.patch   构建期补丁（不进上游，不进本仓库的源码）
.github/workflows/custom-build.yml 拉上游 → 打补丁 → 校验 → 编译 → 发布
```

## 补丁做了什么

- `RestrictedFeaturePolicy.Assess()` 恒定返回非受限 → 解除大陆地区对 Minecraft 下载/安装/补全/整合包的限制
- `Config.Update.UpdateMode` 默认值改为 `Disable` → 新装版本不自动检查更新，避免被官方受限版覆盖

## 为什么这样比 fork 好

- 仓库里没有上游那 100MB 历史和几万个文件，永远不需要 merge，永远不会冲突
- 上游改了什么都不用管；补丁失效时构建直接红，不会静默产出一个还被限制着的包

## 构建前门禁

1. 补丁里的两处改动确实落到源码里
2. 打过补丁后，测试目录外不存在任何 `RegionPolicy.Evaluate` 调用（上游新增调用点即失败）
3. 下载入口 `McDownloadClient` 仍存在
4. 配置了 `CLIENT_ID` 时校验它真的编进了 exe

## 触发方式

- 每天 12:00（北京时间）自动构建一次
- 手动：Actions → Custom build → Run workflow

## 使用要求

按 `Plain Craft Launcher 2/LICENCE`（重度使用条款）：标明第三方二次创作、关于页保留龙腾猫跃署名与赞助链接、公开源代码。个人自用不分发时不受分发义务约束。