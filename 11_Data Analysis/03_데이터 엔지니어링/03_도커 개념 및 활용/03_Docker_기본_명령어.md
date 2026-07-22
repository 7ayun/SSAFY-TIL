# [데이터 엔지니어링] Docker 기본 명령어

---

## 1. 이미지 관련 명령어

| 명령어 | 설명 |
|---|---|
| `docker search [이름]` | Docker Hub에서 이미지를 검색. 직접 이미지를 pull 받기 전에 정확한 이미지명을 확인할 때 사용 |
| `docker pull [이미지명]` | 이미지 다운로드. 이미지명을 명확하게 선언해서 레지스트리로부터 받아옴 |
| `docker images` (또는 `docker image ls`) | 로컬 이미지 목록 확인 |
| `docker rmi [이미지ID/이름]` | 이미지 삭제 |
| `docker tag [기존이름] [새이름]` | 이미지에 별칭(태그) 붙이기. 처음 이미지를 만들 때 옵션으로 이름을 준 것처럼, 이후에도 다양한 이름(버전)으로 관리할 수 있음 |

## 2. 컨테이너 실행 & 관리 명령어

가장 많이 쓰게 되는 것은 결국 컨테이너를 실행하는 명령어들이다.

| 명령어 | 설명 |
|---|---|
| `docker run [이미지]` | 컨테이너 실행 |
| `docker run -it [이미지]` | 대화형(interactive) 실행. 터미널에 접근할 수 있는 형태로 실행 |
| `docker run -d [이미지]` | 백그라운드(detached) 실행. 현재 터미널에 실행 로그를 출력하지 않고 뒤에서 동작시킴 |
| `docker run --name [이름] [이미지]` | 컨테이너에 이름을 부여한 다음 실행 |
| `docker ps` | 실행 중인 컨테이너 목록 확인 |
| `docker ps -a` | 종료된 컨테이너까지 포함한 모든 컨테이너 목록 확인 |
| `docker stop [이름/ID]` | 컨테이너 정지 |
| `docker restart [이름/ID]` | 컨테이너 재시작 |
| `docker rm [이름/ID]` | 컨테이너 삭제 |

> 컨테이너를 정리할 때는 항상 **stop을 먼저 하고 rm을 해야** 한다. 실행 중인 컨테이너는 바로 삭제되지 않기 때문이다.

## 3. 컨테이너 내부 다루기

실행 중인 컨테이너 안에 들어가서 뭔가 명령어를 실행해보고 싶을 때 사용하는 명령어들이다.

| 명령어 | 설명 |
|---|---|
| `docker exec [이름] [명령]` | 실행 중인 컨테이너에 명령을 실행. 예를 들어 `docker exec pycheck ls`처럼 실행하면, 컨테이너 내부의 현재 작업 디렉토리에 있는 파일 목록을 바로 확인할 수 있다 |
| `docker exec -it [이름] bash` | 컨테이너 내부로 직접 접속. `-it` 옵션 덕분에 터미널에서 인터랙티브하게 컨테이너 내부로 들어가, 그 안에서 추가적인 명령어를 자유롭게 입력할 수 있다 (bash shell로 진입) |
| `docker cp [파일] [컨테이너]:[경로]` | 컨테이너에 파일 복사. 보통은 Dockerfile에서 COPY로 미리 선언해두지만, 이를 못했거나 단발성으로 파일 하나만 옮겨야 할 때 사용. 예: `docker cp test.txt pycheck:/workspace` |
| `docker container stats [이름]` | 컨테이너 자체의 리소스 사용량(CPU, 메모리 등)을 확인 |

## 4. 이미지 빌드 명령어

| 명령어 | 설명 |
|---|---|
| `docker build -t [이미지이름] .` | 현재 디렉토리에 있는 Dockerfile과 관련 파일을 기반으로 이미지를 생성 |

## 5. Docker Compose 명령어 (간단 소개)

여러 컨테이너를 한 번에 실행하고 관리하는 도구인 Docker Compose의 명령어도 기본 골자는 비슷하다. 자세한 내용은 다음 강의(Docker Compose)에서 다룬다.

| 명령어 | 설명 |
|---|---|
| `docker-compose up -d` (또는 `docker compose up -d`) | 여러 컨테이너를 한 번에 백그라운드로 실행 |
| `docker-compose ps` | 실행 상태 확인 |
| `docker-compose stop` | 중지 |
| `docker-compose down` | 컨테이너와 네트워크까지 삭제 |

> 참고: 예전에는 `docker-compose`처럼 중간에 dash(-)가 반드시 필요했지만, 최근 버전에서는 `docker compose`처럼 띄어써도 동일하게 동작한다. 최근에는 띄어쓰는 방식이 더 많이 쓰이는 추세다.

---

## 💡 한 줄 요약
> 도커의 기본 명령어는 이미지 관련(search/pull/images/rmi/tag), 컨테이너 실행·관리(run/ps/stop/restart/rm), 컨테이너 내부 제어(exec/cp/stats), 빌드(build) 네 그룹으로 나눠서 기억하면 된다.

## ❓ 더 찾아볼 것
- docker exec와 docker attach의 차이
- docker logs 명령어로 컨테이너 로그 확인하는 방법
- docker system prune으로 불필요한 리소스 정리하기
