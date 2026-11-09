# Codyssey 입학연수 미션1 — 내 컴퓨터에 개발자용 '작업실' 꾸미기

## 0) 프로젝트 개요
터미널(CLI) · Docker · Git/GitHub 를 직접 손으로 세팅하고, **"내 PC에서만 되는 코드"가 아니라 누구나 같은 방식으로 실행·검증할 수 있는 환경**을 만드는 미션입니다.
이 저장소에는 (1) 터미널 조작 로그, (2) 권한 실습, (3) 직접 작성한 Dockerfile + 웹 서버 소스, (4) 포트 매핑 · 바인드 마운트 · 볼륨 영속성 검증 로그, (5) Git 설정 기록을 모두 담았습니다.

> ⚠️ **먼저 밝히는 수행 범위 (정직한 기록)**
> 이번 수행은 **Docker Hub / GitHub 등 외부 네트워크가 차단된 격리 리눅스 환경**에서 진행했습니다.
> 그래서 아래 항목은 **수행하지 못했거나 대체 방법을 썼고**, 체크리스트와 5) 트러블슈팅에 사유를 적었습니다.
> - `hello-world`, `ubuntu`, `nginx` 이미지 pull → `Forbidden` 으로 불가 → **대체 실습으로 진행**
> - VSCode GitHub 로그인 / 저장소 push / SSH 키 등록 → 계정 로그인이 필요해 **미수행**
> - 브라우저 접속 화면 스크린샷 → 브라우저가 없어 **curl 응답(과제에서 허용한 대체 증거)** 으로 제출

---

## 1) 실행 환경
| 항목 | 값 |
|---|---|
| OS | Ubuntu 24.04.5 LTS (x86_64) |
| Shell | GNU bash 5.2.21 |
| Docker | Docker Engine 29.9.2 (Compose v5.7.0 포함) |
| Git | 2.43.0 |
| 터미널 | 샌드박스 bash (CLI 전용) |

```text
Docker version 29.9.2, build a82e528
git version 2.43.0
```

> **재현 시 주의 (PC 의존 사항)**: 서울 캠퍼스처럼 `sudo`가 제한된 환경에서는 OrbStack 사용을 권장합니다(과제 안내). 본 수행은 root 권한 환경이라 Docker 데몬을 `dockerd &` 로 직접 띄웠습니다. 일반 PC에서는 Docker Desktop/OrbStack 실행으로 대체하세요.

## 2) 수행 체크리스트
- [x] 터미널 기본 조작 (pwd / ls -la 숨김파일 / mkdir / cd / cp / mv / rm / cat / touch)
- [x] 절대경로 vs 상대경로 실습
- [x] 권한 변경 실습 (파일 1개 + 디렉토리 1개, 전/후 비교)
- [x] Docker 설치 점검 (`docker --version`, `docker info`)
- [x] 이미지/컨테이너 운영 명령 (`images`, `ps`, `ps -a`, `logs`, `stats`, `stop`)
- [ ] `hello-world` 이미지 실행 → **네트워크 차단으로 미수행** (대체: 로컬 `mini-linux` 컨테이너 실행)
- [~] `ubuntu` 컨테이너 진입 → **대체**: 호스트 bash/ls로 만든 `mini-linux:local` 에서 `exec` / `run` 실습
- [x] 직접 작성한 Dockerfile 로 커스텀 이미지 빌드/실행 (`FROM scratch` 방식)
- [x] 포트 매핑 접속 2회 (8080, 8081)
- [x] 바인드 마운트 반영 (호스트 수정 → 재빌드 없이 반영)
- [x] 볼륨 영속성 (컨테이너 삭제 전/후 + 볼륨 없는 대조군)
- [x] Git 사용자 정보 · 기본 브랜치(main) 설정, `git config --local --list` 기록
- [ ] VSCode GitHub 로그인 / 원격 저장소 연동 / push → **미수행** (계정 인증 필요)
- [ ] (보너스) Compose / SSH 키 → 미수행
- [x] 민감정보(토큰·비밀번호) 미포함 확인

## 3) 터미널 조작 · 권한 실습 로그
홈 디렉토리 아래 `~/codyssey/practice` 에서 실습했습니다. (`HOME=/root`)

