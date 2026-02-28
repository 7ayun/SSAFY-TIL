# Day02 - 1월 23일 수업 정리

# Part 1. Python 자료형 & 문법

## 1. 수업 핵심 목표

* **시퀀스 타입 완전 이해**
* 문자열 → 리스트 → 튜플 → range 개념 확장
* **인덱싱, 슬라이싱, 메모리 구조, 가변/불변** 개념 정확히 정리

---

## 2. 시퀀스 타입 공통 개념

### 시퀀스 타입이란?

* **순서가 있는 연속된 자료형**
* 공통 특징:

  * 순서 존재
  * 인덱싱 가능
  * 슬라이싱 가능
  * 길이 측정 가능 (`len()`)

---

## 3. 문자열(String) 핵심 정리

### 기본 특징

* 변경 불가능 (**Immutable**)
* 순서 있음

```python
s = "abc"
```

### 인덱싱

```python
s[0] → 'a'
s[1] → 'b'
```

### 슬라이싱

```python
s[0:2] → 'ab'
s[1:3] → 'bc'
```

### 불변(Immutable) 의미

```python
s[0] = 'k'  # ❌ 오류
```

* **원본 문자열은 변경 불가**
* 슬라이싱은 **새 문자열 생성**

```python
a = s[0:2]  # 새로운 문자열 생성
```

---

## 4. 리스트(List) 핵심 정리

### 기본 특징

* **변경 가능 (Mutable)**
* 순서 있음
* 모든 자료형 저장 가능

```python
lst = [1, 'a', 3, 'b', [4,5]]
```

### 인덱싱 / 슬라이싱

```python
lst[1] → 'a'
lst[1:4] → ['a', 3, 'b']
```

### 가변(Mutable)

```python
lst[0] = 100
```

---

## 5. 리스트 메모리 구조 핵심 이해

### 리스트가 다양한 자료형을 저장할 수 있는 이유

* **값 자체가 아닌 주소(reference)를 저장**

구조:

```
[ 주소1 | 주소2 | 주소3 | ... ]
```

### 인덱싱이 O(1)인 이유

* 리스트 시작 주소 + (인덱스 × 8바이트)

```text
주소 = 시작주소 + index * 8
```

→ **즉시 접근 가능 (시간복잡도 O(1))**

---

## 6. 튜플(Tuple) 핵심 정리

### 기본 특징

* **변경 불가능 (Immutable)**
* 순서 있음

```python
t = (1, 2, 3)
```

### 단일 요소 튜플

```python
(1,)  # 반드시 콤마 필요
```

### 튜플의 본체는 콤마

```python
1, 2, 3  → tuple
```

---

## 7. 튜플 활용 예시

### 다중 할당

```python
x, y = 10, 20
```

### 값 교환

```python
x, y = y, x
```

### 그룹화 / 언패킹

```python
student = ('Kim', 20, 'CS')
name, age, major = student
```

---

## 8. range 핵심 정리

### 기본 구조

```python
range(start, end, step)
```

### 생성 규칙

* **start 포함, end 미포함**

```python
range(5) → 0,1,2,3,4
range(1,5) → 1,2,3,4
range(5,0,-1) → 5,4,3,2,1
```

### 특징

* 변경 불가능
* 반복문(for)에서 핵심적으로 사용

---

## 9. 핵심 요약

| 타입    | 변경 | 특징        |
| ----- | -- | --------- |
| str   | ❌  | 문자 시퀀스    |
| list  | ⭕  | 가장 많이 사용  |
| tuple | ❌  | 고정 데이터    |
| range | ❌  | 정수 시퀀스 생성 |

---

## 10. 오늘의 핵심 키워드

* 시퀀스 타입
* 인덱싱 / 슬라이싱
* mutable / immutable
* reference 기반 저장 구조
* O(1) 인덱싱
* 다중 할당
* range 생성 규칙

---

# Part 2. Non-Sequence 타입

## 11. Non-Sequence 타입 개요

* **순서가 없는 자료형**
* 대표: **dict, set**

---

## 12. 딕셔너리(Dictionary)

### 기본 개념

* **키(Key) + 값(Value) 쌍** 구조
* 순서 ❌ / 중복 ❌ / 변경 ⭕

