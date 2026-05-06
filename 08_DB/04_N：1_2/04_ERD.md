# [DB] ERD (Entity-Relationship Diagram)

---

## 1. ERD란

**ERD(Entity-Relationship Diagram)**는 데이터베이스의 구조를 시각적으로 표현하는 도구이다. Entity(개체), 속성(Attribute), 그리고 Entity 간의 관계(Relationship)를 그래픽 형태로 나타내어 시스템의 논리적 구조를 모델링한다.

- **박스(Entity)**: 테이블을 나타냄
- **박스 안의 필드(Attribute)**: 컬럼(속성)을 나타냄
- **박스 간의 선(Relationship)**: 테이블 간의 관계를 나타냄

> ERD는 개발자뿐 아니라 기획자, 디자이너 등 다양한 직군이 데이터 구조를 공통으로 이해하기 위한 설계도이다. **데이터베이스 모델링은 실제 코드 작성 전에 가장 먼저 진행해야 한다.** 미리 ERD를 그려보면 중복 설계나 누락된 관계를 사전에 파악하고, 팀원 간 소통 비용도 줄일 수 있다.

---

## 2. ERD의 구성 요소

### Entity (개체)
데이터베이스에 저장되는 객체나 개념. 하나의 테이블에 해당한다.

- ex) 게시글(Article), 댓글(Comment), 회원(User)

### Attribute (속성)
Entity가 가지는 고유한 데이터 항목. 테이블의 컬럼(Column)으로 표현된다.

- ex) `Article` 엔티티의 속성: `id(integer)`, `title(varchar)`, `content(TEXT)`, `created_at(datetime)`

### Relationship (관계)
Entity 간의 연관성. 테이블 간 연결된 선으로 표현한다.

- ex) 회원이 '작성'한 댓글 → User와 Comment의 관계

---

## 3. Cardinality (카디널리티)

한 Entity와 다른 Entity 간의 **수적 관계**를 나타내는 표현이다. 선의 끝부분 기호(Crow's foot, 까마귀 발)로 표기한다.

| 표기 기호 | 의미 |
|---|---|
| `|` (직선 하나) | One |
| `||` (직선 두 개) | One and only one |
| `O` (원) | Zero |
| `<` (까마귀 발) | Many |
| `O<` | Zero or many |
| `|<` | One or many |
| `O|` | Zero or one |

### 적용 예시: User와 Comment의 관계

"회원은 여러 댓글을 작성한다. 각 댓글은 하나의 회원만 존재한다."

```
User   ||----O<   Comment
```

- User 쪽: `||` → 하나의 User만 (One and only one)
- Comment 쪽: `O<` → 0개 이상의 Comment (Zero or many)

정확히는 1대 N이지만, 댓글을 0개 작성할 수도 있으므로 "Zero or many"로 표현한다.

---

## 4. ERD의 중요성

**데이터베이스 설계 시 가장 먼저 진행**해야 한다. 코드 작성 전에 ERD를 완성해두면 다음과 같은 이점이 있다.

- 복잡한 비즈니스 로직을 직관적인 다이어그램으로 정리할 수 있다.
- 중복 데이터나 비효율적인 구조를 미리 발견할 수 있다.
- 개발자, 기획자, 디자이너 등 다양한 직군이 DB 구조를 공통으로 이해할 수 있다.
- 변경이나 유지보수 시 ERD를 기반으로 안정적으로 수정할 수 있다.

> 관통 프로젝트 시 페어 프로그래밍을 시작하기 전에, 서로 ERD를 먼저 그려보고 모델 관계를 합의한 후 개발을 시작하는 것을 권장한다.

---

## 5. 무료 ERD 제작 사이트

| 사이트 | 특징 |
|---|---|
| **Draw.io (diagrams.net)** | 회원가입 없이 바로 사용 가능, 다양한 다이어그램 템플릿 제공 / https://app.diagrams.net/ |
| **ERDCloud** | 실시간 협업 기능 지원, 공유 링크로 팀원과 동시 편집 가능 / https://www.erdcloud.com/ |

팀 프로젝트에서는 실시간 협업이 가능한 **ERDCloud**를 주로 사용하게 된다. ERDCloud에서는 다른 사람이 작성한 ERD도 열람할 수 있어, 실제 서비스의 데이터 구조를 참고 학습하는 데에도 유용하다.

---

## 6. 참고 - 인증된 사용자만 댓글 작성 및 삭제

비로그인 사용자가 Postman 등으로 직접 POST 요청을 보내면 댓글을 작성하거나 삭제할 수 있다. `@login_required`를 추가해 이를 방지한다.

```python
# articles/views.py
from django.contrib.auth.decorators import login_required

@login_required
@require_POST
def comments_create(request, pk):
    ...

@login_required
@require_POST
def comments_delete(request, article_pk, comment_pk):
    ...
```

`@require_POST`와 `@login_required`는 서로 독립적인 조건을 검사하므로 순서의 영향이 크지 않다. 원하는 순서로 적용하면 된다.

---

## 7. 다음 시간 예고 - 다대다(M:N) 관계

지금까지 배운 1:N 관계 외에 **M:N (다대다) 관계**가 존재한다. 대표적인 사례가 **좋아요**와 **팔로우** 기능이다.

- **좋아요**: User ↔ Article (서로가 서로에 대해 1:N 관계 → 다대다)
- **팔로우**: User ↔ User (자기 자신을 재귀적으로 참조하는 다대다)

M:N 관계는 외래 키를 어느 한쪽에 둘 수 없기 때문에, **중개 테이블(intermediary table)**이 생성된다. 중개 테이블은 양쪽 테이블에 대한 외래 키를 각각 하나씩 가지며, 이를 통해 양방향 참조가 가능해진다.

```
Article  ||----O<   LikeTable   >O----||  User
                    (article_id, user_id)
```

Django에서는 `ManyToManyField`를 사용하면 이 중개 테이블을 자동으로 생성해 준다.

---

## 💡 한 줄 요약
> ERD는 DB 설계의 첫 번째 단계로, Entity·Attribute·Relationship 세 가지 구성요소와 Crow's foot 기호로 테이블 간 관계를 시각화한다.

## ❓ 더 찾아볼 것
- ERD 표기법 종류 (Crow's foot, Chen notation, UML)
- 정규화(Normalization) 1NF ~ 3NF 개념
- Django `ManyToManyField`와 `through` 옵션
- ERDCloud에서 실제 프로젝트 ERD 작성해보기