```bash
$ pwd
/home/claude/codyssey-mission1
$ mkdir -p ~/codyssey/practice/docs && cd ~/codyssey/practice
$ touch empty.txt .hidden_file
$ echo "hello codyssey" > docs/note.txt
$ ls -la
total 12
drwxr-xr-x 3 root root 4096 Nov  9 16:10 .
drwxr-xr-x 3 root root 4096 Nov  9 16:10 ..
-rw-r--r-- 1 root root    0 Nov  9 16:10 .hidden_file
drwxr-xr-x 2 root root 4096 Nov  9 16:10 docs
-rw-r--r-- 1 root root    0 Nov  9 16:10 empty.txt
$ cat docs/note.txt
hello codyssey
$ cp docs/note.txt docs/note_copy.txt
$ mv docs/note_copy.txt docs/renamed.txt
$ ls -la docs
total 16
drwxr-xr-x 2 root root 4096 Nov  9 16:10 .
drwxr-xr-x 3 root root 4096 Nov  9 16:10 ..
-rw-r--r-- 1 root root   15 Nov  9 16:10 note.txt
-rw-r--r-- 1 root root   15 Nov  9 16:10 renamed.txt
$ rm docs/renamed.txt && ls docs
note.txt
--- 절대경로 vs 상대경로
$ cd docs && pwd && cat /root/codyssey/practice/docs/note.txt && cat note.txt && cat ../docs/note.txt
/root/codyssey/practice/docs
hello codyssey
hello codyssey
hello codyssey
--- 권한 실습 (파일)
$ ls -l docs/note.txt
-rw-r--r-- 1 root root 15 Nov  9 16:10 docs/note.txt
$ chmod 600 docs/note.txt && ls -l docs/note.txt
-rw------- 1 root root 15 Nov  9 16:10 docs/note.txt
$ chmod 755 docs/note.txt && ls -l docs/note.txt
-rwxr-xr-x 1 root root 15 Nov  9 16:10 docs/note.txt
$ chmod 644 docs/note.txt && ls -l docs/note.txt
-rw-r--r-- 1 root root 15 Nov  9 16:10 docs/note.txt
--- 권한 실습 (디렉토리)
$ ls -ld docs
drwxr-xr-x 2 root root 4096 Nov  9 16:10 docs
$ chmod 700 docs && ls -ld docs
drwx------ 2 root root 4096 Nov  9 16:10 docs
$ chmod 755 docs && ls -ld docs
drwxr-xr-x 2 root root 4096 Nov  9 16:10 docs
```

**관찰 정리**
- 절대경로는 `/`(루트)부터 시작해 어디서 실행해도 같은 위치를 가리키고, 상대경로는 **현재 위치(pwd)** 기준이라 `../docs/note.txt` 처럼 위치에 따라 의미가 달라집니다.
- 권한 `rwx = 4+2+1`. 755 = `rwxr-xr-x`(소유자 7=rwx, 그룹 5=r-x, 기타 5=r-x), 644 = `rw-r--r--`, 600 = `rw-------`.
- 디렉토리의 `x` 는 "그 안으로 들어가고(cd) 이름으로 접근할 수 있는 권한"이라 디렉토리는 보통 755 입니다.

## 4) Docker 운영 · 검증 로그

### 4-1) 설치 · 데몬 점검
처음에는 데몬이 꺼져 있어 `docker info` 의 Server 항목이 에러였고, `dockerd` 를 띄운 뒤 정상 확인했습니다. (5-1 참고)

