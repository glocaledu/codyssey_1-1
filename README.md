# Codyssey 입학연수 미션1 — 내 컴퓨터에 개발자용 '작업실' 꾸미기

## 0) 프로젝트 개요
터미널(CLI) · Docker · Git/GitHub 를 직접 손으로 세팅하고, **"내 PC에서만 되는 코드"가 아니라 누구나 같은 절차로 실행·검증할 수 있는 환경**을 만드는 미션입니다.

이 저장소에는 아래 산출물이 들어 있습니다.
- 직접 작성한 `Dockerfile` (`nginx:alpine` 기반 커스텀 이미지)와 정적 콘텐츠 `site/index.html`
- 포트 매핑 · 바인드 마운트 · 볼륨 영속성 · 컨테이너 운영 명령의 **실행 로그(명령어 + 출력)**
- `hello-world` / `ubuntu` 컨테이너 실습, `run` · `exec` · `attach` 차이 관찰
- Git 설정과 트러블슈팅 기록

> 모든 명령은 터미널(CLI)에서 수행했고, 로그의 `$` 는 명령어 입력, 그 아래 줄은 출력입니다.

---

## 1) 실행 환경
| 항목 | 값 |
|---|---|
| OS | Windows + WSL2 (Ubuntu 24.04.1 LTS, x86_64) |
| Shell | GNU bash 5.3.9 |
| Docker | Docker Desktop (WSL2 Engine), Docker 29.8.0 (Compose v5.5.1 포함) |
| Git | 2.53.0 |
| 터미널 | Windows Terminal + WSL2 Ubuntu |
| 작업 폴더 | `~/projects/myapp/codyssey_1-1` (`HOME=/home/daniel`) |

WSL 배포판 상태 (Windows PowerShell):
```text
PS C:\Windows\system32> wsl -l -v
  NAME              STATE           VERSION
* Ubuntu            Running         2
  docker-desktop    Running         2
```

버전 확인 명령:
```text
TODO(붙여넣기): lsb_release -a / bash --version / docker --version / git --version 출력
```

> **재현 시 주의(PC 의존 사항)**: 이 환경은 Windows + Docker Desktop(WSL2) 기준입니다. Docker Desktop의 **Settings → Resources → WSL Integration** 에서 사용 중인 Ubuntu 배포판이 켜져 있어야 WSL 터미널에서 `docker` 명령이 동작합니다. (5-1 참고)

## 2) 수행 체크리스트
- [ ] 터미널 기본 조작 · 권한 변경 실습 (3장, `practice.sh` 실행 후 체크)
- [x] Docker 설치/점검 (`docker --version`, `docker info`)
- [x] `hello-world` 실행
- [x] `ubuntu` 컨테이너 실행, 내부 진입 후 `ls` · `echo` 수행
- [x] `run` / `exec` / `attach` 차이 관찰
- [x] 직접 작성한 Dockerfile 로 커스텀 이미지 빌드/실행 (`nginx:alpine` 기반)
- [ ] 포트 매핑 접속 2회 (8080 확인 완료, 8081 `curl` 증거와 브라우저 접속 화면 추가 후 체크)
- [x] 바인드 마운트 반영 (호스트 수정 → 재빌드 없이 반영)
- [x] 볼륨 영속성 (컨테이너 삭제 전/후 + 볼륨 없는 대조군)
- [x] 컨테이너 운영 명령 (`images`, `ps -a`, `logs`, `stats`, `stop`/`start`)
- [ ] Git 사용자 정보 · 기본 브랜치 설정 (`git config --list` 최종 기록 후 체크)
- [ ] VSCode GitHub 로그인 · 저장소 연동 증거
- [ ] (보너스) Compose / SSH 키 — 미수행
- [x] 민감정보(토큰 · 비밀번호) 미포함 확인

## 3) 터미널 조작 · 권한 실습 로그
`~/projects/myapp/codyssey_1-1/practice` 에서 실습했습니다. (`HOME=/home/daniel`)
현재 위치 확인(`pwd`), 목록(`ls -la`, 숨김 파일 포함), 이동, 생성, 복사, 이동/이름변경, 삭제, 파일 내용 확인(`cat`), 빈 파일 생성(`touch`), 권한 변경(`chmod`, 파일 1개 + 디렉토리 1개)을 수행합니다.

실행한 스크립트(`practice/practice.sh`):
```bash
PS4='$ '; set -x
pwd
mkdir -p docs
touch empty.txt .hidden_file
echo "hello codyssey" > docs/note.txt
ls -la
cat docs/note.txt
cp docs/note.txt docs/note_copy.txt
mv docs/note_copy.txt docs/renamed.txt
ls -la docs
rm docs/renamed.txt
ls docs
cd docs
pwd
cat ~/projects/myapp/codyssey_1-1/practice/docs/note.txt
cat note.txt
cat ../docs/note.txt
cd ..
ls -l docs/note.txt
chmod 600 docs/note.txt
ls -l docs/note.txt
chmod 755 docs/note.txt
ls -l docs/note.txt
chmod 644 docs/note.txt
ls -l docs/note.txt
ls -ld docs
chmod 700 docs
ls -ld docs
chmod 755 docs
ls -ld docs
```

실행 결과 (`bash practice.sh 2>&1 | tee ../logs/terminal.log`):
```text
TODO(붙여넣기): logs/terminal.log 출력
```

