# BrainNet Viewer LAN：Unraid安装说明

应用已收录至[Unraid Community Applications](https://ca.unraid.net/apps?q=brainnet)。默认无需证书、密码或提前填写服务器地址。

## 1. 安装并使用

1. 在Unraid的 **Apps** 搜索 **brainnet-viewer-lan** 或 **BrainNet Viewer**。
2. 点击 **Install**，端口保留 **1031**；如有冲突，换一个可用主机端口。
3. 点击 **Apply**，启动后打开 **WebUI**，访问 `http://Unraid-IP:1031`，直接进入应用。

首次会话初始化图形工作进程可能需要最多90秒。每个浏览器会话使用独立进程和工作目录，同一浏览器配置下的标签页共用会话。默认HTTP不加密且没有访问密码，适用于可信局域网，不要将该入口映射到公网。

模板使用 `mingruixia/brainnet-viewer:2.0.0-unraid.1`，只增加便捷局域网网关；原 `2.0.0` 和 `latest` 镜像不变。[源码和构建说明](https://github.com/mingruixia/BrainNet-Viewer/blob/maintenance/unraid-easy-lan-20261102/docs/web/zh-CN/UNRAID_LAN.md)。

如需手动安装，将 `templates/brainnet-viewer-lan.xml` 复制到 `/boot/config/plugins/dockerMan/templates-user/my-brainnet-viewer-lan.xml`，从 **Docker → Add Container → Template** 选择。

## 2. 可选HTTPS和密码

需要时才打开高级设置：

| 设置 | 默认 | 可选配置 |
|---|---|---|
| HTTP or HTTPS | http | https |
| Password access | off | on |
| Public origin | 留空 | 完整http(s)://主机:端口，不含路径 |
| Private server files | 留空，不挂载 | 保存下列私有文件的目录，只读挂载 |

启用HTTPS时，准备匹配服务器IP/域名的 `server.crt` 和 `server.key`，在Public origin填写准确的HTTPS地址，选择https，再挂载文件目录到 `/run/bnv-secrets`。客户端应信任证书签发者；相应把WebUI快捷地址改为同一HTTPS地址，不跳过证书警告。

启用密码时，准备 `access_password.txt`，选择Password access为on。密码为本机生成的一行随机字符，占16—256个UTF-8字节，没有通用默认密码。仅启用密码不会加密HTTP；需要加密时同时启用HTTPS。只需准备已启用选项对应的文件。

私有目录不经SMB/NFS共享，文件须可由UID **10001** 读取。例如新建专用目录 `/mnt/user/appdata/brainnet-viewer-lan/secrets`，放好所需文件后，仅对该目录设置权限：

```sh
bnvSecrets=/mnt/user/appdata/brainnet-viewer-lan/secrets
chown 10001:10001 "$bnvSecrets"
chmod 700 "$bnvSecrets"
for bnvFile in server.crt server.key access_password.txt; do
  if [ -f "$bnvSecrets/$bnvFile" ]; then
    chown 10001:10001 "$bnvSecrets/$bnvFile"
    chmod 400 "$bnvSecrets/$bnvFile"
  fi
done
```

模板只读挂载该目录，不在表单填写密码/私钥，不使用PUID/PGID。HTTP模式使用域名时也须明确填写Public origin；留空只自动接收私有/回环IP。

## 3. 资源与GPU

默认最多4会话、空闲30分钟回收，每worker两个科学计算线程。全部会话共用1 CPU、16 GiB内存、512进程及2 GiB临时存储上限；内存上限不是预分配。按硬件在Extra Parameters调整 `--cpus`、`--memory` 和 `/tmp`，小型设备减少会话数。线程数不等于CPU配额。

默认不分配GPU，auto探测到可用绘制设备时使用GPU，否则CPU。需要NVIDIA加速时，先配置匹配的Unraid NVIDIA驱动及容器运行时，再增加 `--gpus all` 和环境变量 `NVIDIA_DRIVER_CAPABILITIES=graphics,utility,compute`。Docker分配失败时，移除 `--gpus all` 才能恢复CPU启动；应用回退不能修复容器启动前的分配失败。不启用privileged或挂载Docker socket。

## 4. 保存、退出和升级

Add Files上传到当前会话；Save Scene下载可迁移ZIP；Export Figure分别下载图片和JSON。退出、超时、重启或升级前保存Scene，临时上传文件不会长期保存。

访问 `http://服务器IP:端口/_bnv/session`（已启用HTTPS时使用HTTPS）结束会话，其他会话继续运行。退出页不会立即重建新进程，返回应用或点击Start new session才重新开始。

旧安装会保留Unraid保存的HTTPS配置。仅更换镜像不会自动移除旧入口覆盖、Post Arguments或密钥挂载。希望采用新默认模式时，先保存Scene并停止旧容器，再从Apps使用新模板和可用名称/端口安装；保留旧配置和私有文件，确认新安装正常后再处理。

本次定向检查覆盖Docker/Linux默认HTTP会话、一次CPU曲面绘制/导出、浏览器进入及可选HTTPS/密码，不替代原生Unraid安装表单、NVIDIA透传或原生ARM64图形验收。问题请通过[GitHub](https://github.com/mingruixia/BrainNet-Viewer/issues)反馈，不上传密码、私钥或受试者信息。