```text
Client: Docker Engine - Community
 Version:    29.9.2
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.38.1
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v5.7.0
    Path:     /usr/libexec/docker/cli-plugins/docker-compose

Server:
 Containers: 3
  Running: 2
  Paused: 0
  Stopped: 1
 Images: 2
 Server Version: 29.9.2
 Storage Driver: overlayfs
  driver-type: io.containerd.snapshotter.v1
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 CDI spec directories:
  /etc/cdi
  /var/run/cdi
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b7a25261e029d68978bbf290df0a62fc819b3c51
 runc version: v1.5.2-0-g29dd3dc2
 init version: de40ad0
 Security Options:
  seccomp
   Profile: builtin
 Kernel Version: 6.18.44-fc-v310
 Operating System: Ubuntu 24.04.5 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 7.835GiB
 Name: vm
 ID: 61437e58-104a-4de6-b843-740c36e9d5f1
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 HTTPS Proxy: http://127.0.0.1:40279
 No Proxy: localhost,127.0.0.1,::1,127.0.0.0/8,0.0.0.0/8,::,169.254.0.0/16,api.anthropic.com,api-staging.anthropic.com,api-pr-preview.anthropic.com,mcp-proxy.anthropic.com,mcp-proxy-staging.anthropic.com,registry.npmjs.org,jsr.io,npm.jsr.io,pypi.org,files.pythonhosted.org,index.crates.io,proxy.golang.org,host.docker.internal,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,100.64.0.0/10,.svc.cluster.local,*.svc.cluster.local
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
  ::1/128
 Live Restore Enabled: false
 Firewall Backend: iptables
  EnableUserlandProxy: true
  UserlandProxyPath: /usr/bin/docker-proxy

WARNING: Support for cgroup v1 is deprecated and planned to be removed by no later than May 2029 (https://github.com/moby/moby/issues/51111)
```

### 4-2) 직접 작성한 웹 서버 · Dockerfile
`app/server.c` (정적 C 웹 서버), `site/index.html`, `data/.keep`, `Dockerfile` 을 직접 작성했습니다.

**선택한 방식**: 과제의 (B) 유형에 해당 — 기존 베이스(`scratch`)에 기능을 얹은 커스텀 이미지.
> 원래는 `nginx:alpine` 을 쓰고 싶었으나 레지스트리가 차단되어, **베이스가 가장 단순한 `scratch`** 를 택하고 호스트에서 정적 빌드한 바이너리를 넣었습니다.

```dockerfile
# 베이스: scratch (빈 이미지). Docker Hub 접근이 막힌 환경이라 호스트에서 정적 빌드한 바이너리만 담는다.
FROM scratch
LABEL org.opencontainers.image.title="codyssey-mini-web"
ENV APP_ENV=dev PORT=5000
COPY app/server /server
COPY site/ /www/
# /data 디렉토리를 이미지에 만들어 둔다 (scratch에는 mkdir가 없어 COPY로 생성)
COPY data/ /data/
EXPOSE 5000
ENTRYPOINT ["/server"]
```

| 커스텀 포인트 | 목적 |
|---|---|
| `FROM scratch` | OS 없이 바이너리만 담아 이미지를 1MB대로 최소화, 공격면 축소 |
| `LABEL` | 이미지 메타데이터(이름) 표기 |
| `ENV APP_ENV=dev PORT=5000` | 설정과 코드 분리 — 실행 시 `-e` 로 덮어쓰기 가능 |
| `COPY app/server /server` | 직접 작성한 웹 서버 실행파일 |
| `COPY site/ /www/` | 정적 콘텐츠 (바인드 마운트로 교체 실험 대상) |
| `COPY data/ /data/` | 볼륨을 연결할 디렉토리 생성 (scratch는 `mkdir` 불가) |
| `EXPOSE 5000` / `ENTRYPOINT` | 문서화용 포트 선언 / 컨테이너 시작 시 서버 실행 |

서버 소스 핵심 동작: `GET /` → `/www/index.html` 을 **요청마다 읽어** 응답(바인드 마운트 변경이 즉시 반영됨), `GET /hits` → `/data/hits.log` 줄 수 반환(볼륨 영속성 확인용).

빌드 방법(재현): 
```bash
gcc -O2 -static -o app/server app/server.c   # 호스트에서 정적 빌드
docker build -t my-web:1.0 .
```

### 4-3) 빌드 · 실행 · 포트 매핑 2회 · logs · ps
컨테이너 내부 포트는 5000 으로 같고, 호스트 포트만 8080 / 8081 로 달리 매핑했습니다. (8081 은 `-e APP_ENV=staging` 도 주입)