**관찰 정리**
- **절대경로**는 `/`(루트)부터 시작해서 어디서 실행해도 같은 위치를 가리킵니다. (예: `/home/daniel/projects/myapp/codyssey_1-1/practice/docs/note.txt`) **상대경로**는 현재 위치(`pwd`) 기준이라 `note.txt`, `../docs/note.txt` 처럼 위치가 바뀌면 의미가 달라집니다.
- 권한은 `r=4, w=2, x=1` 의 합입니다. 755 = `rwxr-xr-x`(소유자 rwx, 그룹 r-x, 기타 r-x), 644 = `rw-r--r--`, 600 = `rw-------`, 700 = `rwx------` 입니다.
- 디렉토리의 `x` 는 "그 안으로 들어가고(`cd`) 이름으로 접근할 수 있는 권한"이라 디렉토리는 보통 755 입니다.

## 4) Docker 운영 · 검증 로그

### 4-1) 설치 · 데몬 점검
```text
TODO(붙여넣기): docker --version / docker info / docker version(Client와 Server 모두 나오는지) 출력
```
Docker Desktop 이 켜져 있어야 `docker info` 의 Server 항목이 나옵니다.

### 4-2) 직접 작성한 Dockerfile · 커스텀 이미지 (nginx:alpine)
과제의 **(A) 웹 서버 베이스 이미지 활용** 방식을 선택했습니다. 기존 `nginx:alpine` 을 베이스로 하고, 정적 콘텐츠와 설정만 바꿨습니다.

**선택한 기존 베이스:** `nginx:alpine` (공식 nginx 이미지의 경량 Alpine 버전)

작업 폴더 구성:
```text
codyssey_1-1/
├── Dockerfile
└── site/
    └── index.html
```

`site/index.html` (빌드 시점의 내용)
```html
<h1>Codyssey Mission1</h1>
<p>version: v1 (이미지에 포함된 기본 페이지)</p>
```

`Dockerfile`
```dockerfile
FROM nginx:alpine
LABEL org.opencontainers.image.title="my-custom-nginx"
ENV APP_ENV=dev
COPY site/ /usr/share/nginx/html/
EXPOSE 80
HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD wget -qO- http://localhost/ >/dev/null || exit 1
```

| 커스텀 포인트 | 목적 |
|---|---|
| `FROM nginx:alpine` | 웹 서버가 이미 설치된 검증된 베이스를 재사용. Alpine 이라 이미지가 작음 |
| `LABEL` | 이미지 이름 등 메타데이터 표기 |
| `ENV APP_ENV=dev` | 설정과 코드 분리. 실행 시 `-e APP_ENV=...` 로 덮어쓰기 가능 |
| `COPY site/ /usr/share/nginx/html/` | nginx 기본 페이지를 내가 만든 정적 콘텐츠로 교체 |
| `EXPOSE 80` | 컨테이너가 사용하는 포트를 문서화 (실제 공개는 `-p` 로 결정) |
| `HEALTHCHECK` | 컨테이너가 정상 응답하는지 Docker 가 10초마다 확인 |

빌드 명령과 출력 (`nginx:alpine` 을 처음 내려받는 로그 포함):
```text
$ docker build -t my-web:1.0 .
[+] Building 6.7s (7/7) FINISHED                                                    docker:default
 => [internal] load build definition from Dockerfile                                          0.0s
 => => transferring dockerfile: 278B                                                          0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine                               2.4s
 => [internal] load .dockerignore                                                             0.0s
 => => transferring context: 2B                                                               0.0s
 => [internal] load build context                                                             0.1s
 => => transferring context: 157B                                                             0.0s
 => [1/2] FROM docker.io/library/nginx:alpine@sha256:713fc2ebfd92e8217db451477182ee91944b123  3.5s
 => => resolve docker.io/library/nginx:alpine@sha256:713fc2ebfd92e8217db451477182ee91944b123  0.0s
 => => sha256:da90db506dd5f2d604467c4dc38442534f74c9e76c1982df0c262e144bad 20.55MB / 20.55MB  2.6s
 => => sha256:69d81d5522572d5b7abcc8f594ecbc4cf9bea72402675a419e8d7b516e4c3f 1.40kB / 1.40kB  0.5s
 => => sha256:b9c6ee6a30f8811636d7273be348906cf0e4e9725378dd54b7d4a3725538ca 1.21kB / 1.21kB  0.6s
 => => sha256:a746bd1dc828f61bf43b775dcfecb9ded8a3d3b41789915fe135fa6b9fa85965 404B / 404B    0.6s
 => => sha256:e011f124249bf1c4be27b3311c3b40cb90dd7737381af1c62b508020790b7bd5 965B / 965B    0.2s
 => => sha256:c7dfd64eeb02740cc41e9f32ad7f75781882d4c167b77427aa9615d1419c4515 629B / 629B    0.2s
 => => sha256:1abafee971b93c1ae05f03c35c6fb6e3376e75e6f16b590092a822b61ebf10 1.94MB / 1.94MB  0.6s
 => => sha256:881fc932939efef04ca120ad44bbd2c0917851eb247aa4995428afb1e80ede 3.85MB / 3.85MB  1.3s
 => => extracting sha256:881fc932939efef04ca120ad44bbd2c0917851eb247aa4995428afb1e80ede17     0.3s
 => => extracting sha256:1abafee971b93c1ae05f03c35c6fb6e3376e75e6f16b590092a822b61ebf10b6     0.3s
 => => extracting sha256:c7dfd64eeb02740cc41e9f32ad7f75781882d4c167b77427aa9615d1419c4515     0.0s
 => => extracting sha256:e011f124249bf1c4be27b3311c3b40cb90dd7737381af1c62b508020790b7bd5     0.0s
 => => extracting sha256:a746bd1dc828f61bf43b775dcfecb9ded8a3d3b41789915fe135fa6b9fa85965     0.0s
 => => extracting sha256:b9c6ee6a30f8811636d7273be348906cf0e4e9725378dd54b7d4a3725538cad4     0.0s
 => => extracting sha256:69d81d5522572d5b7abcc8f594ecbc4cf9bea72402675a419e8d7b516e4c3f14     0.0s
 => => extracting sha256:da90db506dd5f2d604467c4dc38442534f74c9e76c1982df0c262e144bad7bc9     0.6s
 => [2/2] COPY site/ /usr/share/nginx/html/                                                   0.2s
 => exporting to image                                                                        0.3s
 => => exporting layers                                                                       0.1s
 => => exporting manifest sha256:51429c53f0b882b8458a7dd24a4d9d5425ce993f5c1db6b4c87ca5d8342  0.0s
 => => exporting config sha256:ae8f10b9a3625496e3240986e03adc434d26bc6eba5ffdfaf5bc3b47bacd7  0.0s
 => => exporting attestation manifest sha256:4d63cdf0f27f8ab59380bfb670b2a6749078d24ab7be606  0.0s
 => => exporting manifest list sha256:2ce8aaaa5649c0019f2b8d8507b58fba7f3b6ceb8e214979eb97b5  0.0s
 => => naming to docker.io/library/my-web:1.0                                                 0.0s
 => => unpacking to docker.io/library/my-web:1.0
```

