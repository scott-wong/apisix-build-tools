# 升级到 OpenResty 1.31.1.1：vendor 两个 patch.sh 白名单改动

ADR-0003 曾因两个 API7 补丁模块的 `patch.sh` 不识别 1.31.x 而把默认版本回退到 1.29.2.5。随后社区提交了三个 PR（api7/ngx_multi_upstream_module#21、api7/apisix-nginx-module#125、api7/apisix-build-tools#483），作者完成了全链路验证：OpenResty 1.31.1.1 无需任何 C 源码改动——唯一障碍是两个 `patch.sh` 的版本白名单。1.29.2 补丁集可直接用于 1.31.1（nginx 核心 1.29.2→1.31.1 未触碰被补丁的位置；bundle 组件只前进了一档 rc：lua-resty-core 0.1.34rc2→rc3、ngx_lua 0.10.31rc2→rc5、ngx_stream_lua 0.0.19rc3→rc4）。唯一已知非致命项：ngx_lua 0.10.31rc5 上 `ngx_lua-shared_shdict.patch` 有一个 hunk 拒绝（PR125 的 patch.sh 已改为 warn-and-continue；APISIX 不使用 ngx_lua 内置 shdict）。

决定：采用方案 A（vendor）——把两个 PR 对 `patch.sh` 的改动以覆盖文件形式收进本仓库 `patches/{ngx_multi_upstream_module,apisix-nginx-module}/patch.sh`，`build-apisix-runtime.sh` 在克隆模块后覆盖原脚本，默认 `OPENRESTY_VERSION` 升到 1.31.1.1。不 fork 模块仓库（改动只有 patch.sh，fork 维护成本不成比例）。

回退条件：两个 PR 在上游合入并有正式 release tag 后，删除 `patches/` 覆盖与 build 脚本中的覆盖逻辑即可回到纯上游。

后果：runtime 基于 OpenResty 1.31.1.1（nginx 1.31.1，含 HTTP/3 与安全修复）；shared_shdict 一个 hunk 拒绝会在构建日志里以 WARNING 形式出现，属预期；若未来把模块版本升级（apisix_nginx_module_ver 等），需同步复核 vendor 的 patch.sh 与上游 main 的差异。
