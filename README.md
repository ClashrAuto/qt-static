# qt-static

Coast 用的**预编静态 Qt**产物仓库。这里不放源码，只放构建好的 Qt，供
[ClashrAuto/clashauto](https://github.com/ClashrAuto/clashauto) 及其外部签名仓库下载。

## 为什么单独一个仓库

- **不能放 clashauto 自己的 release**：app 的更新列表读 `/releases` 全量，
  `UpdateController::fetchReleases` 取的是列表里第一个非 prerelease 的 release，
  **不看它有没有可用资产**。一个 `qt-static-*` 的 tag 会被当成「最新版本」，
  而它的 tag 不是版本号，版本比较会乱套 —— 已经装出去的老客户端改不了。
- **不能只靠 GitHub Actions 缓存**：缓存按分支隔离（功能分支上编好的，合并回 master 用不上）、
  7 天不访问即淘汰、且跨不了仓库（外部签名仓库只能自己从头编）。
- 本仓库**公开**，所以下载端不需要任何凭据；发布端用本仓库自带的 `GITHUB_TOKEN`，
  也不需要任何 PAT 或 secret。

## 资产命名

    qtstatic-<Qt版本>-<平台>-<配方哈希>.tar.gz

`<配方哈希>` = clashauto 里对应平台配方脚本（`.github/actions/qt-static/build-{posix.sh,windows.ps1}`）
sha256 的前 16 位。**配方一改就是新资产名**，所以不会出现「改了配方却用了旧 Qt」——
那类 bug 的表现是两边都绿而链的是上一版产物，极难发现。

包内是 Qt 安装前缀的**内容**（不多一层目录），解包端 `tar -xzf` 直接铺进前缀即可。

## 怎么更新

到 Actions 里手动跑 **Build static Qt**。配方与模块集都取自 clashauto 的 master，
本仓库不保存任何构建逻辑的副本 —— 只有一份配方，不会漂移。

Windows 那两套一轮编不完 GitHub 的 6 小时上限，会分几轮断点续编（进度存在 Actions 缓存里，
本仓库没有并发组、不会被顶掉）。反复跑同一个 workflow 直到它发布出资产为止。