### 4-3) 포트 매핑 접속 (2회)
컨테이너 내부 포트는 nginx 기본인 **80** 으로 같고, 호스트 포트만 8080 / 8081 로 달리 매핑했습니다. 8081 컨테이너는 `-e APP_ENV=staging` 으로 Dockerfile 의 기본값(`dev`)을 덮어썼습니다.

```bash
docker run -d -p 8080:80 --name my-web-8080 my-web:1.0
docker run -d -p 8081:80 -e APP_ENV=staging --name my-web-8081 my-web:1.0
```

**1회차: 8080 접속 (`curl -i`)**
```text
$ curl -i http://localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.31.8
Date: Mon, 09 Nov 2026 08:28:55 GMT
Content-Type: text/html
Content-Length: 87
Last-Modified: Mon, 09 Nov 2026 08:24:39 GMT
Connection: keep-alive
ETag: "6af18347-57"
Accept-Ranges: bytes

<h1>Codyssey Mission1</h1>
<p>version: v1 (이미지에 포함된 기본 페이지)</p>
```

**2회차: 8081 접속**
```text
TODO(붙여넣기): curl -i http://localhost:8081 출력
```

**환경변수 주입 확인 (8081)**
```text
$ docker exec my-web-8081 printenv APP_ENV
staging
```

두 포트가 동시에 열려 있는 증거는 4-4 의 `docker ps -a` 출력(`0.0.0.0:8080->80/tcp`, `0.0.0.0:8081->80/tcp`)에 있습니다.

**브라우저 접속 화면 (주소창 포함)**
```text
TODO(스크린샷 첨부): http://localhost:8080 접속 화면 (주소창과 응답 화면이 같이 보이게)
```

### 4-4) 바인드 마운트 · 운영 명령
호스트의 `site/` 를 컨테이너의 `/usr/share/nginx/html` 에 바인드 마운트(8082)한 뒤, **호스트에서만** 파일을 수정했습니다. 재빌드 · 재시작 없이 응답이 v1 → v2 로 바뀌었고, 마운트하지 않은 컨테이너(8080)는 이미지에 포함된 v1 그대로입니다.

아래는 `logs/run-4-4.sh` 를 `bash logs/run-4-4.sh 2>&1 | tee logs/docker-ops.log` 로 실행한 로그입니다.
- `$ echo '...'` 줄은 `site/index.html` 에 덮어쓰는 명령입니다. (`set -x` 는 리다이렉션 `> site/index.html` 을 출력하지 않습니다.)
- `docker logs` 의 `"Wget"` 요청은 Dockerfile 의 `HEALTHCHECK` 가 10초마다 호출한 것이고, `curl/8.18.0` 이 제가 보낸 요청입니다.

