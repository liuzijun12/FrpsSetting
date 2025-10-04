# FRP 服务器部署文档

这是一个基于 Docker Compose 的 frps（FRP 服务端）部署方案，配合 Caddy 作为反向代理。

## 📋 项目简介

本项目使用 Docker Compose 部署 frps 服务器，用于实现内网穿透功能。通过 Caddy 提供反向代理和自动 HTTPS 证书管理。

## 🗂️ 项目结构

```
.
├── docker-compose.yml      # Docker Compose 配置文件
├── frps.toml.example       # frps 配置文件示例
├── Caddyfile.example       # Caddy 配置文件示例
└── README.md               # 本文档
```

## 🚀 快速开始

### 1. 准备配置文件

#### 创建 frps 配置文件

复制示例配置并根据需要修改：

```bash
cp frps.toml.example frps.toml
```

编辑 `frps.toml` 文件，基本配置示例：

```toml
bindPort = 7000

# 可选：启用仪表盘
# webServer.addr = "0.0.0.0"
# webServer.port = 7500
# webServer.user = "admin"
# webServer.password = "your_password"

# 可选：启用身份验证
# auth.method = "token"
# auth.token = "your_secret_token"
```

#### 创建 Caddyfile 配置（可选）

如果需要通过域名访问 frps 仪表盘，复制并修改 Caddyfile：

```bash
cp Caddyfile.example Caddyfile
```

编辑 `Caddyfile` 文件：

```
dashboard.yourdomain.com {
    reverse_proxy frps:7500
}
```

将 `dashboard.yourdomain.com` 替换为你的实际域名。

### 2. 启动服务

```bash
docker-compose up -d
```

### 3. 查看日志

```bash
# 查看所有服务日志
docker-compose logs -f

# 仅查看 frps 日志
docker-compose logs -f frps

# 仅查看 Caddy 日志
docker-compose logs -f caddy
```

### 4. 停止服务

```bash
docker-compose down
```

## 🔧 配置说明

### docker-compose.yml

该文件定义了两个服务：

#### frps 服务
- **镜像**: `fatedier/frps:v0.63.0`
- **端口**: `7000` - frps 主服务端口
- **配置文件**: `./frps.toml` 挂载到容器的 `/etc/frp/frps.toml`

#### Caddy 服务
- **镜像**: `caddy:latest`
- **端口**: `80`, `443` - HTTP 和 HTTPS 端口
- **配置文件**: `./Caddyfile` 挂载到容器的 `/etc/caddy/Caddyfile`
- **功能**: 反向代理、自动 HTTPS 证书

### 网络配置

两个服务通过 `frp_network` 桥接网络连接，可以通过服务名相互访问。

## 📱 客户端配置示例

在需要内网穿透的机器上配置 frpc（FRP 客户端）：

### frpc.toml 示例

```toml
serverAddr = "your_server_ip"
serverPort = 7000

# 如果服务端启用了 token 认证
# auth.method = "token"
# auth.token = "your_secret_token"

# 示例：穿透 SSH 服务
[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6000

# 示例：穿透 Web 服务
[[proxies]]
name = "web"
type = "http"
localIP = "127.0.0.1"
localPort = 8080
customDomains = ["web.yourdomain.com"]
```

## 🔐 安全建议

1. **启用身份验证**: 在 `frps.toml` 中配置 `auth.token`
2. **修改默认端口**: 根据需要修改 `bindPort`
3. **限制访问**: 使用防火墙规则限制访问来源
4. **使用强密码**: 如果启用了仪表盘，设置强密码
5. **定期更新**: 保持 frps 版本更新

## 🛠️ 常用命令

```bash
# 重启服务
docker-compose restart

# 更新镜像
docker-compose pull
docker-compose up -d

# 查看运行状态
docker-compose ps

# 进入容器
docker-compose exec frps sh
```

## 📊 端口说明

| 端口 | 服务 | 说明 |
|------|------|------|
| 7000 | frps | frp 主服务端口（客户端连接） |
| 7500 | frps | Web 仪表盘（可选） |
| 80   | Caddy | HTTP 端口 |
| 443  | Caddy | HTTPS 端口 |

## ❓ 故障排查

### 服务无法启动

```bash
# 检查配置文件是否存在
ls -la frps.toml

# 检查配置文件语法
docker-compose config

# 查看详细日志
docker-compose logs frps
```

### 客户端无法连接

1. 检查服务器防火墙是否开放 7000 端口
2. 检查 frps 服务是否正常运行：`docker-compose ps`
3. 确认客户端配置的服务器地址和端口正确
4. 检查 token 是否匹配（如果启用了认证）

### 查看 frps 版本

```bash
docker-compose exec frps frps -v
```

## 📚 更多资源

- [FRP 官方文档](https://github.com/fatedier/frp)
- [FRP 中文文档](https://gofrp.org/zh-cn/)
- [Caddy 官方文档](https://caddyserver.com/docs/)

## 📝 更新日志

- 使用 frps v0.63.0
- 使用 Caddy 最新版本
- 支持 Docker Compose v3.8

## 📄 许可证

本项目配置文件基于 MIT 许可证开源。

---

**注意**: 请确保在生产环境中修改所有默认密码和配置！