```bash
$ docker build -t my-web:1.0 .
#5 [2/2] COPY site/ /www/
#5 DONE 0.0s

#6 exporting to image
#6 exporting layers 0.1s done
#6 exporting manifest sha256:1035263e1b02dcb17935497dcef3dcbdb2b1efad2c48e4dcc42093542bbb030e done
#6 exporting config sha256:68f53c819fdd252eb3006ea2a58a8b8ef4bf7ceea26a7fdef994f25c9cfa6825 done
#6 exporting attestation manifest sha256:842f4a7e399c53d0c0e8a7d3a588146e37dbc63e0fd9b737aac63465d54907bd done
#6 exporting manifest list sha256:d2b014ab32c44453e02cab0807031e666df2853bcf12927f8835f322480d8564 done
#6 naming to docker.io/library/my-web:1.0 done
#6 unpacking to docker.io/library/my-web:1.0 0.0s done
#6 DONE 0.2s
$ docker images
IMAGE        ID             DISK USAGE   CONTENT SIZE   EXTRA
my-web:1.0   d2b014ab32c4       1.22MB          372kB        
$ docker run -d -p 8080:5000 --name my-web-8080 my-web:1.0
ba4f7b306358d7fc2fd253efe27881af60790db6f3614c09458e7fdb0dffa755
$ docker run -d -p 8081:5000 -e APP_ENV=staging --name my-web-8081 my-web:1.0
907136c0c81c4e0ffe82a73c810781cf3c8ceda019985d59f9f7a75bc3a0ca47
$ docker ps
CONTAINER ID   IMAGE        COMMAND     CREATED         STATUS         PORTS                    NAMES
907136c0c81c   my-web:1.0   "/server"   2 seconds ago   Up 2 seconds   0.0.0.0:8081->5000/tcp   my-web-8081
ba4f7b306358   my-web:1.0   "/server"   2 seconds ago   Up 2 seconds   0.0.0.0:8080->5000/tcp   my-web-8080
$ curl -i http://localhost:8080
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 88
Connection: close

<h1>Codyssey Mission1</h1>
<p>version: v1 (image에 구워 넣은 기본 페이지)</p>
$ curl http://localhost:8081
<h1>Codyssey Mission1</h1>
<p>version: v1 (image에 구워 넣은 기본 페이지)</p>
$ docker logs my-web-8080
server start: port=5000 APP_ENV=dev
request: GET / HTTP/1.1
$ docker logs my-web-8081
server start: port=5000 APP_ENV=staging
request: GET / HTTP/1.1
```

> 브라우저 대신 `curl -i` 로 **HTTP 200 응답과 본문**을 증거로 남겼습니다. (과제에서 curl 응답 허용)

### 4-4) 바인드 마운트 · 운영 명령(ps -a / stats / stop / exec)
호스트의 `site/` 를 컨테이너 `/www` 에 마운트한 뒤 호스트에서 파일만 수정했습니다. **재빌드·재시작 없이** 응답이 v1 → v2 로 바뀌었고, 마운트하지 않은 컨테이너(8080)는 그대로 v1 입니다.