```text
$ bash logs/run-4-4.sh 2>&1 | tee logs/docker-ops.log
$ echo '<h1>Codyssey Mission1</h1><p>version: v1 (호스트 원본)</p>'
$ cat site/index.html
<h1>Codyssey Mission1</h1><p>version: v1 (호스트 원본)</p>
$ docker run -d -p 8082:80 -v /home/daniel/projects/myapp/codyssey_1-1/site:/usr/share/nginx/html --name my-web-bind my-web:1.0
160c752b81114a4bbdc7610f1a2e0f5cf9d13763023ca9ef8e2521b71a7c0b64
$ sleep 2
$ docker inspect -f '{{range .Mounts}}{{.Type}} {{.Source}} -> {{.Destination}}{{end}}' my-web-bind
bind /home/daniel/projects/myapp/codyssey_1-1/site -> /usr/share/nginx/html
$ curl http://localhost:8082
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100     64 100     64   0      0  36908      0                              0
<h1>Codyssey Mission1</h1><p>version: v1 (호스트 원본)</p>
$ echo '<h1>Codyssey Mission1</h1><p>version: v2 (호스트에서 수정, 재빌드 없음)</p>'
$ cat site/index.html
<h1>Codyssey Mission1</h1><p>version: v2 (호스트에서 수정, 재빌드 없음)</p>
$ curl http://localhost:8082
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100     88 100     88   0      0  62015      0                              0
<h1>Codyssey Mission1</h1><p>version: v2 (호스트에서 수정, 재빌드 없음)</p>
$ curl http://localhost:8080
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100     87 100     87   0      0  66666      0                              0
<h1>Codyssey Mission1</h1>
<p>version: v1 (이미지에 포함된 기본 페이지)</p>
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                            PORTS                                     NAMES
160c752b8111   my-web:1.0     "/docker-entrypoint.…"   2 seconds ago   Up 2 seconds (health: starting)   0.0.0.0:8082->80/tcp, [::]:8082->80/tcp   my-web-bind
e5e2a288fa5e   2ce8aaaa5649   "/docker-entrypoint.…"   24 hours ago    Up 2 minutes (healthy)            0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   my-web-8081
4317ff6167bd   2ce8aaaa5649   "/docker-entrypoint.…"   24 hours ago    Up 2 minutes (healthy)            0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   my-web-8080
b5e75091fdf0   hello-world    "/hello"                 6 days ago      Exited (0) 6 days ago                                                       flamboyant_ride
f1f7ca62e984   nginx          "/docker-entrypoint.…"   6 days ago      Exited (255) 25 hours ago         0.0.0.0:8080->80/tcp                      amazing_meitner
78fb90dd574d   hello-world    "/hello"                 6 days ago      Exited (0) 6 days ago                                                       eloquent_hawking
$ docker stats --no-stream
CONTAINER ID   NAME          CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
160c752b8111   my-web-bind   0.00%     15.2MiB / 7.721GiB    0.19%     2.05kB / 1.47kB   4.59MB / 8.19kB   17
e5e2a288fa5e   my-web-8081   0.00%     15.96MiB / 7.721GiB   0.20%     1.78kB / 126B     938kB / 4.1kB     17
4317ff6167bd   my-web-8080   0.00%     16.16MiB / 7.721GiB   0.20%     2.58kB / 829B     10.7MB / 4.1kB    17
$ docker logs --tail 5 my-web-8080
::1 - - [10/Nov/2026:08:09:35 +0000] "GET / HTTP/1.1" 200 87 "-" "Wget" "-"
::1 - - [10/Nov/2026:08:09:45 +0000] "GET / HTTP/1.1" 200 87 "-" "Wget" "-"
::1 - - [10/Nov/2026:08:09:55 +0000] "GET / HTTP/1.1" 200 87 "-" "Wget" "-"
::1 - - [10/Nov/2026:08:10:05 +0000] "GET / HTTP/1.1" 200 87 "-" "Wget" "-"
172.17.0.1 - - [10/Nov/2026:08:10:07 +0000] "GET / HTTP/1.1" 200 87 "-" "curl/8.18.0" "-"
$ docker exec my-web-8080 sh -c 'ls /usr/share/nginx/html'
50x.html
index.html
$ docker stop my-web-8081
my-web-8081
$ docker ps -a --filter name=my-web-8081
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS                              PORTS     NAMES
e5e2a288fa5e   2ce8aaaa5649   "/docker-entrypoint.…"   24 hours ago   Exited (0) Less than a second ago             my-web-8081
$ docker start my-web-8081
my-web-8081
$ docker ps --filter name=my-web-8081
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS                                     PORTS                                     NAMES
e5e2a288fa5e   2ce8aaaa5649   "/docker-entrypoint.…"   24 hours ago   Up Less than a second (health: starting)   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   my-web-8081
```

**관찰 정리**
- 8082(바인드 마운트)는 호스트 파일을 수정하는 즉시 `v2` 로 바뀌었고, 8080(마운트 없음)은 이미지에 구워 넣은 `v1` 그대로였습니다. 이미지는 읽기 전용 설계도이고, 바인드 마운트는 호스트 폴더를 그대로 연결한 것이기 때문입니다.
- `docker stop` 으로 정상 종료하면 `Exited (0)` 입니다. (PC 재시작으로 꺼진 경우는 `Exited (255)`, 5-4 참고)
- `docker stats --no-stream` 은 현재 시점의 CPU/메모리를 한 번만 출력합니다. 옵션이 없으면 화면이 계속 갱신됩니다.

### 4-5) 볼륨 영속성 검증 (대조군 포함)
`mydata` 볼륨을 만들어 컨테이너 `/data` 에 연결하고 파일을 기록한 뒤, 컨테이너를 삭제하고 **새 컨테이너에 같은 볼륨을 연결**해 데이터가 남는지 확인했습니다. 대조군으로 볼륨 없이 같은 순서를 해서 데이터가 사라지는 것도 확인했습니다. (`logs/run-4-5.sh`)

