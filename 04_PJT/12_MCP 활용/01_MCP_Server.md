# [관통 PJT] MCP Server

---

## 1. MCP란 무엇인가

기기마다 충전기가 다르던 시대가 USB-C 하나로 통일된 것처럼, AI가 다양한 외부 서비스에 접근하는 방식도 기존에는 각 API마다 개별 코드를 따로 개발해야 했다. MCP는 이 문제를 해결한다.

**MCP (Model Context Protocol)** — 어플리케이션이 LLM(Model)에게 문맥(Context)을 제공하는 방법을 표준화한 프로토콜

- 2024년 11월 26일 Anthropic에서 공개
- 2025년 2월 19일 AI 코드 에디터인 Cursor AI에서 MCP 도입
- 2025년 3월 27일 OpenAI도 MCP 도입 (Sam Altman이 직접 X에 공표)

MCP 규약을 지켜서 API를 만들면, 각 서비스마다 API를 따로따로 연결하지 않아도 된다. LLM(MCP hosts)는 사용자의 입력을 분석해서 필요한 서비스(MCP Server)에 연결하고, 해당 서비스가 제공하는 기능 중 필요한 기능에 연결 후 로직 처리를 한다.

### MCP 아키텍처 구성 요소

| 구성 요소 | 역할 |
|---|---|
| **MCP Hosts** | 사용자의 문장을 해석, 어떤 MCP Server의 어떤 기능을 호출할지 분석 (예: VSCode Copilot, ChatGPT, Claude) |
| **MCP Client** | Hosts가 분석한 내용을 표준화된 JSON(MCP) 형태로 변환, MCP Server와 통신 및 보안 처리 담당 |
| **MCP Server** | 각 회사가 고유한 데이터나 도구를 MCP 표준 규격으로 만들어 제공 (OpenAPI를 만든다고 생각하면 됨) |

현재 약 1,000개 가량의 MCP 서버가 만들어져 있으며, Fetch, Filesystem, Git, Memory, Sequential Thinking 등 다양한 레퍼런스 서버가 공식 GitHub에 공개되어 있다.

---

## 2. 패키지 매니저 uv

MCP 공식과 Anthropic에서 공식적으로 권장하는 Python 패키지 매니저.

- 기존의 `pip`와 `venv`를 **대체**하는 초고속 패키지 매니저
- Rust 언어로 만들어졌으며, 컴파일러 + 멀티태스킹으로 **pip-sync 대비 약 77배 빠름**
- 프로젝트 생성부터 라이브러리 설치, 가상환경 관리를 명령어 하나로 처리

### 프로젝트 초기 설정

```bash
# 1. uv 설치
$ pip install uv

# 2. 프로젝트 생성 및 초기화
$ uv init weather-server
$ cd weather-server

# 3. 가상환경 생성 및 활성화
$ uv venv
$ source .venv/Scripts/activate  # Windows

# 4. MCP SDK 설치 (httpx는 비동기 HTTP 통신 라이브러리)
$ uv add "mcp[cli]" httpx
```

`uv init`으로 프로젝트를 생성하면 `.gitignore`, `.python-version`, `main.py`, `pyproject.toml`, `README.md`, `uv.lock` 파일이 자동 생성된다.

---

## 3. MCP 서버 코드 작성

### FastMCP란

MCP 서버를 빠르고 직관적으로 개발할 수 있도록 도와주는 인터페이스. 주요 기능은 다음과 같다.

- **데코레이터**: 작성한 함수(도구)를 MCP 서버 목록에 자동 등록
- **타입 힌트 자동 변환**: 도구에 필요한 파라미터의 스키마를 자동 생성해서 AI에 전달
- **Docstring 활용**: 함수 바로 아래 적는 `"""함수설명"""`을 AI에게 사용 설명서로 전달

### 전체 코드 구조

```python
# weather-server/weather.py

from typing import Any
import httpx
from mcp.server.fastmcp import FastMCP

# 1. 서버 이름표 붙이기
mcp = FastMCP("weather")

# 2. 도구(Tool) 만들기
@mcp.tool()
async def get_weather_custom(latitude: float, longitude: float) -> str:
    """
    특정 위치(위도, 경도)의 날씨 예보를 반환합니다.
    Args:
        latitude: 위도 (예: 37.56)
        longitude: 경도 (예: 126.97)
    """
    # 오픈소스 날씨 API 주소 (별도의 키 발급 없이 사용 가능)
    url = f"https://api.open-meteo.com/v1/forecast?latitude={latitude}&longitude={longitude}&current=temperature_2m,wind_speed_10m&hourly=temperature_2m,relative_humidity_2m,wind_speed_10m"

    # 비동기(Async) 방식으로 웹사이트에 접속하여 데이터 가져오기
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        data = response.json()

    return str(data)

# 3. 서버 실행 (엔트리 포인트)
if __name__ == "__main__":
    mcp.run()
```

### httpx — 비동기 HTTP 통신

기존의 `requests` 라이브러리는 비동기를 지원하지 않는다. MCP 서버는 비동기로 동작하기 때문에 `httpx.AsyncClient()`를 사용해야 한다.

```python
async with httpx.AsyncClient() as client:
    response = await client.get(url)
    data = response.json()
```

### mcp.run() 동작 방식

일반 서버와 달리 Port(주소)를 열고 기다리지 않으며(**주소가 없음**), AI와 표준 입출력 방식으로 텍스트 문자열(JSON)을 주고받으면서 대화한다. 내부적으로 무한 루프를 돌면서 요청을 기다린다.