```bash
### 바인드 마운트
$ docker run -d -p 8082:5000 -v "$PWD/site:/www" --name my-web-bind my-web:1.0
39c8e7f2d3b6cd4cdeb478c11c57164803fc0db59d2ca671e435032dfff22c00
$ curl -s http://localhost:8082   # 변경 전
<h1>Codyssey Mission1</h1>
<p>version: v1 (image에 구워 넣은 기본 페이지)</p>
$ echo "<h1>Codyssey Mission1</h1><p>version: v2 (호스트에서 수정, 재빌드 없음)</p>" > site/index.html
$ curl -s http://localhost:8082   # 변경 후
<h1>Codyssey Mission1</h1><p>version: v2 (호스트에서 수정, 재빌드 없음)</p>
$ curl -s http://localhost:8080   # 마운트 없는 컨테이너는 그대로
<h1>Codyssey Mission1</h1>
<p>version: v1 (image에 구워 넣은 기본 페이지)</p>

### 운영 명령
$ docker ps -a
CONTAINER ID   IMAGE        COMMAND     CREATED          STATUS          PORTS                    NAMES
e50b96201d55   my-web:1.0   "/server"   1 second ago     Up 1 second     0.0.0.0:8085->5000/tcp   novol
1cd6cd457e05   my-web:1.0   "/server"   3 seconds ago    Up 3 seconds    0.0.0.0:8084->5000/tcp   vol-test2
39c8e7f2d3b6   my-web:1.0   "/server"   6 seconds ago    Up 6 seconds    0.0.0.0:8082->5000/tcp   my-web-bind
907136c0c81c   my-web:1.0   "/server"   19 seconds ago   Up 19 seconds   0.0.0.0:8081->5000/tcp   my-web-8081
ba4f7b306358   my-web:1.0   "/server"   19 seconds ago   Up 19 seconds   0.0.0.0:8080->5000/tcp   my-web-8080
$ docker stats --no-stream
CONTAINER ID   NAME          CPU %     MEM USAGE / LIMIT   MEM %     NET I/O           BLOCK I/O   PIDS
e50b96201d55   novol         0.00%     444KiB / 7.835GiB   0.01%     527B / 662B       0B / 0B     1
1cd6cd457e05   vol-test2     0.00%     304KiB / 7.835GiB   0.00%     821B / 662B       0B / 0B     1
39c8e7f2d3b6   my-web-bind   0.00%     304KiB / 7.835GiB   0.00%     1.63kB / 1.39kB   0B / 0B     1
907136c0c81c   my-web-8081   0.00%     304KiB / 7.835GiB   0.00%     1.45kB / 800B     0B / 0B     1
ba4f7b306358   my-web-8080   0.00%     656KiB / 7.835GiB   0.01%     1.97kB / 1.39kB   0B / 0B     1
$ docker stop my-web-8081 && docker ps -a --filter name=my-web-8081
CONTAINER ID   IMAGE        COMMAND     CREATED          STATUS                                PORTS     NAMES
907136c0c81c   my-web:1.0   "/server"   32 seconds ago   Exited (137) Less than a second ago             my-web-8081
$ docker exec -it my-web-8080 sh
OCI runtime exec failed: exec failed: unable to start container process: exec: "sh": executable file not found in $PATH
```

### 4-5) 볼륨 영속성 검증 (대조군 포함)
- **볼륨 없음**: 요청 4회 기록 후 컨테이너를 지우고 다시 만들자 `hits=4 → hits=1` (데이터 유실)
- **볼륨 `mydata` 연결**: 컨테이너를 지우고 새 컨테이너에 같은 볼륨을 연결하자 `hits=4 → hits=5` (데이터 유지)

```bash
$ docker build -t my-web:1.0 .
#6 DONE 0.0s
#7 naming to docker.io/library/my-web:1.0 done
#7 DONE 0.1s

### [대조군] 볼륨 없이: 기록 후 컨테이너 삭제/재생성
$ curl x3 ; curl /hits
hits=4
$ docker rm -f novol && docker run -d -p 8085:5000 --name novol my-web:1.0 && curl /hits
hits=1

### [볼륨] 기록 후 컨테이너 삭제/재생성
$ docker volume create mydata
mydata
$ docker run -d -p 8083:5000 -v mydata:/data --name vol-test my-web:1.0 ; curl x3 ; curl /hits
hits=4
$ docker rm -f vol-test
vol-test
$ docker volume ls
DRIVER    VOLUME NAME
local     mydata
$ docker run -d -p 8084:5000 -v mydata:/data --name vol-test2 my-web:1.0 ; curl /hits
hits=5
```

### 4-6) 컨테이너 실습 · attach / exec 차이 정리
`hello-world`, `ubuntu` 는 pull 이 불가해 호스트의 `bash`, `ls`, `cat`, `sleep` 과 필요한 라이브러리만 모아 `docker import` 로 만든 **임시 이미지 `mini-linux:local`** 로 대체 실습했습니다. (진짜 ubuntu 이미지는 아님)