```python
d = {'apple': 12, 'list': [1,2,3]}
```

### 접근 방식

```python
d['apple'] → 12
```

* **인덱스 접근 불가 → 키로 직접 접근**

### 키 / 값 규칙

* **Key:** 변경 불가능한 자료형만 가능 (int, str, tuple, range)
* **Value:** 모든 자료형 가능

### 추가 / 수정

```python
d['banana'] = 50     # 추가
d['apple'] = 100    # 수정
```

### 특징 요약

* 검색 속도 매우 빠름 → **O(1)**
* 알고리즘에서 **최고 빈도 사용 자료형**

---

## 13. 세트(Set)

### 기본 개념

* **순서 ❌ / 중복 ❌ / 변경 ⭕**

```python
s = {1,2,3,3,3} → {1,2,3}
```

### 주요 용도

* **중복 제거**
* 빠른 탐색 (O(1))

### 집합 연산

```python
A | B  # 합집합
A & B  # 교집합
A - B  # 차집합
```

---

## 14. None 타입

### 의미

* **값이 없음 (의도적 비어 있음)**

```python
x = None
```

* null 개념
* 아직 값이 정해지지 않은 상태 표현

---

## 15. Boolean 타입

### 기본 값

```python
True, False
```

### False 판정되는 값

* 0
* 0.0
* ''
* []
* {}
* set()
* None

→ **이외 모든 값은 True**

---

## 16. 형 변환(Type Conversion)

### 암시적 형변환 (자동)

```python
3 + 5.0 → 8.0
True + 3 → 4
```

### 명시적 형변환 (수동)

```python
int('10') → 10
float('3.5') → 3.5
str(17) → '17'
```

---

## 17. 연산자 정리

### 산술 연산자

```python
+  -  *  /  //  %  **
```

### 복합 연산자

```python
a += 3
a -= 2
a *= 4
a //= 2
```

### 비교 연산자

```python
==  !=  <  >  <=  >=  is
```

* **== : 값 비교**
* **is : 주소 비교**

### 논리 연산자

```python
and  or  not
```

---

## 18. 단축 평가 (Short-circuit Evaluation)

```python
3 and 5 → 5
0 and 3 → 0
3 or 0 → 3
0 or 3 → 3
```

* and → **False 나오면 즉시 종료**
* or → **True 나오면 즉시 종료**

---

## 19. 멤버십 연산자

```python
'a' in 'apple' → True
4 in [1,2,3] → False
```

---

## 20. Day02 핵심 요약

* **Sequence:** str, list, tuple, range
* **Non-sequence:** dict, set
* **불변:** str, tuple, range
* **가변:** list, dict, set
* **알고리즘 핵심:** dict, set

---

# Part 3. Git 기초 & 로컬 버전 관리

## 21. Git 기초 개념

## 21-1. Git이 필요한 이유

* **변경 이력만 저장 → 저장 공간 효율 극대화**
* **언제든 이전 버전으로 복구 가능 (rollback)**
* **협업 시 충돌 최소화 + 변경 추적 가능**

---

## 21-2. 중앙 집중식 vs 분산 버전 관리

### 중앙 집중식

* 중앙 서버 1곳에 모든 코드 저장
* 서버 장애 시 전체 데이터 손실 위험

### 분산식 (Git)

* **모든 사용자가 전체 저장소 보유**
* 중앙 서버 장애에도 안전
* 오프라인 작업 가능

---

## 22. Git 기본 구조 (3대 영역)

```text
Working Directory  →  Staging Area  →  Repository
(작업 공간)          (임시 저장)         (영구 저장)
```

* **Working Directory**: 실제 작업 폴더
* **Staging Area**: 커밋 전 임시 대기 공간
* **Repository**: 커밋된 버전 저장소

---

## 23. Git 기본 흐름

```text
파일 수정 → git status → git add → git commit
```

---

## 24. 핵심 명령어 정리

### 저장소 생성

```bash
git init
```

### 상태 확인

```bash
git status
```

### 스테이징 등록

```bash
git add 파일명
git add .
git add -A
```

### 커밋 생성

```bash
git commit -m "메시지"
```

### 커밋 기록 확인

```bash
git log
```

---

## 25. Git 실습 흐름 예시

```bash
git init
touch a.txt
git status
git add a.txt
git commit -m "add a.txt"
```

