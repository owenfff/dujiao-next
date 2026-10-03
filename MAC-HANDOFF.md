# 在 Mac 上继续使用 Dujiao-Next

本仓库是官方开源项目的 fork。`my-deployment` 分支以 v1.4.8 为基础，与当前腾讯云安装版本一致。原项目许可证见 LICENSE。

## 获取源码

在 Mac 终端执行（首次使用 Git 时，macOS 可能提示安装命令行工具）：

```bash
git clone --branch my-deployment https://github.com/owenfff/dujiao-next.git
cd dujiao-next
```

也可以在 Codex 中打开这个目录继续开发。源码不包含服务器数据库、真实配置、密码、商品库存或上传图片。

## 直接管理已有商城

商城实际运行在腾讯云 Ubuntu 24.04 服务器上，不需要在 Mac 重新安装程序。Windows 的 SSH 隧道需要在 Mac 上重新建立。

```bash
ssh -N -o ExitOnForwardFailure=yes -L 127.0.0.1:8088:127.0.0.1:8088 ubuntu@你的服务器公网IP
```

把“你的服务器公网IP”替换为腾讯云控制台中的地址。输入 SSH 密码后，保持此终端打开；连接正常时没有输出。

浏览器打开 http://127.0.0.1:8088/ 。在另一个终端登录服务器查看后台路径和初始账号：

```bash
ssh ubuntu@你的服务器公网IP
sudo cat /opt/dujiao-next/initial-login.txt
```

将后台路径接在 http://127.0.0.1:8088 后即可打开后台。若已经修改密码，使用新密码。不要将此文件或 config.yml 上传 GitHub。

## 当前部署状态

- 程序：v1.4.8，目录 `/opt/dujiao-next`。
- 服务：`dujiao-next.service`，开机启动，以专用用户运行。
- 监听：`127.0.0.1:8088`；SQLite 数据库在 `db/dujiao.db`。
- systemd 覆盖配置在 `/etc/systemd/system/dujiao-next.service.d/mode.conf`，启动参数 `-mode api`。
- Redis、任务队列及 SMTP 暂时关闭，尚未完成正式域名、HTTPS 和支付接入。
- 之前健康检查成功，后台已能登录；当前实时状态应再次检查。

服务器上的检查命令：

```bash
sudo systemctl status dujiao-next --no-pager
curl -fsS http://127.0.0.1:8088/health
sudo tail -n 50 /opt/dujiao-next/logs/app.log
```

## 如需在 Mac 本地开发

参照 README 的开发步骤安装对应版本的 Go、Node.js 和 pnpm。复制 `config.yml.example` 为 `config.yml`，生成三个不同的随机密钥。不要直接使用示例占位密钥。

如果没有本地 Redis，将 redis.enabled、queue.enabled 设为 false，并使用 `go run ./cmd/server -mode api`。不要沿用默认 all 模式，否则会因 queue disabled 退出。未配置邮件时关闭 email.enabled。

分别在 `frontend/user`、`frontend/admin` 执行 `pnpm install` 和 `pnpm run dev`。默认后端端口 8080，开发前台端口 5173，开发后台端口 5174。具体版本要求以本分支 go.mod 和各 package.json 为准。

## 数据备份

本 GitHub 仓库仅保存源码和操作说明，不是运行数据备份。迁移服务器前，停止商城服务并备份数据库目录、uploads 和真实 config.yml（包括 app.secret_key），妥善保存在私密位置，再恢复服务。

