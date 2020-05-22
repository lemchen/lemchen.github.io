# Caddy自动申请HTTPS

## 用Docker部署Caddy

Docker 仓库里的2.0版本的caddy没有包含使用 dns-01 方式申请证书的 tls dns cloudflare 模块，需要自己使用 官方的构建镜像  [caddy:builder ](https://github.com/caddyserver/caddy-docker/blob/d9baf11b7abb9343891bb9c28f8cc7137ae94b68/builder/Dockerfile) 构建包含域名模块的 caddy docker 镜像。2.0版本的 DNS provider 模块地址 [https://github.com/caddy-dns](https://github.com/caddy-dns)

```bash
docker pull caddy:builder
```

构建镜像的Dockerfile文件如下，添加 [cloudflare](https://github.com/caddy-dns/cloudflare) 模块。如果使用国内域名解析服务请添加[lego-deprecated](https://github.com/caddy-dns/lego-deprecated) 模块

```yaml
FROM caddy:2.0.0-builder AS builder

RUN caddy-builder \
    github.com/caddy-dns/cloudflare
    github.com/caddy-dns/lego-deprecated #更多的dns provider

FROM caddy:2.0.0

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

国内使用docker 构建镜像因为网络问题，需要使用代理

```bash
export https_proxy="socks5://serverip:port"
```

### 构建镜像

```bash
docker build --network host --build-arg https_proxy -t quadword/caddy:1.0 .
```

caddy docker 镜像构建完成后就可以使用了

```bash
docker run -d --name caddy -p 80:80 -p 443:443 -p 2019:2019 quadword/caddy
```

### Caddyfile 配置

cloudflare tls 的 token 可以直接写在 caddyfile 里

```yaml
tls {
  dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  dns lego_deprecated dnspod #DNSPOD 的写法
}
```

在docker-compose.yml 文件里添加环境变量

```yaml
environment:
  DNSPOD_API_KEY: "id,token" # 值为 ID,TOKEN 注意这个值的格式
  DNSPOD_HTTP_TIMEOUT: 10000 #单位毫秒，10秒 避免timeout错误网络条件好可以不用设置
```







