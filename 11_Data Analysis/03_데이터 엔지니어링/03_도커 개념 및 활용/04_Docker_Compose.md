# [데이터 엔지니어링] Docker Compose

---

## 1. Docker Compose란?

지금까지는 하나의 컨테이너를 `docker run`으로 실행하는 것만 다뤘다. 하지만 실제 서비스는 웹서버, DB, 캐시(Redis) 등 여러 요소가 함께 돌아가야 한다. 이걸 `docker run`으로만 처리하면 명령어를 여러 개 계속 실행해야 하고, 실행 순서와 설정도 매번 맞춰줘야 하는 번거로움이 생긴다.

**Docker Compose**는 이런 문제를 해결하기 위해 나온 도구로, 여러 컨테이너를 하나의 프로젝트처럼 관리할 수 있게 해준다.

- `docker-compose.yml` 파일에 컨테이너 구성과 관계(어떤 이미지를 쓸지, 포트는 어떻게 연결할지, 컨테이너끼리 어떻게 통신할지 등)를 미리 정의
- 명령어 한 줄(`docker compose up`)로 여러 컨테이너를 동시에 실행

## 2. docker-compose.yml 기본 구조

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  db:
    image: mongo
```

- `services`: 컨테이너 목록을 정의하는 최상위 키
- `image`: 사용할 도커 이미지 지정
- `ports`: 호스트와 컨테이너의 포트 매핑. `"8080:80"`은 **호스트에서 8080번으로 접속했을 때, 실제로는 컨테이너 내부의 80번 포트로 연결된다**는 의미다. 호스트(컨테이너를 띄우는 주체)와 컨테이너는 서로 다른 존재이기 때문에 포트를 매핑해줘야 한다. (참고: HTTP의 기본 포트가 80번이라, 대부분의 웹서버는 컨테이너 내부에서 80번을 기본으로 사용한다)

## 3. Docker Compose 실습

`compose-lab` 디렉토리를 만들고 그 안에 `docker-compose.yml`과 `index.html`을 작성한다.

```bash
mkdir compose-lab
cd compose-lab
touch docker-compose.yml
touch index.html
```

**index.html** (nginx 컨테이너에서 서비스될 간단한 확인용 페이지)

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head><title>Docker Compose Web</title></head>
<body><h1>Hello from Nginx!</h1></body>
</html>
```

**docker-compose.yml**

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./index.html:/usr/share/nginx/html/index.html
  db:
    image: mongo
    ports:
      - "27017:27017"
