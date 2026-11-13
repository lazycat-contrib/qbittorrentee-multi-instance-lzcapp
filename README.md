# qbittorrentee-multi-instance-lzcapp

qBittorrent 社区增强版的懒猫应用，**固定多实例**，每位用户拥有独立的 qBittorrent 实例、配置、任务和下载目录。部署凭据来自各自实例的设置向导。

## 两个版本

| 版本 | 仓库 | 包名 | 实例模式 |
| --- | --- | --- | --- |
| 单实例 | `lazycat-contrib/qbittorrentee-lzcapp` | `cloud.lazycat.app.qbittorrentee` | `multi_instance: false` |
| 多实例（本仓库） | `lazycat-contrib/qbittorrentee-multi-instance-lzcapp` | `community.lazycat.app.qbittorrentee` | `multi_instance: true` |

两个版本的软件名称相同，通过软件描述、包名和仓库区分。实例模式在包中固定，不支持通过设置向导动态切换。不同包不会自动迁移配置、任务或下载文件；切换前请备份。

## 镜像与版本

使用 `dockers-x/Docker-qBittorrent-Enhanced-Edition` 重写后的 `czyt/qbittorrentee:<四段版本>-r1`。GitHub Action 将其转存到懒猫镜像仓库，并更新 Manifest；这仍是同一重写版镜像。

应用版本由 GitHub Action 根据镜像自动写入，不手动加修复版本号。例如 `5.2.4.10-r1` 对应应用版本 `5.2.410`。两个仓库沿用同一发布流程、镜像源和组织密钥。

## 设置向导

- 登录用户名和密码：默认 `admin / adminadmin`，每次启动时应用，无需查看终端日志。请通过部署参数改密，不要仅在 WebUI 中修改。
- 启用自动登录：默认开启，使用 `simple-inject-password` 填写并提交凭据；开启时必须先通过懒猫身份认证。关闭后保留手动登录入口。不添加文件选择器拦截。
- 下载目录名称：默认 `Downloads`，只允许单层目录名，不能使用 `.`、`..`、路径分隔符、冒号、美元符或换行。修改名称只会切换目录，不会移动旧文件。

配置文件保存 PBKDF2 密码哈希；部署参数和环境变量仍需保密。

## 存储与权限

- 配置：`/lzcapp/var/config:/config`。
- 下载：`/lzcapp/documents/{{ .S.DeployUID }}/<下载目录名称>:/downloads`，默认使用用户应用文稿下的 `Downloads`。用户标识由系统提供，不是 Linux UID，也不由向导填写。
- 配置保存在应用数据中；下载保存在用户应用文稿中，使用 `document.private` 权限。切换包名、目录名或实例模式不会自动迁移旧文件。
- 使用独立子域名 `qbittorrentee-multi`，避免与单实例包的 `qbittorrentee` 入口冲突。请在设备上用不同账号验证配置和下载的隔离效果。
- 使用 `RUN_AS_ROOT=true`，不设置 `user`、`run_as` 或 PUID/PGID，关闭两项自动 chown。保留现有权限声明，不靠扩大容器权限修复宿主机挂载。
- `error while creating mount source path` 表示宿主机 Docker 在创建挂载源目录时失败，发生在镜像入口脚本执行之前。此前多实例问题的根因是试图通过部署参数动态设置 `multi_instance`，不是 qBittorrent 内部 mkdir 或降权。

## 构建与自动发布

本地执行 `lzc-cli project build -o dist/community.lazycat.app.qbittorrentee-v5.2.410.lpk`。本地构建不会提交商店审核。

每天 23:00 UTC 检查带 `-r1` 后缀的镜像，也可手动触发 `.github/workflows/lazycat.yml`。复用工作流固定为 `ca-x/lazycat-github-action@v1.3.8`，官方和私有商店均沿用现有配置。

GitHub Secrets：官方商店使用 `LZC_API_TOKEN`；私有商店使用 `APPSTORE_URL`、`APPSTORE_TOKEN`，可选 `PRIVATE_STORE_GROUP_CODES`。在本仓库设置专属的 `APP_ID` Secret，指向 `community.lazycat.app.qbittorrentee` 的私有商店条目，以覆盖可能存在的单实例组织级 `APP_ID`。

发布工具会按软件名称回退查找条目，而两版名称相同，因此工作流先核对 `APP_ID` 对应的包名，再运行原有发布流程。缺少新包的条目 ID 或包名不匹配时停止发布，不会自动覆盖单实例条目；首次发布前需先准备好新包的私有商店条目。
