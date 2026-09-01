# 默认回到 OpenResty 1.29 稳定线，master 源码模式保留

我们要 runtime 基于最新 OpenResty 源码（ADR-0001 选择了 master 分支）。但首两轮 CI 失败暴露了硬约束：APISIX 生态的两个补丁模块（`apisix-nginx-module` 1.19.10、`ngx_multi_upstream_module` 1.3.4）的 patch 集只覆盖到 nginx 1.29.2，`patch.sh` 按 OpenResty 目录名前缀选补丁，遇到 1.31.x 直接报 "can't detect OpenResty version" 退出；两个仓库均无 1.31 相关的 issue/PR，没有官方时间表。

决定：`OPENRESTY_SOURCE` 默认值回退为 `release`，钉 1.29 稳定线最新（1.29.2.5，1.29.2 前缀与现有补丁兼容）；master 模式的机制完整保留，可通过 `openresty_source=master` 或改一行默认值随时切回，等 API7 官方发布 1.31 补丁后一键升级。

考虑过的替代方案：把 1.29.2 补丁集 vendor 进本仓库并自行移植到 1.31.4——成本是持续的补丁维护负担，且 nginx 1.31 对 upstream 行为的变更未必能靠改 hunk 解决，收益（吃到 master 修复）与其不确定性不匹配，被否决；等待上游发布 1.31 补丁——即本决定。

后果：deb 内嵌的 runtime 基于 1.29.2.5 而非 OpenResty master；每次构建仍记录来源 stamp（`openresty-commit` 文件，release 模式下内容为 `release-1.29.2.5`，master 模式下为真实 commit SHA）。
