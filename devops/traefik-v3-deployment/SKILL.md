---
name: traefik-v3-deployment
description: 工业级 Traefik v3 部署与路由配置指南 (含 Docker Compose + Let's Encrypt + 避坑指南)
category: devops
---

# Traefik v3 部署与路由指南

Traefik 是一个现代化的 HTTP 反向代理和负载均衡器，原生支持 Docker 容器的动态发现和 Let's Encrypt SSL 证书自动管理。

## 🚀 快速部署 (Docker Compose)

### 1. 准备工作
创建配置目录并初始化证书文件（权限必须为 600，否则 Traefik 会拒绝启动）：
```bash
mkdir -p /opt/traefik
touch /opt/traefik/acme.json
chmod 600 /opt/traefik/acme.json
```

### 2. 编写 `docker-compose.yml`
推荐使用以下配置，包含 **HTTP $\rightarrow$ HTTPS 强制跳转** 和 **ACME 自动证书**：

```yaml
services:
  traefik:
    image: traefik:v3.1
    container_name: traefik
    restart: always
    ports:
      - '80:80'
      - '443:443'
      - '8080:8080' # Dashboard 端口
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./acme.json:/acme.json
    command:
      - '--api.insecure=true' # 开启 Dashboard
      - '--providers.docker=true'
      - '--providers.docker.exposedbydefault=false'
      - '--entrypoints.web.address=:80'
      - '--entrypoints.websecure.address=:443'
      - '--certificatesresolvers.myresolver.acme.tlschallenge=true'
      - '--certificatesresolvers.myresolver.acme.email=your-email@example.com'
      - '--certificatesresolvers.myresolver.acme.storage=/acme.json'
      - '--entrypoints.web.http.redirections.entryPoint.to=websecure'
      - '--entrypoints.web.http.redirections.entryPoint.scheme=https'
    networks:
      - traefik-public

networks:
  traefik-public:
    external: true # 建议创建外部网络，方便其他项目加入
```
*注：启动前请先运行 `docker network create traefik-public`*

## 🛠️ 容器路由配置 (Labels)

要在 Traefik 中暴露一个服务，只需在目标容器的 `labels` 中添加以下内容：

| 标签 | 作用 | 示例 |
| :--- | :--- | :--- |
| `traefik.enable` | 启用该容器的路由 | `true` |
| `traefik.http.routers.<name>.rule` | 定义访问域名 | `Host(\"app.example.com\")` |
| `traefik.http.routers.<name>.entrypoints` | 指定监听端口 | `web,websecure` |
| `traefik.http.routers.<name>.tls.certresolver` | 指定 SSL 解析器 | `myresolver` |
| `traefik.http.services.<name>.loadbalancer.server.port` | 指定容器内部端口 | `80` |

**完整示例：**
```yaml
labels:
  - \"traefik.enable=true\"
  - \"traefik.http.routers.myapp.rule=Host(\\\"app.example.com\\\")\"
  - \"traefik.http.routers.myapp.entrypoints=websecure\"
  - \"traefik.http.routers.myapp.tls.certresolver=myresolver\"
  - \"traefik.http.services.myapp.loadbalancer.server.port=80\"
```

## ⚠️ 核心避坑指南 (实战血泪史)

### 1. 网络隔离 (Gateway Timeout)
**现象**：访问域名返回 `504 Gateway Timeout`。
**根因**：Traefik 容器与目标容器不在同一个 Docker 网络中。
**解决**：确保目标容器加入了 Traefik 所在的网络（例如 `traefik-public`）。
```bash
docker network connect traefik-public <container_id>
```

### 2. Dashboard 404 陷阱
**现象**：访问 `https://traefik.example.com` 看到 `404 page not found`。
**根因**：Traefik Dashboard 的默认路径是 `/dashboard/` (必须带末尾斜杠)，根目录没有内容。
**解决**：配置重定向中间件将 `/` 跳转到 `/dashboard/`。

### 3. 路由语法错误 (Illegal Rune Literal)
**现象**：日志出现 `illegal rune literal` 错误，路由不生效。
**根因**：在 `Host('...')` 中使用了单引号，在某些 Shell/YAML 环境下被误认为是字符字面量。
**解决**：统一使用双引号 $\rightarrow$ `Host(\"example.com\")`。

### 4. SSL 证书申请失败
**现象**：访问 HTTPS 极慢或出现证书错误。
**根因**：`tlschallenge` 要求 443 端口必须直接暴露给公网且无防火墙拦截。
**验证**：检查 `docker logs traefik` 查看 ACME 错误日志。

## ✅ 验证工作流
1. **检查运行状态**：`docker ps` 确保 Traefik 和目标容器均处于 `Up` 状态。
2. **内部路由测试**（排除外部网络干扰）：
   ```bash
   # 在宿主机模拟请求
   curl -k -H 'Host: app.example.com' https://localhost
   ```
3. **检查跳转**：`curl -I http://app.example.com` 确认返回 `308 Permanent Redirect`。
4. **检查日志**：`docker logs traefik` 确认无 `ERR` 级别日志。
