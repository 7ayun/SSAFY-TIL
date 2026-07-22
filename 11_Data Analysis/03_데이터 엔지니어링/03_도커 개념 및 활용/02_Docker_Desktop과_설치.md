# [데이터 엔지니어링] Docker Desktop과 설치

---

## 1. Docker Desktop 설치

공식 사이트(`https://www.docker.com/products/docker-desktop/`)에서 설치 파일을 받아 설치를 진행한다.

설치 중 Configuration 화면에서 체크할 항목들이 있다.

| 옵션 | 설명 |
|---|---|
| Use WSL 2 instead of Hyper-V | **필수 체크.** Hyper-V보다 WSL2가 더 유연하고 안정적이며, Windows Home에서도 사용 가능 |
| Allow Windows Containers to be used with this installation | Windows 컨테이너를 사용할 경우에만 필요. 리눅스 컨테이너만 쓴다면 체크하지 않아도 됨 |
| Add shortcut to desktop | 바탕화면 아이콘 생성 여부. 개인 선택 사항 |

이후 Docker Subscription Service Agreement에서 **Accept**를 눌러 설치를 진행한다 (회사에서 상업적으로 사용할 경우의 구독 조건을 묻는 화면).

설치가 끝나면 "Welcome to Docker" 로그인 화면이 뜨는데, 지금 당장 계정이 필요한 건 아니므로 **Skip** 해도 무방하다. 필요하면 나중에 화면에서 회원가입을 진행하면 된다 (Plan은 개인 사용 기준 **Free**로 충분).

## 2. Docker Desktop과 WSL2(Ubuntu) 연동

Docker는 항상 **Docker Desktop이 켜져 있어야** 명령어가 정상 동작한다.

기존에 Linux 수업 등에서 설치해둔 Ubuntu 배포판과 Docker Desktop을 연동해야 터미널에서 도커 명령어를 사용할 수 있다.

- 경로: `Docker Desktop > Settings > Resources > WSL Integration`
- "Enable integration with my default WSL distro" 체크
- 기존에 설치한 Ubuntu 배포판(예: Ubuntu-22.04)을 토글로 활성화
- `Apply & restart` 클릭

## 3. 설치 확인 (hello-world 테스트)

연동이 끝나면 도커가 정상적으로 동작하는지 확인하기 위해 다음 명령어를 실행한다.

```bash
docker run hello-world
```

이때 로컬에 `hello-world` 이미지가 없으면, `docker run`이 알아서 `docker pull`을 먼저 수행해 이미지를 받아온 뒤 실행한다. 정상적으로 실행되면 "Hello from Docker!"라는 설치 완료 안내 메시지가 출력되고 컨테이너는 종료된다.

이 실행으로 컨테이너 하나가 (종료 상태로) 남게 되는데, 이는 뒤에서 컨테이너 목록 확인 시 다시 다룬다.

## 4. 실습 환경 준비하기

### 4-1. 실습 디렉토리 생성

```bash
mkdir python-docker-env
cd python-docker-env
```

### 4-2. requirements.txt 구성

도커 컨테이너 안에서 파이썬 활용 시 자주 쓰는 라이브러리를 미리 정의해둔다.

```
pandas
matplotlib
```

## 5. Dockerfile 작성

