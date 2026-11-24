# Docker: приложение со счётчиком

## Запуск

```bash
cp .env.example .env
docker compose up -d --build --wait
```

## Настройки Compose

Задаются в `.env` (шаблон: `.env.example`).

| Переменная  | По умолчанию     | Значение                          |
|-------------|------------------|-----------------------------------|
| `HOST_PORT` | `8080`           | Порт приложения на компьютере     |
| `GREETING`  | `Hello, Docker!` | Текст поля `greeting`             |

## Отчёт

### Уровень 4: логи контейнера

```
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker build -t hw3-counter .
[+] Building 4.7s (11/11) FINISHED                                                                       docker:default
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 282B                                                                               0.0s
 => [internal] load metadata for docker.io/library/python:3.13-slim-bookworm                                       0.0s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 88B                                                                                   0.0s
 => [1/6] FROM docker.io/library/python:3.13-slim-bookworm@sha256:f0fa218d5b06f9b9235024fa061cde2cb141f726de52134  0.9s
 => => resolve docker.io/library/python:3.13-slim-bookworm@sha256:f0fa218d5b06f9b9235024fa061cde2cb141f726de52134  0.0s
 => => extracting sha256:b1dd3d113aab1041fd80cc61d9db7ffbc058f756bf46fc149a0d9aa8b85935f4                          0.5s
 => => extracting sha256:7fcbe89daf3349bb70a9e2cd8b1c310bf9a70ce67a8c041b2f7adef819ba68a8                          0.1s
 => => extracting sha256:ec575e739f90cae763d9aeb6467ce6e6b31c4387a029dfbb654e4994efcedfb1                          0.2s
 => => extracting sha256:8b82bf5a48c9ab96df55f5902a30d94e01930c54489652440b0402a2a88d7181                          0.0s
 => [internal] load build context                                                                                  0.0s
 => => transferring context: 5.95kB                                                                                0.0s
 => [2/6] WORKDIR /app                                                                                             0.1s
 => [3/6] COPY app/requirements.txt .                                                                              0.0s
 => [4/6] RUN pip install --no-cache-dir -r requirements.txt                                                       2.7s
 => [5/6] COPY app/app.py .                                                                                        0.0s
 => [6/6] RUN mkdir /data && chown 10001:10001 /data                                                               0.2s
 => exporting to image                                                                                             0.6s
 => => exporting layers                                                                                            0.4s
 => => exporting manifest sha256:818a29be94eda6c79a90b4027dbbfdedea29f8d971841c62874bae6f945147ae                  0.0s
 => => exporting config sha256:2bf274adf557628dd02d231dc59da4e5944741cf66afbe5c588d19af318d46bf                    0.0s
 => => exporting attestation manifest sha256:c5246fbfd79c5fafb84067e2b9a0edeede9562784354b821bce25cc9f6d3a8e5      0.0s
 => => exporting manifest list sha256:be5bf983c73370dfba7cf4479e586b8d285763781dcecfd0d2b37547eb99b715             0.0s
 => => naming to docker.io/library/hw3-counter:latest                                                              0.0s
 => => unpacking to docker.io/library/hw3-counter:latest                                                           0.1s
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker run -d --name counter -p 127.0.0.1:8080:8000 -e GREETING='Hello from a container!' hw3-counter
77f5a84f283681a620c16d68e6ea0af7323866c5c82dc9047de54d196d9a686a
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl http://127.0.0.1:8080/
{"greeting": "Hello from a container!", "count": 0, "hostname": "77f5a84f2836", "storage": "sqlite"}captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl -X POST http://127.0.0.1:8080/increment
{"greeting": "Hello from a container!", "count": 1, "hostname": "77f5a84f2836", "storage": "sqlite"}captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl -X POST http://127.0.0.1:8080/increment
{"greeting": "Hello from a container!", "count": 2, "hostname": "77f5a84f2836", "storage": "sqlite"}captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl http://127.0.0.1:8080/
{"greeting": "Hello from a container!", "count": 2, "hostname": "77f5a84f2836", "storage": "sqlite"}captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker logs counter
2026-11-24 21:20:21,666 listening port=8000 storage=sqlite
2026-11-24 21:20:35,097 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-24 21:20:44,110 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-24 21:21:11,147 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-24 21:21:18,146 http 172.17.0.1 "GET / HTTP/1.1" 200 -
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker rm -f counter
counter
```

