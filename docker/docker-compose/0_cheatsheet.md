# Cheatsheet


- [Cheatsheet](#cheatsheet)
    - [1. host network](#1-host-network)

### 1. host network
docker-compose에서 컨테이너에 host network을 할당하려면 `docker-compose.yaml`에 `network_mode`를 `host`로 설정한다.

```yaml
version: '2.2'
services:
  es01:
    image: realsangil/example
    container_name: example
    network_mode: host
```