```

- `web` 서비스는 nginx 이미지를 사용하며, 호스트의 8080 포트를 컨테이너의 80 포트와 연결한다.
- `volumes`에서 현재 경로의 `index.html`을 컨테이너 내부의 nginx 웹 루트 경로로 연결(마운트)한다. 이렇게 하면 로컬 파일과 컨테이너 내부 파일이 동기화되어, 우리가 만든 파일이 실제로 컨테이너 안에서 서비스되는 화면이 된다.
- `db` 서비스는 실습에서 직접 사용하지는 않지만, 두 개의 컨테이너가 동시에 뜨는 것을 보여주기 위해 mongo 이미지를 추가로 구성했다.

### 실행

```bash
docker compose up -d
```

- `docker compose`: Compose 명령어 실행
- `up`: 정의된 모든 서비스(컨테이너)를 생성하고 실행
- `-d`: detached 모드(백그라운드 실행)

처음 실행하면 이미지들을 pull 받아오는 과정이 보이고, 이 컴포즈 프로젝트만의 네트워크가 자동으로 생성된다. 컨테이너 이름을 따로 지정하지 않았기 때문에 `compose-lab-web-1`, `compose-lab-db-1`처럼 프로젝트명과 서비스명 기반으로 자동 네이밍된다.

### 상태 확인

```bash
docker compose ps
```

`docker compose ps`로 현재 프로젝트에서 실행 중인 컨테이너 목록을 확인할 수 있다. 이후 브라우저에서 `localhost:8080`으로 접속하면 "Hello from Nginx!" 문구가 정상적으로 출력되는 것을 확인할 수 있다 (포트 매핑이 잘 동작하고 있다는 의미).

> 참고: WSL2 환경에서 Docker를 사용하고 있어도, WSL2가 Docker Desktop과 네트워크를 공유하기 때문에 별도 설정 없이 Windows 브라우저의 `localhost`로 그대로 접속이 가능하다.

### 정리

```bash
docker compose down
```

- `down`: 정의된 모든 서비스(컨테이너)와 네트워크를 종료하고 정리

정리 후 `docker compose ps`로 확인하면 컨테이너가 아무것도 표시되지 않아야 정상이며, `localhost:8080`으로 다시 접속하면 "사이트에 연결할 수 없음" 상태가 되는 것까지 확인하면 정리가 잘 되었다는 것을 이중으로 체크할 수 있다.

## 4. 자주 쓰이는 docker-compose.yml 문법

| 키워드 | 용도 | 예시 |
|---|---|---|
| `services` | 실행할 컨테이너(서비스)들을 정의하는 최상위 키. 각 서비스별로 이미지, 포트, 볼륨 등을 설정 | `services: web: ...` |
| `image` | 사용할 도커 이미지를 지정 | `image: nginx:latest` |
| `build` | 이미지를 그대로 받아오는 대신, 로컬의 Dockerfile로 이미지를 직접 빌드할 때 사용 | `build: ./app` |
| `container_name` | 컨테이너 이름을 직접 지정 (지정 안 하면 자동 생성) | `container_name: my_app` |
| `ports` | 호스트 ↔ 컨테이너 포트 연결. `"호스트:컨테이너"` 형식 | `ports: - "8080:80"` |
| `volumes` | 데이터 공유/저장. 로컬 디렉토리를 컨테이너에 마운트하거나 볼륨 이름을 지정 | `volumes: - ./data:/var/lib/mysql` |
| `environment` | 컨테이너 내부의 환경 변수를 직접 지정 | `environment: - MYSQL_ROOT_PASSWORD=1234` |
| `env_file` | `.env` 파일에서 환경 변수를 한꺼번에 가져오기. 민감한 값은 직접 노출하는 것보다 이 방식이 관리에 유리 | `env_file: - .env` |
| `command` | 컨테이너 시작 시 실행할 명령어. Dockerfile의 CMD보다 우선순위가 높아 이를 덮어씀 | `command: python app.py` |

**상위 레벨 키워드** (services와 같은 레벨에 위치)

| 키워드 | 용도 | 예시 |
|---|---|---|
| `volumes` (상위) | 여러 서비스가 공유하는 데이터 볼륨을 정의. 예를 들어 노드 3개가 하나의 데이터를 공통으로 바라봐야 할 때 사용. 서비스 내부의 volumes처럼 실제 로컬 폴더 경로가 아니라, 도커 내부 관리 경로에 저장됨 | `volumes: db_data:` |
| `networks` | 사용자 정의 네트워크 생성 (컨테이너 간 별도의 통신망이 필요할 때) | `networks: my_network:` |

## 5. 자주 쓰이는 Docker Compose 명령어

| 명령어 | 용도 |
|---|---|
| `docker compose up` | yml에 정의된 컨테이너를 생성·시작 (`-d`: 백그라운드 실행) |
| `docker compose down` | 실행 중인 모든 컨테이너, 네트워크, 볼륨을 중단 및 삭제 |
| `docker compose ps` | 현재 Compose로 실행 중인 컨테이너 상태 확인 |
| `docker compose logs` | 컨테이너 로그 출력 (`-f`: 실시간 로그) |
| `docker compose stop` / `start` / `restart` | 컨테이너 중지 / 재시작 / 재시작 |
| `docker compose build` | Dockerfile 기반으로 이미지 빌드 |
| `docker compose exec <서비스명> <명령>` | 실행 중인 컨테이너에서 명령어 실행 (예: `docker compose exec web bash`) |
| `docker compose run --rm web sh` | 새로운 일회성 컨테이너 실행 |

## 6. Docker Compose 네트워크

- **같은 Compose 파일 내 네트워크**: `docker compose up`을 실행하면 자동으로 프로젝트 이름 기반의 네트워크가 생성된다. 같은 `docker-compose.yml` 안의 서비스들은 이 네트워크를 공유하므로 별도 설정 없이 서로 통신할 수 있다. 이때 서비스에 접근할 때는 **IP 대신 서비스명으로 접근하는 것이 권장**된다. IP는 컨테이너를 재시작할 때마다 바뀔 수 있지만, 도커가 내부적으로 DNS를 자동 관리하기 때문에 서비스명(예: `db`)으로 접근하는 게 훨씬 안정적이고 안전하다 (예: `db:27017`).
- **다른 Compose 프로젝트 간 네트워크**: 기본적으로 서로 다른 Compose 프로젝트끼리는 네트워크가 분리되어 있어 통신할 수 없다. 만약 통신이 필요하다면, 외부에 브릿지 네트워크를 미리 하나 만들어두고 각각의 `docker-compose.yml`에서 동일한 네트워크에 `external: true` 옵션을 지정하면 IP/서비스명으로 서로 통신할 수 있다.

---

## 💡 한 줄 요약
> Docker Compose는 docker-compose.yml 하나로 여러 컨테이너의 구성(이미지, 포트, 볼륨 등)을 정의하고, `up`/`down` 명령어만으로 전체를 한 번에 관리할 수 있게 해주는 도구다.

## ❓ 더 찾아볼 것
- Docker Compose에서 여러 서비스 간 depends_on으로 실행 순서 제어하기
- 외부 브릿지 네트워크(external: true)를 실제로 구성해보는 실습
- Docker Swarm / Kubernetes와 Docker Compose의 역할 차이