---

## 4. VSCode에 MCP 서버 등록 및 실행

`ctrl + shift + p` → "MCP: Open User Configuration" → `mcp.json` 파일 열기

```json
// C:\Users\Admin\AppData\Roaming\Code\User\mcp.json
{
    "servers": {
        "weather": {
            "command": "uv",
            "args": [
                "run",
                "--directory",
                "C:\\Users\\Admin\\Downloads\\pjt10\\weather-server",
                "weather.py"
            ],
            "enabled": true
        }
    }
}
```

등록 후 `Start` 버튼을 누르면 서버가 실행된다. 실행 오류 시 OUTPUT 탭의 로그를 확인해서 수정 후 재실행한다. 정상 실행 후 VSCode Copilot을 **Agent 모드**로 변경하면 등록된 MCP 서버 목록에서 `weather` 서버를 확인할 수 있다.

---

## 5. MCP 서버 배포 — GitHub

로컬 파일 경로 없이 GitHub 주소만으로 MCP 서버를 공유하고 활용할 수 있다.

### pyproject.toml 수정

배포 전 `pyproject.toml`에 **엔트리 포인트**를 반드시 명시해야 한다.

```toml
# weather-server/pyproject.toml

[project]
name = "weather-server-jayden"
version = "0.1.0"
description = "MCP 날씨 서버 Example!"
readme = "README.md"
requires-python = ">=3.11"
dependencies = [
    "httpx>=0.28.1",
    "mcp[cli]>=1.22.0",
]

[project.scripts]
weather-server = "weather:main"   # ← 이 부분 추가
```

> **주의**: `[project]`의 `name`과 `[project.scripts]`의 스크립트 이름이 일치해야 한다.

### weather.py 엔트리포인트 수정

```python
# 기존
if __name__ == "__main__":
    mcp.run()

# 변경 후 (main 함수로 감싸기)
def main():
    mcp.run()

if __name__ == "__main__":
    main()
```

### main.py 삭제 주의

`uv init` 시 자동 생성된 `main.py`를 반드시 삭제해야 한다. 삭제하지 않으면 최상위 모듈(`main.py`, `weather.py`)이 여러 개로 간주되어 **빌드 실패**한다.

### GitHub 업로드

`uv init`으로 프로젝트 생성 시 `git init`이 자동으로 진행된다. 저장소 연결 후 업로드만 하면 된다.

```bash
# 1. Stage에 파일 추가
$ git add .

# 2. 커밋 작성
$ git commit -m "Init Commit:MCP Server"

# 3. 저장소 연결
$ git remote add origin [git 주소]

# 4. 저장소에 코드 업로드
$ git push origin main
```

### VSCode에서 GitHub 기반 MCP 서버 등록

```json
{
    "servers": {
        "weather-github": {
            "command": "uvx",
            "args": [
                "--from",
                "git+https://github.com/ssafy/weather-server-jayden.git",  // 본인 git 주소
                "weather-server"
            ],
            "enabled": true
        }
    }
}
```

---

## 6. MCP 서버 배포 — PyPI

PyPI(Python Package Index)는 파이썬 공식 저장소로, 정식 버전 관리가 용이하고 설치 및 실행 속도가 훨씬 빠르다.

### PyPI 계정 및 토큰 발급

1. pypi.org 회원가입 → 이메일 인증 → **2단계 인증** 완료
2. 계정 설정 → API 토큰 추가 → 토큰 생성 (범위: 모든 계정)
3. 생성된 토큰 즉시 복사 (보안상 **한 번만** 보여줌)

### pyproject.toml 이름 고유하게 변경

```toml
[project]
name = "weather-server-jayden"   # ← PyPI에서 고유한 이름으로 설정 (중복 불가)

[project.scripts]
weather-server-jayden = "weather:main"   # ← name과 동일하게
```

### 빌드 및 배포

```bash
# 프로젝트 빌드 (소스 코드를 배포용 파일로 변환)
$ uv build
# → dist/ 폴더에 .tar.gz, .whl 파일 생성

# 프로젝트 배포
$ uv publish
# Enter username: __token__     (이 글자 그대로 입력)
# Enter password: [복사해둔 토큰 붙여넣기]
```

배포 완료 후 pypi.org에서 프로젝트 명을 검색해 정상 업로드 여부를 확인할 수 있다.

### VSCode에서 PyPI 기반 MCP 서버 등록

```json
{
    "servers": {
        "weather-pypi": {
            "command": "uvx",
            "args": [
                "weather-server-jayden"   // 배포한 패키지 이름만 입력
            ],
            "enabled": true
        }
    }
}
```

GitHub 배포와 달리 `uvx`가 패키지를 **설치 없이 CDN처럼 가져와** 실행한다.

---

## 💡 한 줄 요약
> MCP는 AI가 외부 서비스를 표준 방식으로 연결하는 프로토콜이며, FastMCP + uv로 나만의 MCP 서버를 만들고 GitHub 또는 PyPI에 배포해 누구나 사용할 수 있게 공유할 수 있다.

## ❓ 더 찾아볼 것
- MCP 공식 문서: https://modelcontextprotocol.io/docs/develop/build-server
- MCP 레퍼런스 서버 목록: https://github.com/modelcontextprotocol/servers
- uv 공식 문서: https://github.com/astral-sh/uv
- `@mcp.resource`, `@mcp.prompt` 데코레이터의 역할 및 활용법
- SSE(Server-Sent Events) 방식 MCP 서버와 stdio 방식 차이
