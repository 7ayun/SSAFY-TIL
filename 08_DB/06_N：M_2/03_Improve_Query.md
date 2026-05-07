# [DB] Improve Query

---

## 1. N+1 Problem

ORM에서 관련 객체를 반복 접근할 때 발생하는 과도한 쿼리 현상이다.

- 초기 1번의 쿼리로 기본 데이터를 조회한 뒤, 각 객체의 연관 데이터를 조회하며 N번의 추가 쿼리가 실행된다.
- 게시글 10개를 출력할 때 각 게시글의 댓글 개수를 별도로 조회하면 총 **1 + 10 = 11번** 쿼리가 발생한다.
- 데이터가 많거나 동시 요청이 늘어날수록 서버 부하가 급증한다.

**핵심 원칙**: 결과는 동일하게 유지하면서 쿼리 횟수를 줄인다. 처음 요청할 때 필요한 데이터를 한 번에 잘 가져오는 것이 목표다.

> Django Debug Toolbar를 사용하면 현재 페이지에서 DB에 쿼리가 몇 번 전송되었는지 확인할 수 있다.

---

## 2. annotate — 집계 함수로 필드 추가

SQL의 `GROUP BY`에 해당하는 기능으로, 쿼리셋의 각 객체에 계산된 필드를 추가한다.

**문제 상황**: 게시글 목록에 댓글 개수를 표시할 때 템플릿에서 `article.comment_set.count`를 반복 호출 → 게시글 수만큼 추가 쿼리 발생

**해결**: `annotate`로 게시글 조회 시 댓글 개수를 미리 계산해 새 필드로 포함

```python
# articles/views.py
from django.db.models import Count

def index_1(request):
    # 기존 (N+1 발생)
    # articles = Article.objects.order_by('-pk')

    # 개선 (쿼리 1개로 해결)
    articles = Article.objects.annotate(comment_count=Count('comment')).order_by('-pk')
    context = {'articles': articles}
    return render(request, 'articles/index_1.html', context)
```

```html
<!-- index_1.html -->
<!-- 기존: {{ article.comment_set.count }} → DB 재요청 -->
<!-- 개선: 이미 계산된 필드를 참조만 함 -->
<p>댓글 개수: {{ article.comment_count }}</p>
```

**쿼리 변화**: 11개 → **1개** (LEFT OUTER JOIN + GROUP BY로 한 번에 조회)

---

## 3. select_related — ForeignKey/OneToOne 참조 최적화

ForeignKey 또는 OneToOne 관계에서 **INNER JOIN**을 사용해 관련 객체를 미리 가져온다.

**문제 상황**: 게시글 목록에 작성자 이름을 표시할 때 `article.user.username`을 반복 호출 → 게시글 수만큼 유저 조회 쿼리 발생 + 같은 작성자 중복 조회

**해결**: `select_related`로 게시글 조회 시 유저 정보까지 JOIN해서 한 번에 가져옴

```python
# articles/views.py
def index_2(request):
    # 기존 (N+1 발생)
    # articles = Article.objects.order_by('-pk')

    # 개선 (쿼리 1개로 해결)
    articles = Article.objects.select_related('user').order_by('-pk')
    context = {'articles': articles}
    return render(request, 'articles/index_2.html', context)
```

**쿼리 변화**: 11개 → **1개** (INNER JOIN으로 유저 정보 포함 조회)

---

## 4. prefetch_related — M:N/역참조 최적화

ManyToMany 또는 N:1 역참조 관계에서 관련 객체를 미리 가져온다. SQL이 아닌 **Python** 수준에서 JOIN을 처리한다.

**문제 상황**: 게시글 목록에 댓글 목록을 표시할 때 `article.comment_set.all()`을 반복 호출 → 게시글 수만큼 댓글 조회 쿼리 발생

**해결**: `prefetch_related`로 게시글 조회 시 역참조 연산까지 한 번에 처리

```python
# articles/views.py
def index_3(request):
    # 기존 (N+1 발생)
    # articles = Article.objects.order_by('-pk')

    # 개선 (쿼리 2개로 해결)
    articles = Article.objects.prefetch_related('comment_set').order_by('-pk')
    context = {'articles': articles}
    return render(request, 'articles/index_3.html', context)
```

**쿼리 변화**: 11개 → **2개** (전체 게시글 1번 + `IN` 연산자로 모든 댓글 1번)

---

## 5. select_related + prefetch_related 결합

게시글 → 댓글 목록 → 댓글 작성자까지 출력하는 가장 복잡한 상황에서 두 메서드를 함께 사용한다.

**문제 상황**: 게시글 10개 + 각 댓글 목록 + 댓글 작성자 표시 → **111개** 쿼리 발생

**1단계**: `prefetch_related`로 댓글 역참조 해결 → **102개**로 감소 (댓글 작성자 쿼리는 아직 남음)

**2단계**: `Prefetch` 객체와 `select_related`를 조합해 댓글 조회 시 작성자까지 JOIN

```python
# articles/views.py
from django.db.models import Prefetch

def index_4(request):
    articles = Article.objects.prefetch_related(
        Prefetch('comment_set', queryset=Comment.objects.select_related('user'))
    ).order_by('-pk')
    context = {'articles': articles}
    return render(request, 'articles/index_4.html', context)
```

**쿼리 변화**: 111개 → **2개**

| 쿼리 | 내용 |
|------|------|
| 1번째 | 전체 게시글 조회 |
| 2번째 | 모든 댓글 + 유저 정보 INNER JOIN으로 한 번에 조회 |

---

## 6. 정리: 상황별 최적화 방법

| 상황 | 사용 메서드 | 내부 동작 |
|------|------------|-----------|
| 1→N 집계값 계산 (댓글 개수 등) | `annotate` | SQL GROUP BY |
| N→1 참조 (게시글→작성자) | `select_related` | INNER JOIN |
| 1→N 역참조 / M:N | `prefetch_related` | 별도 쿼리 + Python 결합 |
| 역참조 + 그 역참조의 참조 | `Prefetch` + `select_related` 조합 | 위 두 방식 혼합 |

> **"섣부른 최적화는 악의 근원이다" — Donald E. Knuth**  
> 데이터가 적을 때 최적화에 집중하는 것은 비효율적이다. 기능 구현을 먼저 완성하고, 성능 문제가 실제로 발생했을 때 최적화를 적용하는 순서를 지키는 것이 중요하다.

---

## 💡 한 줄 요약
> N+1 Problem은 `annotate`(집계), `select_related`(FK 참조), `prefetch_related`(역참조/M:N)로 해결하며, 복잡한 관계는 `Prefetch` 객체로 조합해 쿼리 횟수를 크게 줄일 수 있다.

## ❓ 더 찾아볼 것
- Django Debug Toolbar 설치 및 설정 방법
- `annotate`에서 사용할 수 있는 집계 함수 종류 (`Sum`, `Avg`, `Max`, `Min`)
- `only()`와 `defer()`를 활용한 필드 단위 쿼리 최적화
