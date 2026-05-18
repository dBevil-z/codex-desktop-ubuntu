# codex-desktop-ubuntu

这是一个面向 Ubuntu / Debian 系 Linux 发行版的非官方 Codex Desktop 打包发布仓库。

这个仓库并不是从 Codex Desktop 的原始源码直接编译应用，而是将官方发布的 macOS `Codex.dmg` 重新适配为 Linux 可运行的 Electron 应用，再进一步封装为适用于 Ubuntu / Debian 系统的 `.deb` 安装包。

## 当前定位

- 目标平台：Ubuntu / Debian / Pop!_OS / Linux Mint
- 安装包格式：`.deb`
- 分发方式：GitHub Releases 附件
- 仓库作用：保存说明文档、版本信息、校验值，以及发布二进制安装包

`.deb` 文件本身不适合直接提交进 git 历史。真实安装包通常有数百 MB，超过 GitHub 普通仓库文件的常规限制，所以正确做法是放到 GitHub Release 附件里。

## 依赖的上游来源

这套打包流程依赖两个上游来源：

1. OpenAI 官方发布的 Codex Desktop macOS 安装包
   - 上游格式：`Codex.dmg`
   - 作用：提供原始 Electron 应用资源和应用包内容
2. Linux 适配与打包项目
   - 仓库地址：[ilysenko/codex-desktop-linux](https://github.com/ilysenko/codex-desktop-linux)
   - 作用：负责提取 DMG、修补 Linux 兼容性、重建原生模块、下载 Linux Electron 运行时并打包

这个仓库本身尽量保持轻量，重点是对 Ubuntu / Debian 用户提供一个清晰、可追溯的发布入口。

## 原理说明

这不是一个“任何 DMG 都能直接转成 DEB”的通用流程。之所以能对 Codex Desktop 生效，是因为它本质上是 Electron 应用，大部分逻辑位于跨平台的前端资源和 `app.asar` 中。

整体流程大致如下：

1. 下载或复用最新的 `Codex.dmg`
2. 从 DMG 中提取 `Codex.app`
3. 解包 `app.asar`
4. 对打包后的 JavaScript 做 Linux 兼容性补丁
5. 为 Linux 重新编译 `better-sqlite3`、`node-pty` 等原生 Node 模块
6. 下载匹配版本的 Linux Electron 运行时
7. 重新组装 Linux 应用目录
8. 生成原生 Debian 安装包

所以更准确的描述是：围绕上游应用资源重新组装 Linux 运行时，而不是把 DMG 直接格式转换成 DEB。

## 为什么单独做一个发布仓库

这个仓库的目的主要是让 Ubuntu / Debian 用户能方便地拿到：

- 可直接安装的 `.deb`
- 对安装包来源和生成方式的解释
- 对应的上游适配仓库地址
- 版本、构建时间、校验值等发布信息

真正的 Linux 适配逻辑仍然以上游 `ilysenko/codex-desktop-linux` 为准。如果后续官方 `Codex.dmg` 内部结构变化，需要修补丁或者更新打包流程，也应该优先看那个仓库。

## 构建来源

- Linux 适配上游仓库：[ilysenko/codex-desktop-linux](https://github.com/ilysenko/codex-desktop-linux)
- 主入口脚本：`install.sh`
- Debian 打包脚本：`scripts/build-deb.sh`

如果你想自行复现构建，可以使用：

```bash
git clone https://github.com/ilysenko/codex-desktop-linux.git
cd codex-desktop-linux
./install.sh --fresh
make deb
```

如果你手头已经有本地 `Codex.dmg`，调试时更建议直接复用：

```bash
./install.sh /absolute/path/to/Codex.dmg
make deb
```

## 当前发布版本

本节随每次新包发布更新。

- 上游 Linux 适配仓库提交：`39da63072705e5f9330e6a5509626b320e7b8a1c`（短 SHA：`39da630`）
- 上游 `Codex.dmg` 最后修改时间：`2026-05-16 02:26:28 UTC`
- Electron 运行时版本：`42.0.1`
- 安装包文件名：`codex-desktop_2026.05.18.072740_amd64.deb`
- 安装包大小：`242M`（`253,224,686` bytes）
- SHA256：`a643958136e74f769003912aabfb7ccf5d8693cfb32b958ed091dfd620127328`

对应的 `.deb` 文件应作为 GitHub Release 附件发布。

## 建议的 Release 结构

每次发布建议至少上传：

1. `codex-desktop_2026.05.18.072740_amd64.deb`
2. `SHA256SUMS`

用户可使用以下命令安装：

```bash
sudo apt install ./codex-desktop_2026.05.18.072740_amd64.deb
```

## 注意事项

- 这是非官方 Linux 打包，不是 OpenAI 官方发布的 Linux 版本。
- Linux 包内的自动更新本质上是本机本地重建，不是直接从 OpenAI 下载现成 Linux 包。
- 只要官方 `Codex.dmg` 的内部 bundle 结构发生变化，未来仍有可能需要更新补丁。
- 如果出现新 DMG 导致打包失败，通常应先更新 `ilysenko/codex-desktop-linux` 上游仓库，再重新构建。

## 许可与归属

请保留上游归属说明：

- Codex Desktop 应用和品牌归 OpenAI 所有。
- Linux 适配与打包逻辑来自 [ilysenko/codex-desktop-linux](https://github.com/ilysenko/codex-desktop-linux)。

这个仓库应明确说明：它分发的是基于上游组件重新生成的非官方 Linux 安装包。
