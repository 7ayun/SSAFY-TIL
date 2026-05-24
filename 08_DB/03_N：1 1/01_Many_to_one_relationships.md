# [DB] Many to one relationships

---

## 1. 모델 관계의 종류

| 관계 | 설명 | 예시 |
|---|---|---|
| 1:1 | 한쪽 레코드가 다른 테이블의 단 하나의 레코드와만 관계 | 유저 ↔ 프로필 |
| **N:1 (오늘)** | 여러 레코드가 다른 테이블의 레코드 하나를 참조 | 댓글 ↔ 게시글 |
| N:M (다음 주) | 여러 레코드가 서로의 여러 레코드를 양방향으로 참조, 중개 테이블 존재 | 좋아요 등 |

> 1:1과 N:1은 중개 테이블 없이 직접 참조 → 구현 방식 유사  
> N:M은 중개 테이블이 필요해 구현 방식이 다름

---

## 2. Many to one relationships (N:1)

- **정의**: 한 테이블의 **0개 이상**의 레코드가 다른 테이블의 레코드 **1개**와 관련된 관계
- 댓글(N) ↔ 게시글(1): 하나의 게시글에 여러 댓글이 달릴 수 있지만, 댓글 하나가 여러 게시글에 동시에 달릴 수는 없음
- **0개 이상**인 이유: 게시글이 처음 생성될 때는 댓글이 없는 상태도 허용해야 하기 때문

### 외래키 (ForeignKey)

- FK는 항상 **N쪽** (댓글)에 위치
- 외래키 컬럼에 저장되는 건 참조하는 데이터의 **Primary Key(PK) 값**
- 예) 사원증의 사번 → 인사 DB의 사원 정보를 참조하는 외래키

### ForeignKey 선언

```python
# articles/models.py
class Comment(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE)
    content = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

- **인스턴스명은 참조 모델 클래스의 단수형으로 작성** 권장 (`article`)
  - 단일 객체를 참조하므로 의미가 명확해지고, 코드 문맥이 자연스러워짐
- **`on_delete=models.CASCADE`**: 참조하는 게시글이 삭제되면 댓글도 함께 삭제

### Django shell에서 댓글 생성

```python
# 방법 1: 빈 인스턴스 생성 후 할당
comment = Comment()
comment.content = '댓글 내용'
article = Article.objects.get(pk=1)
comment.article = article  # 객체를 직접 할당 (권장)
# comment.article_id = article.pk  # pk를 직접 넣는 방법 (권장하지 않음)
comment.save()

# 방법 2: 초기값으로 한 번에 생성
comment = Comment(content='두 번째 댓글', article=article)
comment.save()
```

> `article_id`에 pk를 직접 넣는 것보다 **객체를 전달하는 것을 권장**  
> 이유: 객체를 할당하면 타입 오류를 Django가 검증해주지만, pk를 직접 넣으면 잘못된 값이 들어가도 에러가 발생하지 않음

### 참조 (N → 1): dot notation

```python
# 댓글에서 게시글로 참조
comment.article          # 게시글 객체
comment.article.content  # 게시글 내용 (dot으로 체이닝)
comment.article.pk       # 게시글 pk
```

> `comment.article.content` 한 줄 = SQL의 INNER JOIN + WHERE 조건  
> 이것이 ORM의 힘이자 프레임워크의 장점

---

## 💡 한 줄 요약
> N:1 관계는 N쪽(댓글)에 ForeignKey를 선언하고, FK 인스턴스명은 참조 모델의 단수형으로 짓는다

## ❓ 더 찾아볼 것
- `on_delete` 옵션 종류 (CASCADE, SET_NULL, PROTECT 등)
- ERD (Entity Relationship Diagram) 다이어그램 읽는 방법
- N:M 관계의 중개 테이블 구조