```text
$ bash logs/run-4-5.sh 2>&1 | tee logs/docker-volume.log
$ docker volume create mydata
mydata
$ docker run -d --name vol-test -v mydata:/data my-web:1.0
5ceef087bb32ddfe6ad3ab6fa519920a5cd85c9fa00a83219b8679bdf8ce1dc4
$ sleep 2
$ docker inspect -f '{{range .Mounts}}{{.Type}} {{.Name}} -> {{.Destination}}{{end}}' vol-test
volume mydata -> /data
$ docker exec vol-test sh -c 'echo hi > /data/hello.txt && cat /data/hello.txt'
hi
$ docker rm -f vol-test
vol-test
$ docker volume ls
DRIVER    VOLUME NAME
local     mydata
$ docker run -d --name vol-test2 -v mydata:/data my-web:1.0
4346f71ba737b9ef818069803bf50712380d51a8ce6f8616e92e1a10d4198010
$ sleep 2
$ docker exec vol-test2 cat /data/hello.txt
hi
$ docker run -d --name novol my-web:1.0
6285c1df4c25de78b88e03ecc27f470ec2c8901c000f67da03059969f6aa5ebf
$ sleep 2
$ docker exec novol sh -c 'mkdir -p /data && echo hi > /data/hello.txt && cat /data/hello.txt'
hi
$ docker rm -f novol
novol
$ docker run -d --name novol2 my-web:1.0
fe3b350b9aecbf72fef2d0bf34ca2182b7233c7aa5e03f37fe12586abc4b6ecc
$ sleep 2
$ docker exec novol2 cat /data/hello.txt
cat: can't open '/data/hello.txt': No such file or directory
$ docker rm -f vol-test2 novol2
vol-test2
novol2
```

| 단계 | 결과 | 의미 |
|---|---|---|
| 볼륨 연결 후 `hello.txt` 기록 | `hi` | 볼륨 안에 데이터 기록 |
| `docker rm -f vol-test` 후 `docker volume ls` | `mydata` 남아 있음 | 컨테이너가 삭제돼도 볼륨은 유지 |
| 새 컨테이너 `vol-test2` 에서 `cat` | `hi` | **삭제 전 데이터가 이어짐 (영속성 증거)** |
| 대조군(볼륨 없음) 삭제 후 재생성 `novol2` 에서 `cat` | `No such file or directory` | 볼륨이 없으면 컨테이너와 함께 데이터가 사라짐 |

### 4-6) hello-world · ubuntu 컨테이너 실습, run / exec / attach 차이

**hello-world · ubuntu 실행** (`logs/run-4-6.sh`, 이미지 목록에서 이 미션과 무관한 이미지 1개는 생략)
```text
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

$ docker run --rm ubuntu bash -c 'head -2 /etc/os-release; ls /; echo hello from ubuntu'
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
f3f25b84a7d7: Pulling fs layer
edafa32f5d9e: Pulling fs layer
f3f25b84a7d7: Download complete
21594acc2d8b: Download complete
edafa32f5d9e: Download complete
f3f25b84a7d7: Pull complete
edafa32f5d9e: Pull complete
Digest: sha256:6a06111c3b0a8600feadb270aec771dc239bf407b25625f93f44ab00e11f4d3c
Status: Downloaded newer image for ubuntu:latest
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
bin
boot
dev
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
hello from ubuntu
$ docker images
IMAGE                  ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest     c567b231f656       25.9kB         9.49kB   U
my-web:1.0             2bf0cf6bca4d       93.7MB         26.4MB   U
nginx:latest           5ab5b0b01097        243MB         66.8MB   U
ubuntu:latest          6a06111c3b0a        163MB         45.9MB
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                      PORTS                                     NAMES
bc325ce67187   hello-world    "/hello"                 13 seconds ago   Exited (0) 12 seconds ago                                             inspiring_benz
160c752b8111   my-web:1.0     "/docker-entrypoint.…"   5 minutes ago    Up 5 minutes (healthy)      0.0.0.0:8082->80/tcp, [::]:8082->80/tcp   my-web-bind
e5e2a288fa5e   2ce8aaaa5649   "/docker-entrypoint.…"   24 hours ago     Up 5 minutes (healthy)      0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   my-web-8081
4317ff6167bd   2ce8aaaa5649   "/docker-entrypoint.…"   24 hours ago     Up 7 minutes (healthy)      0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   my-web-8080
b5e75091fdf0   hello-world    "/hello"                 6 days ago       Exited (0) 6 days ago                                                 flamboyant_ride
f1f7ca62e984   nginx          "/docker-entrypoint.…"   6 days ago       Exited (255) 25 hours ago   0.0.0.0:8080->80/tcp                      amazing_meitner
78fb90dd574d   hello-world    "/hello"                 6 days ago       Exited (0) 6 days ago                                                 eloquent_hawking
```

