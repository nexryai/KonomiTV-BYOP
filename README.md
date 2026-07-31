# KonomiTV BYOP

## これは何
[KonomiTV](https://github.com/tsukumijima/KonomiTV)に独自のリバースプロキシを設定できるようにするやつ

### Why
KonomitTVでは、セキュリティ上の理由から独自の`Akebi reverse proxy`を経由しないとアクセスできないようになっています。しかしこれは、一部のパワーユーザーが使用する場合、既にあるインフラに組み込む場合などに障壁となります。  
このプロジェクトは、リスクを理解し回避できるパワーユーザーのための、最低限の改変で独自のリバースプロキシでKonomiTVを動作可能にするパッチを提供するものです。

### Usage
Dockerメージは以下で配布しています。
```
ghcr.io/nexryai/konomitv:latest
```

ソースコードは[nexryai/KonomiTV](https://github.com/nexryai/KonomiTV)にあります。

```yaml
services:
  mirakurun:
    image: chinachu/mirakurun:latest
    container_name: mirakurun
    hostname: mirakurun
    restart: unless-stopped
    environment:
      TZ: Asia/Tokyo
      DISABLE_PCSCD: "0"
      # arib-b25-stream-testの準備を有効にする
      DISABLE_B25_TEST: "0"
    ports:
      - "40772:40772"
    devices:
      - /dev/bus:/dev/bus
      - /dev/dvb:/dev/dvb
    volumes:
      - ./mirakurun/config:/app-config
      - ./mirakurun/data:/app-data
      - ./mirakurun/opt:/opt
    tmpfs:
      - /tmp
    healthcheck:
      test:
        - CMD
        - node
        - -e
        - >-
          fetch('http://127.0.0.1:40772/api/status')
          .then(response => process.exit(response.ok ? 0 : 1))
          .catch(() => process.exit(1))
      interval: 10s
      timeout: 5s
      retries: 12
      start_period: 40s

#  mirakc:
#    image: docker.io/mirakc/mirakc:alpine
#    restart: unless-stopped
#    devices:
#      - /dev/dvb:/dev/dvb
#    volumes:
#      - ./mirakc/config.yml:/etc/mirakc/config.yml:ro
#      - ./mirakc/epg:/var/lib/mirakc/epg
#      - /run/pcscd/pcscd.comm:/run/pcscd/pcscd.comm
#    environment:
#      TZ: Asia/Tokyo
#    healthcheck:
#      test:
#        - CMD
#        - curl
#        - -fsSL
#        - http://localhost:40772/api/status
#      interval: 10s
#      timeout: 3s
#      retries: 5
#      start_period: 10s

  konomitv:
    image: ghcr.io/nexryai/konomitv:latest
    restart: unless-stopped
    depends_on:
      mirakurun:
        condition: service_healthy
        restart: true
    ports:
      - "127.0.0.1:7000:7000"
    devices:
      - /dev/dri/renderD128:/dev/dri/renderD128
    volumes:
      - ./KonomiTV/config.yaml:/code/config.yaml:ro
      - ./KonomiTV/server/data:/code/server/data
      - ./KonomiTV/server/logs:/code/server/logs
    environment:
      TZ: Asia/Tokyo
```
