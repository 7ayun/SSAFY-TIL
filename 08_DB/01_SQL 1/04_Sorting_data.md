# [DB] Sorting data

---

## 1. ORDER BY 기본 구조

```sql
SELECT field1
FROM table_name
ORDER BY field ASC;   -- 오름차순 (기본값)
ORDER BY field DESC;  -- 내림차순
```

- `SELECT FROM` 뒤에 작성
- `ASC`: 오름차순 (기본값이라 생략 가능하지만 **명시성을 위해 작성 권장**)
- `DESC`: 내림차순

## 2. 다중 정렬

```sql
SELECT LastName, FirstName, Country
FROM customers
ORDER BY Country ASC, LastName DESC;
```

- 여러 필드로 정렬 시 쉼표로 구분
- 앞에 작성된 필드가 우선 적용됨

## 3. SQL 들여쓰기

- SQL은 들여쓰기, 줄바꿈에 관계없이 동작
- 단, **띄어쓰기**와 **세미콜론**은 반드시 지켜야 함
- 들여쓰기는 가독성을 위한 관례

---

## 💡 한 줄 요약
> ORDER BY로 조회 결과를 정렬하며, ASC(오름차순)/DESC(내림차순)를 명시하고 여러 필드로 다중 정렬도 가능하다

## ❓ 더 찾아볼 것
- NULL 값이 포함된 경우 정렬 순서