---

## 26. Git 상태 색상 의미

* **빨간색**: Working Directory (추적 안 됨)
* **초록색**: Staging Area (커밋 대기)

---

## 27. 커밋 작성 방식

### 한 줄 메시지

```bash
git commit -m "commit message"
```

### 상세 메시지 (vim)

```bash
git commit
```

* `i` → 입력
* `ESC → :wq` → 저장 후 종료

---

## 28. 사용자 정보 설정

```bash
git config --global user.email "email"
git config --global user.name "name"
```

---

## 29. 변경 취소 (복구)

```bash
git restore 파일명
```

* **마지막 커밋 기준으로 되돌림**

---

## 30. 상대 경로 + add 동작 이해

| 명령          | 의미            |
| ----------- | ------------- |
| git add 파일명 | 특정 파일만 스테이징   |
| git add .   | 현재 디렉토리 기준 전체 |
| git add -A  | 저장소 전체 변경 사항  |

---

## 31. Day02 최종 핵심 정리

* **자료형 구조 완전 이해 → Python 기본기 완성**
* **Git 기본 흐름 완전 숙지 → 실무/협업 준비 완료**
* **Working → Staging → Commit → Push → Pull 흐름 숙달 필수**

---

# Part 4. Git 원격 저장소 & 협업

## 32. 원격 저장소(Remote Repository) & 협업 흐름

## 32-1. Git vs GitHub/GitLab

* **Git**: 버전 관리 도구
* **GitHub / GitLab**: 원격 저장소 서비스

---

## 32-2. 원격 저장소 연결

```bash
git remote add origin <REMOTE_URL>
```

* `origin`: 원격 저장소 별명
* 연결 확인:

```bash
git remote -v
```

---

## 33. Push / Pull / Clone

### Push (업로드)

```bash
git push origin master
```

* 로컬 → 원격 업로드

### Pull (동기화)

```bash
git pull origin master
```

* 원격 → 로컬 업데이트

### Clone (전체 복제)

```bash
git clone <REMOTE_URL>
```

* 최초 1회 전체 복제

---

## 34. 협업 기본 시나리오

```text
1. 최초 참여 → git clone
2. 개발 → add → commit → push
3. 다른 사람 변경 수신 → git pull
```

---

## 35. Alias 설정 (명령어 단축)

```bash
git config --global alias.st status
```

* 이후 `git st` = `git status`

---

## 36. Git init 중복 사용 금지

* **이미 git init 된 폴더 내부에서 다시 git init 금지**
* `.git` 폴더 중복 생성 → **버전 충돌 위험**

### 실수 사례

* Desktop에서 `git init`
* 하위 폴더에서 다시 `git init`

→ **중첩 저장소 생성 → 심각한 문제 발생**

---

## 37. .gitignore (추적 제외 설정)

### 목적

* **보안 정보(API KEY, env, 개인 설정 등) 추적 방지**

### 생성

```bash
touch .gitignore
```

### 예시

```
.env
secret.txt
__pycache__/
```

---

## 38. .gitignore 주의사항 (중요)

* **이미 추적 중인 파일은 gitignore에 추가해도 무시되지 않음**

→ 해결: 캐시 제거 필요 (추후 학습)

---

## 39. gitignore 자동 생성 사이트

* [https://www.toptal.com/developers/gitignore](https://www.toptal.com/developers/gitignore)

→ 프로젝트 이름 입력 → 자동 생성

---

## 40. Git 실전 전체 흐름 요약

```text
git init
→ 파일 작성
→ git status
→ git add .
→ git commit -m "message"
→ git remote add origin URL
→ git push origin master
```

---

## 41. 개발 학습 핵심 메시지 요약

* **언어는 수단, 본질은 문제 해결 능력**
* AI는 적극 활용하되, **이해 없이 복붙 금지**
* 논리적 사고 + 검증 능력 필수

---

## 42. Day02 전체 요약

* Python 자료형 → **기초 문법 기반 완성**
* Git 로컬 → 원격 → 협업 → **실전 개발 흐름 완성**
* 실습 반복 → **손에 익히는 것이 핵심**


### 🔗 이동

- 📝 복습 노트: [review.md](./review.md)
- ⬆ 메인: [README](../README.md)