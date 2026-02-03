# Day07 — Git · 파일 시스템 · AI 활용 복습 노트

> **목표:** 도구 암기 ❌ → 선택 기준 & 흐름 이해 ⭕

---

## 1️⃣ Git 복구 전략

1. 기록을 남기며 취소 → ______
2. 과거 커밋으로 이동 → ______

<details>
<summary>정답</summary>

1. git revert
2. git reset
</details>

---

## 2️⃣ git reset 옵션

1. 스테이징에 남김 → ______
2. 워킹 디렉토리에 남김(기본) → ______
3. 완전 삭제 → ______

<details>
<summary>정답</summary>

soft
mixed
hard
</details>

---

## 3️⃣ pathlib 개념

1. pathlib은 경로를 ______로 다룬다.
2. 현재 작업 디렉토리 → `Path.______()`

<details>
<summary>정답</summary>

객체
cwd
</details>

---

## 4️⃣ 파일 생성

```python
p = Path('data')
p.mkdir(exist_ok=True)
```

1. `exist_ok=True` 의미: ______

<details>
<summary>정답</summary>

이미 존재해도 에러 발생 안 함
</details>

---

## 5️⃣ with 문

1. with의 목적: 리소스 ______ 관리
2. 파일을 추가 모드로 열기 → `'__'`

<details>
<summary>정답</summary>

안전한
'a'
</details>

---

## 6️⃣ 파일 탐색

1. 현재 폴더만 탐색 → ______
2. 하위 폴더까지 탐색 → ______

<details>
<summary>정답</summary>

iterdir / glob
rglob
</details>

---

## 7️⃣ 인코딩

1. 한글 깨짐 방지 인코딩 → ______

<details>
<summary>정답</summary>

UTF-8
</details>

---

## 8️⃣ 사고 연결 문제

파일 여러 개를 읽어 요약 md 파일을 만드는 순서:

1. ______ 탐색
2. 내용 ______
3. 텍스트 ______
4. AI API ______
5. 파일로 ______

<details>
<summary>정답</summary>

1. 경로
2. 읽기
3. 결합
4. 요청
5. 저장
</details>

---

### 이동
👉 [학습 정리](./README.md)
👉 [메인 README](../README.md)