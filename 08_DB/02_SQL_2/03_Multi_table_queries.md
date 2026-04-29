# [DB] Multi table queries - 2026.04.28

---

## 1. JOIN이 필요한 이유

커뮤니티 게시판을 예로 들면, 게시글 테이블에 작성자 정보(writer, role)까지 함께 저장할 경우 문제가 발생한다.

- 동명이인 구분 불가
- 작성자 이름 변경 시 모든 게시글의 데이터 수정 필요
- 게시글과 성격이 다른 데이터(회원 정보)가 섞임

→ **정규화**: 게시글 / 유저 / 역할 테이블을 독립적으로 분리하고, FK로 연결

> JOIN = **여러 테이블 간의 논리적 연결** → 관계형 데이터베이스의 핵심

---

## 2. INNER JOIN

```sql
SELECT 조회할_필드
FROM 테이블A
INNER JOIN 테이블B
  ON 테이블A.필드 = 테이블B.필드;
```

- 두 테이블에서 **ON 조건이 일치하는 레코드만** 조회 (교집합)
- 어느 한쪽에만 있는 데이터는 결과에서 제외됨

### 예시

```sql
-- 1번 회원(하석주)이 작성한 모든 게시글의 제목과 작성자명 조회
SELECT articles.title, users.name
FROM articles
INNER JOIN users
  ON users.id = articles.userId
WHERE users.id = 1;
```

---

## 3. LEFT JOIN

```sql
SELECT 조회할_필드
FROM 테이블A
LEFT JOIN 테이블B
  ON 테이블A.필드 = 테이블B.필드;
```

- **왼쪽 테이블(A)의 모든 레코드** 출력
- 오른쪽 테이블(B)에 매칭되는 레코드가 없으면 **NULL** 표시

### 활용 예시
- 게시글을 한 번도 작성하지 않은 회원 찾기
- 주문이 없는 고객 찾기

---

## 4. INNER JOIN vs LEFT JOIN 비교

| 구분 | INNER JOIN | LEFT JOIN |
|---|---|---|
| 결과 범위 | 두 테이블 모두 일치하는 데이터만 | 왼쪽 테이블 전체 |
| 매칭 없을 때 | 해당 행 제외 | NULL로 표시 |
| 활용 | 연관된 데이터 함께 조회 | 없는 데이터 찾기 |

---

## 💡 한 줄 요약
> INNER JOIN은 두 테이블의 교집합, LEFT JOIN은 왼쪽 테이블 전체를 기준으로 연결하며 매칭 없으면 NULL로 표시한다

## ❓ 더 찾아볼 것
- RIGHT JOIN, FULL OUTER JOIN 개념
- 정규화 (1NF, 2NF, 3NF) 상세 개념
- Django ORM에서 JOIN이 어떻게 표현되는지 (select_related, prefetch_related)