```bash
### hello-world/ubuntu 대체 실습: 호스트 bash/ls/echo로 만든 임시 이미지 mini-linux:local
$ docker images
IMAGE              ID             DISK USAGE   CONTENT SIZE   EXTRA
mini-linux:local   4a5ef30b991b        7.4MB         2.33MB        
my-web:1.0         39b7e1226f66       1.22MB          372kB   U    
$ docker run --rm mini-linux:local /bin/echo "hello from container"
docker: Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: exec: "/bin/echo": stat /bin/echo: no such file or directory

Run 'docker run --help' for more information
$ docker run --rm mini-linux:local /bin/ls /
bin
dev
etc
lib
lib64
proc
sys
tmp

### exec vs run (attach)
$ docker run -d --name mini mini-linux:local /bin/sleep 300
bcc2191670c0f0b014d9d6027efb34c0a42f170467e73a0ef8ee446078c38bbd
$ docker exec mini /bin/ls /bin   # 실행 중 컨테이너에 새 프로세스로 진입
bash
cat
ls
sleep
$ docker exec mini /bin/bash -c "echo hi > /tmp/a.txt; cat /tmp/a.txt"
hi
$ docker run -i --rm --name mini2 mini-linux:local /bin/bash -c "echo 메인 프로세스가 bash; ls /"   # run: 이 프로세스가 곧 컨테이너
메인 프로세스가 bash
bin
dev
etc
lib
lib64
proc
sys
tmp
$ docker ps -a --filter name=mini
CONTAINER ID   IMAGE              COMMAND            CREATED        STATUS                  PORTS     NAMES
bcc2191670c0   mini-linux:local   "/bin/sleep 300"   1 second ago   Up Less than a second             mini
$ docker stop mini && docker ps -a --filter name=mini
CONTAINER ID   IMAGE              COMMAND            CREATED          STATUS                                PORTS     NAMES
bcc2191670c0   mini-linux:local   "/bin/sleep 300"   11 seconds ago   Exited (137) Less than a second ago             mini
$ docker logs --tail 3 my-web-8080 (이미 삭제됨 -> novol 사용)
server start: port=5000 APP_ENV=dev
request: GET /hits HTTP/1.1
```

| 구분 | `docker run` | `docker exec` | `attach` |
|---|---|---|---|
| 하는 일 | 새 컨테이너 생성 + 시작 | **이미 실행 중인** 컨테이너 안에서 새 프로세스 실행 | 컨테이너의 **메인 프로세스** 입출력에 붙음 |
| 종료 영향 | 메인 프로세스가 끝나면 컨테이너 종료 | 내 프로세스가 끝나도 컨테이너는 유지 | `Ctrl+C` 가 메인 프로세스에 전달될 수 있어 주의 |

> `attach` 는 이번 환경에서 직접 실행하지 못해 개념만 정리했습니다. `scratch` 이미지에서는 `docker exec my-web sh` 가 "executable file not found" 로 실패했습니다 — 이미지 안에 셸이 아예 없기 때문입니다.

## 5) 트러블슈팅

### 5-1) `docker info` 가 데몬에 연결하지 못함
- **문제**: `failed to connect to the docker API at unix:///var/run/docker.sock`
- **원인 가설**: 클라이언트(CLI)만 있고 데몬(dockerd)이 실행되지 않음
- **확인**: `docker info` 에서 Client 항목만 출력, Server 항목 에러 → `which dockerd` 로 바이너리 존재 확인
- **해결**: `dockerd > /tmp/dockerd.log 2>&1 &` 로 기동 후 로그의 `Daemon has completed initialization` 확인, `docker info` 정상

### 5-2) 이미지 pull 이 `Forbidden`
- **문제**: `docker pull hello-world` → `Head "https://registry-1.docker.io/...": Forbidden`
- **원인 가설**: 이 환경은 외부 접속이 허용 목록(allowlist)으로 제한됨
- **확인**: 다른 경로로 우회하지 않고 `docker images` 가 비어 있음을 확인
- **대안**: ① 호스트에서 `gcc -static` 으로 만든 바이너리 + `FROM scratch` 로 커스텀 이미지 작성 ② `docker import` 로 임시 리눅스 이미지 작성. 일반 PC에서는 `docker pull hello-world` 로 그대로 수행 가능