### Уровень 6 и 8: проверки

Команды:

```bash
for i in 1 2 3; do curl -X POST http://127.0.0.1:8080/increment; done
docker compose down
docker compose up -d --wait
curl http://127.0.0.1:8080/

docker compose stop redis
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/healthz
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/readyz
curl -s -o /dev/null -w '%{http_code}\n' -X POST http://127.0.0.1:8080/increment

docker compose start redis
until curl -fs http://127.0.0.1:8080/readyz > /dev/null; do sleep 1; done
curl http://127.0.0.1:8080/
curl -X POST http://127.0.0.1:8080/increment
```

Результаты:

```
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ cp .env.example .env
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker compose config
name: docker-counter
services:
  app:
    cpus: 1
    environment:
      GREETING: Hello, Docker!
      REDIS_URL: redis://redis:6379/0
      STORAGE: redis
    init: true
    mem_limit: "268435456"
    networks:
      default: null
    read_only: true
    build:
      context: /home/captainlevi/hw_3_docker
      dockerfile: Dockerfile
    depends_on:
      redis:
        condition: service_healthy
        required: true
    healthcheck:
      test:
        - CMD
        - python
        - -c
        - import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/readyz',
          timeout=2)
      timeout: 3s
      interval: 5s
      retries: 5
      start_period: 3s
    ports:
      - mode: ingress
        host_ip: 127.0.0.1
        target: 8000
        published: "8080"
        protocol: tcp
  redis:
    command:
      - redis-server
      - --appendonly
      - "yes"
    image: redis:7.4-alpine
    networks:
      default: null
    volumes:
      - type: volume
        source: redis-data
        target: /data
        volume: {}
    healthcheck:
      test:
        - CMD-SHELL
        - redis-cli ping | grep -q PONG
      timeout: 3s
      interval: 5s
      retries: 5
      start_period: 3s
networks:
  default:
    name: docker-counter_default
volumes:
  redis-data:
    name: docker-counter_redis-data
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker compose up -d --build --wait
[+] up 10/10
 ✔ Image redis:7.4-alpine Pulled                                                         5.0s
[+] Building 0.4s (13/13) FINISHED
 => [internal] load local bake definitions                                              0.0s
 => => reading from stdin 512B                                                          0.0s
 => [internal] load build definition from Dockerfile                                    0.0s
 => => transferring dockerfile: 282B                                                    0.0s
 => [internal] load metadata for docker.io/library/python:3.13-slim-bookworm            0.0s
 => [internal] load .dockerignore                                                       0.0s
 => => transferring context: 88B                                                        0.0s
 => [1/6] FROM docker.io/library/python:3.13-slim-bookworm@sha256:f0fa218d5b06f9b92350  0.0s
 => => resolve docker.io/library/python:3.13-slim-bookworm@sha256:f0fa218d5b06f9b92350  0.0s
 => [internal] load build context                                                       0.0s
 => => transferring context: 94B                                                        0.0s
 => CACHED [2/6] WORKDIR /app                                                           0.0s
 => CACHED [3/6] COPY app/requirements.txt .                                            0.0s
 => CACHED [4/6] RUN pip install --no-cache-dir -r requirements.txt                     0.0s
 => CACHED [5/6] COPY app/app.py .                                                      0.0s
 => CACHED [6/6] RUN mkdir /data && chown 10001:10001 /data                             0.0s
 => exporting to image                                                                  0.1s
 => => exporting layers                                                                 0.0s
 => => exporting manifest sha256:fa8a451e1e284fa361a97e14c68b43e96c9f7fd1587e2e4c90bd8  0.0s
 => => exporting config sha256:c5bfca4e711672890cd03bce377d1c1cae4b48056642f6fc514cf7c  0.0s
 => => exporting attestation manifest sha256:b163c8a2158c0a415df2db6f80230ad18c10cdb95  0.0s
 => => exporting manifest list sha256:08f503ae59b8a73c02a1f14aed667f1c8ce1e03839abee55  0.0s
 => => naming to docker.io/library/docker-counter-app:latest                            0.0s
 => => unpacking to docker.io/library/docker-counter-app:latest                         0.0s
 => resolving provenance for metadata file                                              0.0s
[+] up 15/15
 ✔ Image redis:7.4-alpine           Pulled                                               5.0s
 ✔ Image docker-counter-app         Built                                                0.5s
 ✔ Network docker-counter_default   Created                                              0.0s
 ✔ Volume docker-counter_redis-data Created                                              0.0s
 ✔ Container docker-counter-redis-1 Healthy                                              4.1s
 ✔ Container docker-counter-app-1   Healthy                                              7.0s
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker compose ps
NAME                     IMAGE                COMMAND                  SERVICE   CREATED              STATUS                        PORTS
docker-counter-app-1     docker-counter-app   "python -u app.py"       app       About a minute ago   Up About a minute (healthy)   127.0.0.1:8080->8000/tcp
docker-counter-redis-1   redis:7.4-alpine     "docker-entrypoint.s…"   redis     About a minute ago   Up About a minute (healthy)   6379/tcp
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl -X POST http://127.0.0.1:8080/increment
{"greeting": "Hello, Docker!", "count": 1, "hostname": "d1401f74a55e", "storage": "redis"}captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl -X POST http://127.0.0.1:8080/increment
{"greeting": "Hello, Docker!", "count": 2, "hostname": "d1401f74a55e", "storage": "redis"}captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl -X POST http://127.0.0.1:8080/increment
{"greeting": "Hello, Docker!", "count": 3, "hostname": "d1401f74a55e", "storage": "redis"}captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker compose down
[+] down 3/3
 ✔ Container docker-counter-app-1   Removed                                              0.3s
 ✔ Container docker-counter-redis-1 Removed                                              0.2s
 ✔ Network docker-counter_default   Removed                                              0.2s
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker compose up -d --wait
[+] up 3/3
 ✔ Network docker-counter_default   Created                                              0.0s
 ✔ Container docker-counter-redis-1 Healthy                                              4.6s
 ✔ Container docker-counter-app-1   Healthy                                              7.0s
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl http://127.0.0.1:8080/
{"greeting": "Hello, Docker!", "count": 3, "hostname": "68d0e8c2c8cd", "storage": "redis"}captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker volume ls
DRIVER    VOLUME NAME
local     docker-counter_redis-data
```
```
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker compose stop redis
[+] stop 1/1
 ✔ Container docker-counter-redis-1 Stopped                                              0.2s
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/healthz
200
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/readyz
503
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl -s -o /dev/null -w '%{http_code}\n' -X POST http://127.0.0.1:8080/increment
503
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker compose start redis
[+] start 1/1
 ✔ Container docker-counter-redis-1 Started                                              0.1s
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ until curl -fs http://127.0.0.1:8080/readyz > /dev/null; do sleep 1; done
captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl http://127.0.0.1:8080/
{"greeting": "Hello, Docker!", "count": 3, "hostname": "68d0e8c2c8cd", "storage": "redis"}captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ curl -X POST http://127.0.0.1:8080/increment
{"greeting": "Hello, Docker!", "count": 4, "hostname": "68d0e8c2c8cd", "storage": "redis"}captainlevi@DESKTOP-UTLULQU:~/hw_3_docker$ docker compose down
[+] down 3/3
 ✔ Container docker-counter-app-1   Removed                                              0.3s
 ✔ Container docker-counter-redis-1 Removed                                              0.3s
 ✔ Network docker-counter_default   Removed                                              0.2s
```