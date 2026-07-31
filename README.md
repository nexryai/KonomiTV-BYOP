# KonomiTV BYOP

## これは何
[KonomiTV](https://github.com/tsukumijima/KonomiTV)に独自のリバースプロキシを設定できるようにするやつ

### Why
KonomitTVではセキュリティ上の理由から独自の`Akebi reverse proxy`を使用しないとアクセスできないようになっています。しかしこれは、一部のパワーユーザーや既にある環境に組み込む場合、障壁となります。  
最低限の改変で独自のリバースプロキシでKonomiTVを動作可能にするパッチです。

### Usage
Dockerメージは以下で配布しています。
```
ghcr.io/nexryai/konomitv:latest
```

ソースコードは[nexryai/KonomiTV](https://github.com/nexryai/KonomiTV)にあります。

```yaml
services:
  mirakc:
    image: docker.io/mirakc/mirakc:alpine
    restart: unless-stopped
    devices:
      - /dev/dvb:/dev/dvb
    volumes:
      - ./mirakc/config.yml:/etc/mirakc/config.yml:ro
      - ./mirakc/epg:/var/lib/mirakc/epg
      - /run/pcscd/pcscd.comm:/run/pcscd/pcscd.comm
    environment:
      TZ: Asia/Tokyo
    healthcheck:
      test:
        - CMD
        - curl
        - -fsSL
        - http://localhost:40772/api/status
      interval: 10s
      timeout: 3s
      retries: 5
      start_period: 10s

  konomitv:
    image: ghcr.io/nexryai/konomitv:latest
    restart: unless-stopped
    depends_on:
      mirakc:
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
