# [Python] 제어문 및 모듈 (Control Flow & Modules)

> **핵심 키워드:** #Conditional #Loop #If #For #While #Module #Package #PIP #List_Comprehension

---

## 🎯 학습 목표
* 조건별 코드 실행 흐름 제어(If) 및 반복 구조(For, While) 완벽 숙달
* 중첩 반복문 활용 복합 데이터 구조 순회 역량 강화
* 반복 제어어(break, continue, pass) 정밀 동작 원리 파악
* 모듈/패키지 구조 및 외부 라이브러리 관리(PIP) 체계 이해

---

## 💡 주요 개념 정리

### 1. 모듈(Module)과 패키지(Package)
* **모듈:** 특정 기능 수행 변수 및 함수 집합체 (.py 파일 단위)
* **패키지:** 관련 모듈 관리용 디렉토리(폴더) 구조
* **표준 라이브러리:** 파이썬 설치 시 기본 포함된 math, time, os 등 내장 모듈
* **의존성 관리 (PIP):** 외부 패키지 및 관련 하위 패키지 자동 설치 시스템

### 2. 조건문 (Conditional Statements)
* **순차 평가:** 상단 조건부터 순차 검사 및 첫 번째 참(True) 블록 실행 구조
* **구성 요소:** `if`(필수), `elif`(복수 조건 선택), `else`(모든 조건 미충족 시 실행)

### 3. 반복문 (Loops)
* **For 문:** 리스트, range 등 정해진 시퀀스 대상 순차 순회 (유한 반복)
* **While 문:** 조건식이 거짓(False)이 될 때까지 지속 실행 (탈출 조건 필수)

---

## 💻 기능 구현 및 코드 실습

### 1. 모듈 활용 및 별칭 사용
내장 모듈 로드 및 이름 충돌 방지용 별칭(alias) 적용 기법

```python
import math as m # math 모듈 m으로 로드

# 점(.) 연산자 기반 모듈 내 기능 접근
print(m.pi)        # 원주율 상수 확인
print(m.sqrt(16))  # 제곱근 계산 함수 호출
```

### 2. 중첩 조건문 및 복수 조건 처리
미세먼지 농도 수치 기반 경고 문구 출력 논리 분기

```python
dust = 155

if dust > 150:
    print("매우 나쁨")
    if dust > 300: # 내부 중첩 조건
        print("실외 활동 금지")
elif dust > 80:    # 선행 조건 거짓일 때만 평가
    print("나쁨")
else:              # 모든 조건 미충족 시 실행
    print("보통/좋음")
```

### 3. 반복문 제어 (break, continue, for-else)
특정 조건 만족 시 반복 즉시 중단 또는 다음 순회 진행 제어 기법



```python
numbers = [1, 3, 5, 8, 9]

for num in numbers:
    if num % 2 == 0:
        print(f"첫 번째 짝수 {num} 발견")
        break # 루프 즉시 탈출
else:
    # break 없이 루프가 정상 종료된 경우에만 실행
    print("짝수 미발견")

# 짝수 건너뛰기 로직
for i in range(5): 
    if i % 2 == 0:
        continue # 하단 코드 무시 및 다음 순회 진행
    print(i)     # 홀수만 출력
```

### 4. 고급 반복 기법 (enumerate, Comprehension)
인덱스 동시 추출 및 간결한 리스트 생성을 위한 파이썬 특화 기법

```python
members = ["Patrick", "Minkyu", "Shark"]

# enumerate: 인덱스와 값을 쌍으로 반환
for idx, name in enumerate(members):
    print(f"{idx}번 멤버: {name}")

# List Comprehension: 한 줄 제어문 구성
# [결과표현식 for 변수 in 객체 if 조건]
squared_odds = [x**2 for x in range(10) if x % 2 != 0]
print(squared_odds) # 홀수 제곱값 리스트 생성
```

---

## 🚀 복습 및 AI 인사이트
* **설계 주의 사항:** `while`문 활용 시 무한 루프 방지를 위한 명확한 탈출 조건 설계 필수
* **가독성 확보:** 복잡한 중첩 루프 지양 및 내장 함수(`map`, `enumerate`) 활용 지향
* **핵심 본질:** 문법은 단순 도구이며, 제어문 조합을 통한 논리적 문제 해결 역량이 개발자의 실력