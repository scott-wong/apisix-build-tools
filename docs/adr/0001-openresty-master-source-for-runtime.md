# 基于 OpenResty master 源码构建 runtime

我们要为 Debian 12 构建自带最新运行时的 APISIX deb 包。决定：runtime 阶段不从 openresty.org 下载钉死的 release tarball（上游钉 1.29.2.4），而是从 openresty/openresty master 分支拉源码（`util/mirror-tarballs` 流程）构建。

原因：目标是让包内运行时始终吃到最新的 nginx/OpenResty 修复与特性，而不是跟随上游缓慢的版本钉定节奏。master 当前位于 1.31.x mainline（nginx 1.31），apisix-nginx-module 等 API7 侧补丁模块官方只验证到 1.29.x，因此 master 源码随时可能补丁打不上、编译失败或冒烟不过。

考虑过的替代方案：跟随最新稳定线（1.29.x，兼容风险低）；可配置、默认稳定线。均被否决——用户明确要 master 源码的时效性。

后果：构建失败时采取 fix-forward 策略（改补丁/脚本直到编过并冒烟通过），不自动回退旧版、不静默降级。同一 APISIX 标签在不同日期构建的产物可能内嵌不同运行时；每次构建记录 OpenResty commit SHA 以便追溯。