### 5-3) 대조군 볼륨 테스트에서 `hits=0` (실험 설계 오류)
- **문제**: 볼륨 없는 컨테이너가 계속 `hits=0` → "유실"처럼 보였지만 사실은 잘못된 비교
- **원인 가설**: `scratch` 이미지에는 `/data` 디렉토리가 없어 기록 자체가 실패
- **확인**: 볼륨 연결 시에만 기록되는 점 + 이미지 안 파일 구조 확인
- **해결**: `COPY data/ /data/` 로 디렉토리를 이미지에 포함한 뒤 재실험 → `4 → 1`(유실) vs `4 → 5`(유지) 로 올바른 대조 확인

### 5-4) 임시 이미지에서 `/bin/echo` 실행 실패
- **문제**: `exec: "/bin/echo": stat /bin/echo: no such file or directory`
- **원인 가설**: bash 에서 `echo` 가 셸 내장 명령이라 파일로 복사되지 않음
- **확인**: 같은 이미지에서 `bash -c "echo ..."` 는 정상 동작
- **해결**: 컨테이너 안에서 `echo` 는 `/bin/bash -c "echo ..."` 로 호출

## 6) Git 설정
```bash
$ git init -b main
Initialized empty Git repository in /home/claude/codyssey-mission1/.git/
$ git config user.name "<교육생 이름>"
$ git config user.email "<교육생 GitHub 이메일>"
$ git config init.defaultBranch main
$ git config --local --list
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
user.name=<교육생 이름>
user.email=<교육생 GitHub 이메일>
init.defaultbranch=main
```
- 기본 브랜치를 `main` 으로 설정했습니다.
- ⚠️ `user.name` / `user.email` 은 **실제 본인 정보로 바꿔야 하는 자리표시자**입니다. (이 수행 환경에서는 본인 계정 정보가 없어 임의 값을 넣지 않았습니다.)
- **VSCode GitHub 로그인 · 원격 저장소 연동 · push 는 미수행**입니다. 실제 PC에서는 `git remote add origin <저장소 URL>` → `git push -u origin main` 순서로 진행하고, 토큰·비밀번호는 화면/로그에 노출되지 않도록 가리고 캡처합니다.

## 7) 학습 정리 (과제 목표 자가 점검)
- **포트 매핑이 필요한 이유**: 컨테이너는 격리된 네트워크 네임스페이스를 쓰므로, 내부 포트(5000)는 호스트에서 보이지 않습니다. `-p 8080:5000` 으로 호스트 포트를 컨테이너 포트에 연결해야 외부에서 접속할 수 있고, 같은 이미지를 호스트 포트만 바꿔 여러 개 띄울 수 있습니다.
- **Docker 볼륨**: 컨테이너 쓰기 레이어는 컨테이너와 함께 사라지지만, 볼륨은 컨테이너 수명과 분리된 저장소라 삭제 후에도 데이터가 남습니다. (4-5 로그로 확인)
- **바인드 마운트 vs 볼륨**: 바인드 마운트는 호스트의 특정 경로를 그대로 연결(개발 중 코드 즉시 반영에 유리), 볼륨은 Docker가 관리하는 영속 저장소(DB 데이터 등에 유리).
- **이미지 vs 컨테이너**: 이미지는 읽기 전용 설계도, 컨테이너는 그 위에 쓰기 레이어를 얹어 실행한 인스턴스입니다. 같은 이미지(`my-web:1.0`)로 컨테이너 여러 개를 띄운 것으로 확인했습니다.
- **Git vs GitHub**: Git은 내 PC의 **로컬 버전 관리 도구**, GitHub는 Git 저장소를 올려 **원격 백업·협업**(PR, 이슈 등)을 하는 플랫폼입니다.

## 8) 보안 · 개인정보
- 로그/문서에 토큰, 비밀번호, 개인키, 인증 코드는 **포함하지 않았습니다.**
- 컨테이너 ID 해시 등은 민감정보가 아닌 로컬 식별자입니다.

## 9) 저장소 구성
```text
.
├── README.md          # 이 문서
├── Dockerfile
├── app/server.c       # 직접 작성한 웹 서버 소스 (바이너리는 .gitignore)
├── site/index.html    # 정적 콘텐츠 (바인드 마운트 대상)
├── data/.keep         # /data 디렉토리 생성용
└── logs/              # 원본 수행 로그
```