```dockerfile
# python-docker-env/Dockerfile
FROM python:3.10-slim

WORKDIR /workspace

COPY requirements.txt .
RUN pip install --upgrade pip && pip install -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

## 6. 자주 쓰이는 Dockerfile 문법

| 키워드 | 용도 | 예시 |
|---|---|---|
| FROM | 베이스 이미지 지정 (반드시 첫 줄). 이미지를 만드는 흐름의 시작점 | `FROM python:3.10-slim` → 최소한의 리눅스 환경에 파이썬이 이미 설치된 이미지 |
| WORKDIR | 컨테이너 내부에서 작업할 디렉토리 지정 | `WORKDIR /app` |
| COPY | 로컬 파일/디렉토리를 컨테이너로 복사 | `COPY . /app` |
| RUN | **이미지 빌드 시점**에 명령을 실행 | `RUN apt-get update` |
| CMD | **컨테이너 실행 시점**에 기본으로 실행되는 명령어 | `CMD ["python", "app.py"]` |
| ENV | 환경 변수 설정 | `ENV APP_ENV=prod` |
| EXPOSE | 컨테이너가 사용할 포트를 명시(실제로 포트를 여는 건 아니고, "이 포트를 쓸 것이다"라고 알려주는 역할) | `EXPOSE 8000` |

> RUN과 CMD를 헷갈리기 쉬운데, RUN은 이미지를 빌드할 때 한 번 실행되고 CMD는 컨테이너가 실행(런타임)될 때마다 실행된다는 차이를 기억해두자.

## 7. Docker 이미지 빌드

```bash
docker build -t python-lab .
```

- `docker build`: 도커 이미지를 빌드하는 명령어
- `-t python-lab`: 생성할 이미지에 `python-lab`이라는 이름(태그)을 붙임
- `.`(마지막 인자): 현재 디렉토리의 Dockerfile과 관련 파일을 기반으로 이미지를 제작한다는 의미

## 8. Docker Container 실행

```bash
docker run -it --name pycheck python-lab
```

- `docker run`: 도커 컨테이너를 실행하는 명령어
- `-it`: 인터랙티브 모드 + 터미널 연결(터미널로 직접 명령 입력 가능한 상태로 실행)
- `--name pycheck`: 컨테이너 이름을 `pycheck`로 지정 (이후 관리 시 편의성 확보)
- `python-lab`: 실행할 이미지 이름

## 9. 컨테이너 목록 확인

```bash
docker ps      # 실행 중인 컨테이너만 확인
docker ps -a   # 종료된 컨테이너까지 전부 확인 (-a: all)
```

`docker ps`만 치면 안 뜰 수 있는데, 이는 현재 완전히 실행 중인 컨테이너가 없기 때문이다. `-a` 옵션을 붙이면 컨테이너 ID, 이미지, 상태(Exited 포함)까지 전부 확인할 수 있다. 같은 정보는 Docker Desktop GUI의 Containers 탭에서도 확인 가능하다.

## 10. Docker Container 정리

도커에서 컨테이너는 실행 중인 상태에서 바로 삭제할 수 없다. **반드시 stop 후 rm 순서로 정리**해야 한다.

```bash
docker stop pycheck   # 실행 중인 컨테이너를 멈춤
docker rm pycheck     # 멈춘 컨테이너를 삭제
```

정리 후 `docker ps -a`로 다시 확인하면 해당 컨테이너가 목록에서 사라진 것을 확인할 수 있다.

## 11. 로컬 도커 이미지 확인 및 삭제

```bash
docker images   # 로컬에 저장된 이미지 목록 확인
```

태그에 보통 `latest`가 붙어 있는데, 이는 가장 최신 버전이라는 의미이며 직접 태그를 지정해 `v1`, `v2` 등으로 버전 관리도 가능하다.

컨테이너를 삭제했다고 해서 이미지가 함께 삭제되는 것은 아니므로, 이미지를 지우고 싶다면 별도로 삭제해야 한다.

```bash
docker rmi {이미지명}:{버전}
```

컨테이너 삭제 명령어는 `rm`, 이미지 삭제 명령어는 `rmi`라는 점에 유의한다.

---

## 💡 한 줄 요약
> Docker Desktop을 WSL2와 연동해 설치를 마치고, requirements.txt와 Dockerfile을 작성해 이미지를 빌드·실행한 뒤, 컨테이너는 stop→rm, 이미지는 rmi 순서로 정리하는 전체 흐름을 실습했다.

## ❓ 더 찾아볼 것
- Docker Desktop의 Resources 탭에서 CPU/메모리 리소스 제한 설정 방법
- python:3.10-slim과 python:3.10의 차이(slim 이미지의 장단점)
- .dockerignore 파일의 역할
