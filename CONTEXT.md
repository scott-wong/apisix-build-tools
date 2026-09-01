# APISIX Debian Build (Fork)

本仓库是 apache/apisix-build-tools 的 fork，唯一用途：从源码构建 APISIX 的 Debian 12 (bookworm) 安装包（deb），运行时基于最新的 OpenResty master 源码。产物只进 GitHub Actions artifact，不发布任何软件仓库。

## Language

### 包 (Packages)

**apisix deb**:
最终交付物。包含 APISIX Lua 应用、CLI 与 systemd 服务，并**内嵌**完整 `/usr/local/openresty` 运行时树。不声明对 openresty 包的依赖。
_Avoid_: 与上游概念混用的 "apisix 包"（上游还指 RPM）、dashboard 包

**apisix-runtime deb**:
apisix deb 的构建中间产物，包含编好的 OpenResty 及其 Nginx 模块。不单独交付；不追求成为可复用的运行时包。
_Avoid_: 把它当作与 apisix deb 同级的发布物

**runtime 内嵌 (bundled runtime)**:
apisix deb 内 `/usr/local/openresty` 的副本，由 runtime deb 阶段编译并拷贝而来。

### 版本 (Versions)

**APISIX 标签 (APISIX tag)**:
每次构建的版本锚点。打 `apisix/<ver>` 标签即以 apache/apisix 的同名 tag 为源码快照构建。语义与上游一致：标签号 = APISIX 版本号，可复现。

**OpenResty master 滚动源码**:
runtime 阶段构建时从 openresty/openresty 的 master 分支拉取源码（当前为 1.31.x mainline），**不钉版本**。同一 APISIX 标签在不同日期构建，内嵌 runtime 可能不同；以构建日志记录的 OpenResty commit SHA 为准。
_Avoid_: "最新 OpenResty"（歧义：可能指 release）

**runtime 构建时间戳 (runtime build stamp)**:
区分同标签、不同日期构建产物的标识（构建日期 + OpenResty commit SHA），记录在构建日志与 deb 描述中。

### 流水线 (Pipeline)

**两段构建 (two-stage build)**:
同一工作流内先编 runtime deb，再以其为基础编 apisix deb。两段产物都在同一次 run 中产出。

**冒烟测试 (smoke test)**:
在 `debian:bookworm` 容器中安装产出的 deb，执行 `openresty -V` 与 `apisix version` 验证可用。

**artifact-only 交付**:
构建产物仅作为 GitHub Actions artifact 下载，无 apt 仓库、无 GPG 签名、无 COS 上传。

### 构建失败 (Build failures)

**修到编过 (fix forward)**:
OpenResty master 破坏兼容时（补丁打不上、编译失败、冒烟不过），修改补丁/脚本直到再次构建成功；不自动回退到旧版源码，不静默降级。

## Decisions

- ADR-0001: runtime 基于最新的 OpenResty master 源码构建（不钉版本，接受兼容性维护成本）
- ADR-0002: 交付方式为 artifact-only，不发布 apt 仓库