**① `docker run -it` : 새 컨테이너를 만들고 내부 진입 → `exit` 하면 종료**
```text
$ docker run -it --name ubuntu-test ubuntu bash
root@dd4ecaebd33d:/# pwd
/
root@dd4ecaebd33d:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@dd4ecaebd33d:/# echo "hello from ubuntu"
hello from ubuntu
root@dd4ecaebd33d:/# exit
exit
$ docker ps -a --filter name=ubuntu-test
CONTAINER ID   IMAGE     COMMAND   CREATED              STATUS                      PORTS     NAMES
dd4ecaebd33d   ubuntu    "bash"    About a minute ago   Exited (0) 56 seconds ago             ubuntu-test
```

**② `docker exec -it` : 실행 중인 컨테이너에 새 프로세스로 진입 → `exit` 해도 컨테이너는 유지**
```text
$ docker run -dit --name ubuntu-keep ubuntu bash
f7d0374257b363ce76ffcc6ad30a991dd8e034196cd2ab590297a19b064f132c
$ docker exec -it ubuntu-keep bash
root@f7d0374257b3:/# echo "Here is exec"
Here is exec
root@f7d0374257b3:/# exit
exit
$ docker ps --filter name=ubuntu-keep
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
f7d0374257b3   ubuntu    "bash"    4 minutes ago   Up 4 minutes             ubuntu-keep
```

**③ `docker attach` : 메인 프로세스(bash)에 직접 연결 → `exit` 하면 컨테이너 종료**
```text
$ docker attach ubuntu-keep
root@f7d0374257b3:/# echo "attach"
attach
root@f7d0374257b3:/# exit
exit
$ docker ps --filter name=ubuntu-keep
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
$ docker ps -a --filter name=ubuntu-keep
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS                     PORTS     NAMES
f7d0374257b3   ubuntu    "bash"    7 minutes ago   Exited (0) 2 minutes ago             ubuntu-keep
```

| 구분 | `docker run -it` | `docker exec -it` | `docker attach` |
|---|---|---|---|
| 하는 일 | 새 컨테이너를 만들어 시작 | **이미 실행 중인** 컨테이너 안에서 새 프로세스 실행 | 컨테이너의 **메인 프로세스** 입출력에 연결 |
| `exit` 후 컨테이너 | 종료 (`Exited (0)`) | **유지 (`Up`)** | 종료 (`Exited (0)`) |

- 같은 컨테이너(`ubuntu-keep`)에서 `exec` 로 들어갔다 나오면 `Up` 이 유지됐고, `attach` 로 들어갔다 나오니 `Exited (0)` 가 됐습니다. `attach` 는 메인 프로세스에 붙기 때문에 거기서 `exit` 하면 메인 프로세스가 끝나 컨테이너도 종료되고, `exec` 는 별도 프로세스라 컨테이너가 유지됩니다. (종료하지 않고 빠져나오려면 `Ctrl+P` → `Ctrl+Q`)
- **호스트와 컨테이너의 OS 가 다릅니다.** WSL 은 Ubuntu 24.04.1 인데 `ubuntu` 컨테이너 안에서는 `Ubuntu 26.04.1 LTS` 가 나왔습니다. 컨테이너는 호스트와 커널은 공유하지만 파일시스템/라이브러리 등 사용자 공간은 이미지에 따로 들어 있습니다.
- `ubuntu:latest` 는 시점에 따라 버전이 바뀔 수 있습니다. 위 결과는 2026-11-10 에 받은 이미지 기준이고, 재현성이 중요하면 `ubuntu:24.04` 처럼 태그를 지정합니다.

## 5) 트러블슈팅

### 5-1) WSL 에서 `docker` 명령을 찾지 못함
- **문제**
  ```text
  $ docker --version
  The command 'docker' could not be found in this WSL 2 distro.
  We recommend to activate the WSL integration in Docker Desktop settings.
  ```
- **원인 가설**: Docker Desktop 의 WSL 연동이 이 Ubuntu 배포판에 켜져 있지 않아 CLI 가 연결되지 않음
- **확인**: `wsl -l -v` 에서 `Ubuntu`, `docker-desktop` 모두 WSL **2** 로 Running 이어서 WSL 버전 문제는 아님
- **해결**: Docker Desktop → Settings → Resources → WSL Integration 에서 Ubuntu 배포판 연동을 켜고 터미널을 새로 연 뒤 `docker info` 가 정상 출력됨

### 5-2) `Dockerfile cannot be empty`
- **문제**
  ```text
  $ docker build -t my-web:1.0 .
  [+] Building 0.2s (1/1) FINISHED                                                    docker:default
   => [internal] load build definition from Dockerfile                                          0.1s
   => => transferring dockerfile: 31B                                                           0.0s
  ERROR: failed to build: failed to solve: the Dockerfile cannot be empty
  ```
- **원인 가설**: VSCode 에서 입력한 내용이 저장되기 전(또는 일부만 저장된 상태)에 빌드해서 Dockerfile 에 명령이 없음
- **확인**: `transferring dockerfile: 31B` 로 내 Dockerfile(약 300B)보다 훨씬 작음. 저장 후 다시 빌드하니 `transferring dockerfile: 278B` 로 바뀜
- **해결**: Dockerfile 을 저장하고 `cat Dockerfile` 로 터미널에서 내용을 확인한 뒤 다시 빌드 → 4-2 와 같이 성공

