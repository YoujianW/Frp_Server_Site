# 服务器网站搭建教程

基于 nginx 、樱花 frp 服务器搭建的服务器门户网站（纯静态站点）。

## 快速开始

### 本地开发网页（推荐 Python）

```bash
# 在项目根目录启动
cd mc-server-site/
python3 -m http.server 8080 --bind 127.0.0.1

# 访问 http://127.0.0.1:8080
```

> 不需要安装任何依赖，Python 自带 HTTP 服务器即可工作。
> 也可使用 VS Code Live Server 等插件。

### 生产部署到云服务器上（推荐 Nginx）

适用于云服务器正式上线，支持高性能并发、gzip 压缩、缓存控制。

```nginx
# /etc/nginx/sites-available/mcserver
server {
    listen 80;
    server_name mc.kkkttt1234.top;

    root /home/你的用户名/web/minecraft/mc-server-site;
    index index.html;

    gzip on;
    gzip_types text/html text/css application/javascript application/json;
    gzip_min_length 1024;

    location ~* \.(jpg|jpeg|png|gif|ico|svg|webp)$ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }

    location ~* \.(css|js)$ {
        expires 1d;
        add_header Cache-Control "public, immutable";
    }

    location ~* \.(json|md)$ {
        expires -1;
        add_header Cache-Control "no-cache";
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

```bash
# 启用站点
sudo ln -sf /etc/nginx/sites-available/mcserver /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl restart nginx
```

### 生产部署（Python 轻量方案）（但是）

不想装 Nginx 时可临时用 Python 跑：

```bash
cd ~/web/minecraft/mc-server-site

# 后台运行（关终端也不会停）
nohup python3 -m http.server 8080 --bind 0.0.0.0 > server.log 2>&1 &

# 查看运行状态
ps aux | grep http.server

# 停止服务
kill $(pgrep -f "http.server 8080")
```

### 通过 frp 内网穿透公网访问

如果云服务器没有公网 IP，可通过 frp（如樱花 frp）将域名流量转发到本地：

```
用户 → 你的域名 → frp 服务器 → 云服务器 frpc → localhost:80 → Nginx
```

frpc 配置示例（`frpc.toml`）：

```toml
[[proxies]]
name = "mcserver"
type = "http"
localPort = 80
localIp = "127.0.0.1"
customDomains = ["mc.summer
.top"]
```

### git 同步更新

```bash
# 本地修改后推送
git add -A && git commit -m "更新说明" && git push

# 服务器拉取
ssh 你的服务器
cd ~/web/server/
git pull
# Nginx/Python 自动服务新文件，无需重启
```

> 不需要安装任何依赖，直接使用任意 HTTP 服务器托管静态文件即可。
> 推荐使用 Python 内置 HTTP 服务器、Nginx、或 VS Code Live Server 插件。
