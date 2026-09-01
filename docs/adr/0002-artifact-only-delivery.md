# 交付方式为 artifact-only，不发布 apt 仓库

本 fork 的唯一用途是构建 APISIX 的 Debian 12 deb 包供自行安装。决定：产物只作为 GitHub Actions artifact 下载，删除上游 publish-deb.yml 中发布到 APISIX 官方 apt 仓库（腾讯云 COS + freight + GPG 签名）的全部步骤与相关脚本依赖。

原因：不需要公共 apt 仓库；保留它们需要维护 COS secrets 与 GPG key，纯属负担。若未来需要自有 apt 仓库，可基于 git 历史恢复 freight 流程并指向自己的存储。

后果：发布链路的 secrets（VAR_COS_BUCKET_*、TENCENT_COS_SECRET*、RPM_GPG_*）不再需要；deb 未签名，本机安装需 `--allow-unauthenticated` 或 `dpkg -i`。