### 5-3) 바인드 마운트한 컨테이너가 호스트 수정사항을 반영하지 않음 (경로 오타)
- **문제**: 호스트에서 `site/index.html` 을 v2 로 바꿨는데 8082 응답이 `v1` 그대로
  ```text
  $ docker run -d -p 8082:80 -v "$PWD/site:/usr/share/ngnix/html" --name my-web-bind my-web:1.0
  2fb0227bbab087a5962c952376b5827cb6f8939f466ec034adf115daa8ec8b08
  $ curl http://localhost:8082
  <h1>Codyssey Mission1</h1>
  <p>version: v1 (이미지에 포함된 기본 페이지)</p>
  ```
- **원인 가설**: 마운트 대상 경로가 nginx 문서 루트와 다름. 실행 명령의 `/usr/share/ngnix/html` 이 `nginx` 가 아니라 **`ngnix`** 오타여서 엉뚱한 경로에 마운트되고, nginx 는 이미지 안의 원래 파일을 보여줌
- **확인**: 명령의 철자를 확인하고, 수정 후 `docker inspect` 로 마운트 대상을 확인
- **해결**: `my-web-bind` 삭제 후 `/usr/share/nginx/html` 로 다시 실행 → 4-4 에서 `bind .../site -> /usr/share/nginx/html` 로 확인되고 응답이 v2 로 바뀜

### 5-4) PC 재시작 후 컨테이너가 모두 `Exited (255)`
- **문제**: 다음 날 `docker ps -a` 에서 8080 / 8081 / 8082 컨테이너가 모두 종료 상태
  ```text
  $ docker ps -a
  CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS                        PORTS                  NAMES
  c1177355c73e   2ce8aaaa5649   "/docker-entrypoint.…"   23 hours ago   Exited (255) 31 minutes ago   0.0.0.0:8082->80/tcp   my-web-bind
  e5e2a288fa5e   2ce8aaaa5649   "/docker-entrypoint.…"   24 hours ago   Exited (255) 31 minutes ago   0.0.0.0:8081->80/tcp   my-web-8081
  4317ff6167bd   2ce8aaaa5649   "/docker-entrypoint.…"   24 hours ago   Exited (255) 31 minutes ago   0.0.0.0:8080->80/tcp   my-web-8080
  ```
- **원인 가설**: 컨테이너 오류가 아니라 PC / Docker Desktop 재시작으로 모든 컨테이너가 한꺼번에 종료됨 (같은 시각에 종료)
- **확인**: 세 컨테이너의 종료 시각이 모두 `31 minutes ago` 로 같고, 이미지와 컨테이너는 남아 있음
- **해결**: `docker start my-web-8080 my-web-8081` 로 다시 시작 → 4-4 의 `docker ps -a` 에서 `Up (healthy)` 확인. 컨테이너는 삭제하지 않는 한 `docker start` 로 다시 켤 수 있음

### 5-5) 명령 오타로 인한 에러 4건
| 입력한 명령 | 에러 메시지 | 원인 → 해결 |
|---|---|---|
| `cat /root/projects/myapp/codyssey_1-1/practice/docs/note.txt` | `Permission denied` | 홈 경로를 `/root` 로 잘못 씀 (`/root` 는 root 전용). 내 홈은 `/home/daniel` → `~/...` 또는 `/home/daniel/...` 로 수정 |
| `docker ps -a --filter name-ubuntu-test` | `bad format of filter (expected name=value)` | `=` 를 `-` 로 입력 → `name=ubuntu-test` 로 수정 |
| `docker run -dit --name ubuntu-keep ununtu bash` | `pull access denied for ununtu, repository does not exist` | 이미지 이름 오타 (`ununtu`) → `ubuntu` 로 수정 |
| `it config --global --list` | `it: command not found` | `git` 의 `g` 누락 → `git config --global --list` |

## 6) Git 설정

**로컬 저장소 설정 · 원격 · 브랜치**
```text
$ git config --local --list
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
remote.origin.url=https://github.com/glocaledu/codyssey_1-1
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.main.remote=origin
branch.main.merge=refs/heads/main
branch.main.vscode-merge-base=origin/main
$ git branch --show-current
main
$ git remote -v
origin  https://github.com/glocaledu/codyssey_1-1 (fetch)
origin  https://github.com/glocaledu/codyssey_1-1 (push)
```

**전역 설정** (이메일은 마스킹)
```text
$ git config --global --list
user.name=glocaledu
user.emai=g***@gmail.com
user.email=g***@gmail.com
init.defaultbranch=main
credential.https://github.com.helper=
credential.https://github.com.helper=!/usr/bin/gh auth git-credential
credential.https://gist.github.com.helper=
credential.https://gist.github.com.helper=!/usr/bin/gh auth git-credential
```
- 사용자 정보(`user.name`, `user.email`)와 기본 브랜치(`init.defaultbranch=main`)를 설정했습니다.
- 설정 중 `user.email` 을 `user.emai` 로 잘못 입력한 키가 남아 있어 `git config --global --unset user.emai` 로 제거합니다.
- 인증은 GitHub CLI(`gh auth git-credential`)가 처리해서 토큰이나 비밀번호가 설정 파일과 원격 URL 에 들어 있지 않습니다.

최종 `git config --list` (오타 키 제거 후):
```text
TODO(붙여넣기): git config --list 출력 (이메일은 마스킹)
```

