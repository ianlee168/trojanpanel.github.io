# Docker Compose方式安装

## 准备

1. 新建Caddy配置文件

文件路径：`/tpdata/caddy/config.json`

[Caddy配置举例](custom-installation.html#caddy配置举例)

2. 新建Nginx配置文件

文件路径：`/tpdata/nginx/default.conf`

[Nginx配置举例](custom-installation.html#nginx配置举例)

3. 上传**静态**伪装网站

文件夹路径：`/tpdata/caddy/srv/`

## 配置文件

```yml
version: '3'

services:
  trojan-panel-caddy:
    image: caddy:2.6.2
    container_name: trojan-panel-caddy
    restart: always
    network_mode: host
    volumes:
      - "/tpdata/caddy/config.json:/tpdata/caddy/config.json"
      - "/tpdata/caddy/cert/:/tpdata/caddy/cert/certificates/acme-v02.api.letsencrypt.org-directory/${domain}/"
      - "/tpdata/caddy/srv/:/tpdata/caddy/srv/"
      - "/tpdata/caddy/logs/:/tpdata/caddy/logs/"
    command: caddy run --config /tpdata/caddy/config.json

  trojan-panel-mariadb:
    image: mariadb:10.7.3
    container_name: trojan-panel-mariadb
    restart: always
    network_mode: host
    environment:
      MYSQL_DATABASE: trojan_panel_db
      MYSQL_ROOT_PASSWORD: "${mariadb_pas}"
      TZ: Asia/Shanghai
    command: --port=9507

  trojan-panel-redis:
    image: redis:6.2.7
    container_name: trojan-panel-redis
    restart: always
    network_mode: host
    command: redis-server --requirepass ${redis_pass} --port 6378

  trojan-panel:
    image: jonssonyan/trojan-panel
    container_name: trojan-panel
    restart: always
    network_mode: host
    volumes:
      - "/tpdata/caddy/srv/:/tpdata/trojan-panel/webfile/"
      - "/tpdata/trojan-panel/logs/:/tpdata/trojan-panel/logs/"
      - "/etc/localtime:/etc/localtime"
    environment:
      - "mariadb_ip=127.0.0.1"
      - "mariadb_port=9507"
      - "mariadb_user=root"
      - "mariadb_pas=${mariadb_pas}"
      - "redis_host=127.0.0.1"
      - "redis_port=6378"
      - "redis_pass=${redis_pass}"

  trojan-panel-ui:
    image: jonssonyan/trojan-panel-ui
    container_name: trojan-panel-ui
    restart: always
    network_mode: host
    volumes:
      - "/tpdata/nginx/default.conf:/etc/nginx/conf.d/default.conf"
      - "/tpdata/caddy/cert/:/tpdata/caddy/cert/"

  trojan-panel-core:
    image: jonssonyan/trojan-panel-core
    container_name: trojan-panel-core
    restart: always
    network_mode: host
    volumes:
      - "/tpdata/trojan-panel-core/bin/xray/config:/tpdata/trojan-panel-core/bin/xray/config"
      - "/tpdata/trojan-panel-core/bin/trojango/config:/tpdata/trojan-panel-core/bin/trojango/config"
      - "/tpdata/trojan-panel-core/bin/hysteria/config:/tpdata/trojan-panel-core/bin/hysteria/config"
      - "/tpdata/trojan-panel-core/bin/naiveproxy/config:/tpdata/trojan-panel-core/bin/naiveproxy/config"
      - "/tpdata/trojan-panel-core/logs/:/tpdata/trojan-panel-core/logs/"
      - "/tpdata/trojan-panel-core/config/sqlite/:/tpdata/trojan-panel-core/config/sqlite/"
      - "/tpdata/caddy/cert/:/tpdata/caddy/cert/"
      - "/tpdata/caddy/srv/:/tpdata/caddy/srv/"
      - "/etc/localtime:/etc/localtime"
    environment:
      - "mariadb_ip=127.0.0.1"
      - "mariadb_port=9507"
      - "mariadb_user=root"
      - "mariadb_pas=${mariadb_pas}"
      - "database=trojan_panel_db"
      - "account-table=account"
      - "redis_host=127.0.0.1"
      - "redis_port=6378"
      - "redis_pass=${redis_pass}"
      - "crt_path=/tpdata/caddy/cert/${domain}.crt"
      - "key_path=/tpdata/caddy/cert/${domain}.key"
      - "grpc_port=8100"
```

参数解释：

- `${mariadb_pas}`：MariaDB 数据库密码
- `${redis_pass}`：Redis 的密码
- `${domain}`：你的域名