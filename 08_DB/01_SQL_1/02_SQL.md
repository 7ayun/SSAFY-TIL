# [DB] SQL

---

## 1. SQL이란?

- **SQL** (Structured Query Language): 관계형 데이터베이스에 요청(질의)을 보내는 언어
- Django에서는 ORM을 통해 간접적으로 활용했지만, SQL로 직접 데이터베이스를 조작할 수 있음

## 2. SQL 작성 규칙

- **키워드는 대문자** 권장 (소문자도 가능하지만 테이블명·필드명과의 구분을 위해)
- **문장의 끝에 세미콜론(`;`)** 필수 → 세미콜론이 없으면 엔터를 쳐도 문장이 끝나지 않음

## 3. SQL Statements 4가지 유형

| 유형 | 역할 | SQL 키워드 |
|---|---|---|
| DDL (Data Definition Language) | 데이터의 기본 구조 및 형식 변경 | CREATE, DROP, ALTER |
| DQL (Data Query Language) | 데이터 검색 | SELECT |
| DML (Data Manipulation Language) | 데이터 조작 (추가, 수정, 삭제) | INSERT, UPDATE, DELETE |
| DCL (Data Control Language) | 사용자 권한 제어 | COMMIT, ROLLBACK, GRANT, REVOKE |

> DCL은 RDBMS마다 차이가 크고 별도 시스템 설치가 필요해 정규 과정에서는 다루지 않음  
> 오늘 수업의 핵심은 **DQL(SELECT)** — 잘 검색하는 것이 조작보다 더 어렵고 중요함

---

## 💡 한 줄 요약
> SQL은 관계형 DB에 요청을 보내는 언어이며, 목적에 따라 DDL·DQL·DML·DCL로 나뉜다

## ❓ 더 찾아볼 것
- DDL, DML 각 키워드 상세 사용법
- DCL (권한 제어) 개념 정리