**VSCode GitHub 로그인 · 저장소 연동**
```text
TODO(스크린샷 첨부): VSCode 에서 GitHub 로그인된 계정과 이 저장소(glocaledu/codyssey_1-1) 연동 화면 (토큰·인증 코드는 가리고 캡처)
```
로컬 설정의 `branch.main.vscode-merge-base=origin/main` 은 VSCode 가 이 저장소의 `main` 과 `origin/main` 을 연결해 둔 흔적입니다.

## 7) 학습 정리 (과제 목표 자가 점검)
- **절대경로 vs 상대경로**: 절대경로는 `/` 부터 시작해 위치와 무관하게 같은 대상을 가리키고, 상대경로는 현재 위치 기준입니다. (3장, 5-5 의 `/root` 경로 에러가 절대경로를 잘못 쓴 예)
- **파일 권한**: `rwx = 4+2+1`. 755 = `rwxr-xr-x`, 644 = `rw-r--r--`. 순서는 소유자 / 그룹 / 기타이고, 디렉토리의 `x` 는 진입 권한입니다.
- **Dockerfile 로 커스텀 이미지**: `nginx:alpine` 위에 `COPY`, `ENV`, `LABEL`, `HEALTHCHECK` 를 얹어 `my-web:1.0` 을 만들었습니다. (4-2)
- **포트 매핑이 필요한 이유**: 컨테이너는 격리된 네트워크를 쓰므로 내부 포트(80)는 호스트에서 바로 보이지 않습니다. `-p 8080:80` 으로 호스트 포트를 컨테이너 포트에 연결해야 접속할 수 있고, 같은 이미지를 호스트 포트만 바꿔 여러 개 띄울 수 있습니다. (4-3)
- **바인드 마운트 vs 볼륨**: 바인드 마운트는 호스트의 특정 폴더를 그대로 연결해 호스트 수정이 즉시 반영되고(4-4), 볼륨은 Docker 가 관리하는 저장소로 컨테이너를 삭제해도 데이터가 남습니다. (4-5)
- **Docker 볼륨(영속 데이터)**: 컨테이너 쓰기 레이어는 컨테이너와 함께 사라지지만 볼륨은 컨테이너 수명과 분리되어 있습니다. 대조군(볼륨 없음)에서는 데이터가 사라졌습니다. (4-5)
- **이미지 vs 컨테이너**: 이미지는 읽기 전용 설계도이고, 컨테이너는 그 위에 쓰기 레이어를 얹은 실행 인스턴스입니다. 같은 `my-web:1.0` 으로 컨테이너 여러 개를 띄웠습니다.
- **Git vs GitHub**: Git 은 내 PC 의 로컬 버전 관리 도구이고, GitHub 는 Git 저장소를 올려 원격 백업과 협업(PR, 이슈 등)을 하는 플랫폼입니다.

## 8) 보안 · 개인정보
- 로그와 문서에 토큰, 비밀번호, 개인키, 인증 코드를 포함하지 않았습니다. 원격 URL 에도 토큰이 없고, 인증은 `gh auth git-credential` 이 처리합니다.
- 이메일은 `g***@gmail.com` 으로 마스킹했습니다.
- 컨테이너 ID 와 이미지 해시는 민감정보가 아닌 로컬 식별자입니다.

## 9) 저장소 구성 · 재현 방법
```text
.
├── README.md          # 이 문서
├── Dockerfile         # nginx:alpine 기반 커스텀 이미지
├── site/index.html    # 정적 콘텐츠 (바인드 마운트 대상)
└── logs/              # 원본 수행 로그 (terminal.log, docker-ops.log, docker-volume.log 등)
```
(`practice/`, `README_backup.md` 는 `.gitignore` 로 제외)

**재현 순서** (Docker Desktop + WSL2 Ubuntu 기준)
```bash
# 1) 빌드 시점의 site/index.html 은 'v1 (이미지에 포함된 기본 페이지)' 입니다.
docker build -t my-web:1.0 .

# 2) 포트 매핑 2회
docker run -d -p 8080:80 --name my-web-8080 my-web:1.0
docker run -d -p 8081:80 -e APP_ENV=staging --name my-web-8081 my-web:1.0
curl -i http://localhost:8080
curl -i http://localhost:8081

# 3) 바인드 마운트: 호스트 파일만 수정해서 반영되는지 확인
docker run -d -p 8082:80 -v "$PWD/site:/usr/share/nginx/html" --name my-web-bind my-web:1.0
curl http://localhost:8082
echo '<h1>Codyssey Mission1</h1><p>version: v2 (호스트에서 수정, 재빌드 없음)</p>' > site/index.html
curl http://localhost:8082   # v2
curl http://localhost:8080   # v1 그대로

# 4) 볼륨 영속성
docker volume create mydata
docker run -d --name vol-test -v mydata:/data my-web:1.0
docker exec vol-test sh -c 'echo hi > /data/hello.txt'
docker rm -f vol-test
docker run -d --name vol-test2 -v mydata:/data my-web:1.0
docker exec vol-test2 cat /data/hello.txt   # hi
```
> 바인드 마운트 실습에서 `site/index.html` 을 v2 로 바꿨기 때문에, 처음부터 다시 빌드할 때는 파일 내용이 v2 일 수 있습니다. 위 4-2 의 v1 내용으로 되돌린 뒤 빌드하면 로그와 같은 결과가 나옵니다.
