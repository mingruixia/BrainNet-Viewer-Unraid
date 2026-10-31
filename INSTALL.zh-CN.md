# BrainNet Viewer LAN：Unraid 安装说明

本模板使用已发布的 `mingruixia/brainnet-viewer:2.0.0`，通过 HTTPS 在局域网内访问。
每个登录后的浏览器会话使用独立进程；同一浏览器配置下的标签页共用会话。
团队共用访问密码，不提供实名账户或按用户分级授权。

## 1. 准备证书与密码

在 Unraid 启用 Docker，确定固定 IP 和可用端口，默认 1031。限制为可信局域网访问。
向管理员取得包含服务器 IP 的 TLS 证书及对应私钥，客户端需信任签发 CA。
不要跳过浏览器证书警告，也不要直接复用或暴露 Unraid 管理界面的私钥。

在私有目录 `/mnt/user/appdata/brainnet-viewer-lan/secrets` 放置：

| 文件 | 内容 |
| --- | --- |
| `server.crt` | PEM 证书，必要时包含中间证书链 |
| `server.key` | 对应私钥 |
| `access_password.txt` | 本机生成的随机访问密码，一行，16–256 个 UTF-8 字节 |

该目录不要经 SMB/NFS 对外共享。容器以 UID 10001 运行。仅对上述新建专用目录，
管理员可以执行：

```sh
bnvSecrets=/mnt/user/appdata/brainnet-viewer-lan/secrets
chown 10001:10001 "$bnvSecrets" "$bnvSecrets/server.crt" "$bnvSecrets/server.key" "$bnvSecrets/access_password.txt"
chmod 700 "$bnvSecrets"
chmod 400 "$bnvSecrets/server.crt" "$bnvSecrets/server.key" "$bnvSecrets/access_password.txt"
```

不要将这些命令用于其他共享目录。模板只读挂载上述目录，不在安装表单中填写密码或私钥，
不支持通过 PUID/PGID 环境变量更改用户。

## 2. 安装应用

应用尚未获 Community Applications 收录。收录前，可将仓库的
`templates/brainnet-viewer-lan.xml` 复制到 Unraid 的
`/boot/config/plugins/dockerMan/templates-user/my-brainnet-viewer-lan.xml`，
再从 **Docker → Add Container → Template** 选择。收录后可在 **Apps** 搜索
`brainnet-viewer-lan`。

- **Network**：保持 `bridge`。
- **HTTPS port**：默认 `1031`；冲突时更换主机端口，容器端口仍为 1031。
- **Public HTTPS origin**：填写完整地址，例如 `https://192.168.1.10:1031`，不含路径。
- **Private server files**：选择上一步的私有目录。
- **Rendering backend**：保持 `auto`；没有可用 GPU 时使用 CPU。

地址必须与浏览器访问地址、证书和主机端口一致。如使用域名，直接打开该域名的完整 HTTPS
地址，并相应修改 WebUI 快捷入口。暂不直接叠加 Tailscale 或 TLS 终止反向代理。
点击 **Apply**，启动后打开 **WebUI**，输入团队密码。新会话图形初始化可能需要最多 90 秒。
健康状态仅表示入口服务已就绪，不代表已经完成图形绘制。

## 3. 资源与 GPU

默认最多 4 个会话，空闲 30 分钟回收，每个工作进程 2 个科学计算线程。全部会话共用
1 CPU、16 GiB 内存上限、512 进程及 2 GiB 临时存储上限；内存上限不是预先占用量。
可在高级设置 **Extra Parameters** 调整 `--cpus`、`--memory` 和 `/tmp` 大小，
不得超过服务器实际资源。小型设备应减少会话数量。线程数不等于 CPU 配额。

首次模板不请求 GPU，因此没有 GPU 也能启动。需要 NVIDIA 加速时，先准备匹配的
Unraid NVIDIA 驱动和容器运行时，再在 **Extra Parameters** 添加 `--gpus all`，
增加环境变量 `NVIDIA_DRIVER_CAPABILITIES=graphics,utility,compute`，渲染后端保持 `auto`。
工作进程会探测实际绘制能力。若 Docker 在分配 GPU 时直接拒绝启动，移除 `--gpus all`
恢复 CPU 启动；应用内回退无法处理容器启动之前的分配失败。
不要启用 privileged，也不需要挂载 Docker socket。

## 4. 保存、退出与升级

Add Files 上传客户端文件至当前会话。Save Scene 下载可迁移 ZIP；Export Figure 分别下载
图片和 JSON 配套说明。临时上传文件不会长期保存在服务器，应在退出、超时、停止或升级前保存 Scene。
立即结束会话可访问 `https://服务器IP:端口/_bnv/session` 并选择 **Sign out and end session**。

模板固定使用 2.0.0，不自动追踪 latest。升级前保存 Scene、阅读发行说明，保留旧镜像标签以便回退。
重启会结束所有未保存会话。

2026-10-31 的 Docker 专项已通过 HTTPS 就绪、登录页、两个独立安全会话 Cookie、
HTTPS 健康检查、非 root 运行、只读密钥挂载及资源限制检查；未重跑工作进程绘制。
目前尚未完成 Unraid 实机安装、安装表单和 NVIDIA 透传验证；普通 Docker 验证不能代替这些结果。
原生 ARM64 容器图形验证也仍有缺口。问题请通过
[BrainNet Viewer GitHub](https://github.com/mingruixia/BrainNet-Viewer/issues)反馈，勿上传密码、私钥或受试者信息。
