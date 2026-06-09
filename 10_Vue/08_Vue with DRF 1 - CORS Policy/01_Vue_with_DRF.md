# [Vue] Vue with DRF

---

## 1. 프로젝트 개요

Vue와 DRF(Django REST Framework)를 연결하는 실습 프로젝트를 3일에 걸쳐 진행한다.

| 일차 | 내용 |
|------|------|
| 1일차 (오늘) | Vue와 DRF 간 기본적인 요청과 응답 |
| 2일차 | 인증 시스템 (회원가입, 로그인, 로그아웃) |
| 3일차 | User 커스터마이징 |

제공되는 코드는 두 가지 타입이다. 직접 따라 작성할 **스켈레톤 코드**와, 강의를 보며 참고할 **완성 코드** — 둘 다 clone해서 시작한다.

---

## 2. DRF 스켈레톤 코드 구조

제공된 `django-pjt`의 주요 파일 구성을 파악한다.

### 모델 (articles/models.py)

```python
class Article(models.Model):
    # user = models.ForeignKey(           # 내일 주석 해제
    #     settings.AUTH_USER_MODEL, on_delete=models.CASCADE
    # )
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

class User(AbstractUser):               # accounts/models.py
    pass
```

### URL 구조 (my_api/urls.py, articles/urls.py)

```python
# my_api/urls.py
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('articles.urls')),
    # path('accounts/', include('dj_rest_auth.urls')),       # 내일
    # path('accounts/signup/', include('dj_rest_auth.registration.urls')),  # 내일
]

# articles/urls.py
urlpatterns = [
    path('articles/', views.article_list),
    path('articles/<int:article_pk>/', views.article_detail),
]
```

HTTP 메서드별 동작 요약:

| URL | GET | POST | PUT | DELETE |
|-----|-----|------|-----|--------|
| `articles/` | 전체 목록 | 게시글 생성 | - | - |
| `articles/<pk>/` | 단일 조회 | - | 수정 | 삭제 |

### 시리얼라이저 (articles/serializers.py)

```python
class ArticleListSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = ('id', 'title', 'content')   # 목록용: 3개 필드만

class ArticleSerializer(serializers.ModelSerializer):
    class Meta:
        model = Article
        fields = '__all__'
        # read_only_fields = ('user',)          # 내일 주석 해제
```

### 뷰 함수 (articles/views.py)

```python
@api_view(['GET', 'POST'])
def article_list(request):
    if request.method == 'GET':
        articles = get_list_or_404(Article)
        serializer = ArticleListSerializer(articles, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = ArticleSerializer(data=request.data)
        if serializer.is_valid(raise_exception=True):
            serializer.save()
            # serializer.save(user=request.user)  # 내일
            return Response(serializer.data, status=status.HTTP_201_CREATED)

@api_view(['GET'])
def article_detail(request, article_pk):
    article = get_object_or_404(Article, pk=article_pk)
    if request.method == 'GET':
        serializer = ArticleSerializer(article)
        return Response(serializer.data)
```

> **get_object_or_404 / get_list_or_404**: 데이터가 없으면 500(서버 오류) 대신 404(Not Found)를 반환한다. 데이터가 없는 것은 서버 문제가 아니라 클라이언트 요청 문제이기 때문에 404가 올바른 응답이다.

### settings.py 주요 설정

```python
INSTALLED_APPS = [
    'articles',
    'accounts',
    'rest_framework',
    # 'rest_framework.authtoken',   # 내일
    # 'corsheaders',                # CORS 파트에서
    ...
]

MIDDLEWARE = [
    ...
    # 'corsheaders.middleware.CorsMiddleware',   # CORS 파트에서
    ...
]

# CORS_ALLOWED_ORIGINS = [           # CORS 파트에서
#     'http://127.0.0.1:5173',
#     'http://localhost:5173',
# ]
```

### DRF 서버 초기 세팅

```bash
# 가상 환경 생성 및 활성화
$ python -m venv venv
$ source venv/Scripts/activate

# 패키지 설치
$ pip install -r requirements.txt

# 마이그레이션
$ python manage.py makemigrations
$ python manage.py migrate

# Fixtures 데이터 로드
$ python manage.py loaddata articles.json

# 서버 실행
$ python manage.py runserver
```

Fixtures는 `articles/fixtures/articles.json`에 위치하며, 미리 작성된 샘플 게시글 데이터를 DB에 로드한다.

---

## 3. Vue 스켈레톤 코드 구조

제공된 `vue-project`의 구성이다. DRF와 달리 Vue 프로젝트는 **스켈레톤 코드 위에 직접 코드를 작성**하며 진행한다.

### 컴포넌트 구조

```
App (RouterView)
├── ArticleView     → ArticleList → ArticleListItem
├── DetailView
├── CreateView
├── SignUpView
└── LogInView
```

### 주요 파일

```javascript
// App.vue - RouterView와 RouterLink만 있는 껍데기
<template>
  <header><nav></nav></header>
  <RouterView />
</template>
<script setup>
import { RouterView } from 'vue-router'
</script>

// store/articles.js - Pinia store (persist 설정 포함)
import { defineStore } from 'pinia'
export const useArticleStore = defineStore('article', () => {
  return {}
}, { persist: true })

// main.js - pinia-plugin-persistedstate 등록
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'
const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)
app.use(pinia)
app.use(router)
```

### Vue 서버 세팅

```bash
$ npm install
$ npm run dev
```

---

## 4. 게시글 목록 출력 구현

### Step 1. router에 ArticleView 등록

```javascript
// router/index.js
import ArticleView from '@/views/ArticleView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'ArticleView',
      component: ArticleView
    },
  ]
})
```

### Step 2. App.vue에 RouterLink 작성

```vue
<!-- App.vue -->
<template>
  <header>
    <nav>
      <RouterLink :to="{ name: 'ArticleView' }">Articles</RouterLink>
    </nav>
  </header>
  <RouterView />
</template>
<script setup>
import { RouterView, RouterLink } from 'vue-router'
</script>
```

### Step 3. ArticleView에 ArticleList 등록

```vue
<!-- views/ArticleView.vue -->
<template>
  <div>
    <h1>Article Page</h1>
    <ArticleList />
  </div>
</template>
<script setup>
import ArticleList from '@/components/ArticleList.vue'
</script>
```

### Step 4. store에 임시 articles 데이터 작성

```javascript
// store/articles.js
const articles = ref([
  { id: 1, title: 'Article 1', content: 'Content of article 1' },
  { id: 2, title: 'Article 2', content: 'Content of article 2' },
])
return { articles }
```

### Step 5. ArticleList에서 v-for로 ArticleListItem에 props 전달

```vue
<!-- components/ArticleList.vue -->
<template>
  <div>
    <h3>Article List</h3>
    <ArticleListItem
      v-for="article in store.articles"
      :key="article.id"
      :article="article"
    />
  </div>
</template>
<script setup>
import { useArticleStore } from '@/stores/articles'
import ArticleListItem from '@/components/ArticleListItem.vue'
const store = useArticleStore()
</script>
```

### Step 6. ArticleListItem에서 props 받아 렌더링

```vue
<!-- components/ArticleListItem.vue -->
<template>
  <div>
    <h5>{{ article.id }}</h5>
    <p>{{ article.title }}</p>
    <p>{{ article.content }}</p>
    <hr>
  </div>
</template>
<script setup>
defineProps({
  article: Object
})
</script>
```

---

## 5. DRF와의 요청과 응답 — Axios 연결

임시 데이터를 제거하고 DRF 서버로부터 실제 데이터를 받아온다.

### Axios 설치

```bash
# Vue 서버 종료 후 설치, 재실행
$ npm install axios
$ npm run dev
```

### store에 Axios 설정

```javascript
// store/articles.js
import { ref } from 'vue'
import { defineStore } from 'pinia'
import axios from 'axios'

export const useArticleStore = defineStore('article', () => {
  const articles = ref([])              // 임시 데이터 제거, 빈 배열로 초기화
  const API_URL = 'http://127.0.0.1:8000'

  const getArticles = function () {
    axios({
      method: 'get',
      url: `${API_URL}/api/v1/articles/`  // 마지막 슬래시 필수!
    })
      .then(res => {
        console.log(res)
        console.log(res.data)
      })
      .catch(err => console.log(err))
  }

  return { articles, API_URL, getArticles }
}, { persist: true })
```

> **주의**: Django URL 끝의 후행 슬래시(trailing slash)는 반드시 포함해야 한다.

### ArticleView에서 onMounted로 getArticles 호출

```vue
<!-- views/ArticleView.vue -->
<script setup>
import { onMounted } from 'vue'
import { useArticleStore } from '@/stores/articles'
import ArticleList from '@/components/ArticleList.vue'

const store = useArticleStore()

onMounted(() => {
  store.getArticles()   // 컴포넌트가 마운트될 때 자동으로 데이터 요청
})
</script>
```

**onMounted**를 사용하는 이유: 컴포넌트가 DOM에 마운트되는 시점에 DRF에서 데이터를 가져와야, 화면이 렌더링될 때 항상 최신 데이터를 보여줄 수 있다.

### 결과 확인

양쪽 서버(`python manage.py runserver`, `npm run dev`)를 모두 실행하고 브라우저 콘솔(F12)을 열면 빨간 에러가 표시된다. DRF는 200 응답을 보냈지만, **브라우저가 CORS Policy로 인해 차단**한 것이다.

```
Access to XMLHttpRequest ... has been blocked by CORS policy:
No 'Access-Control-Allow-Origin' header is present...
```

→ CORS 에러 해결은 다음 강의에서 진행한다.

---

## 💡 한 줄 요약
> Vue의 Pinia store에서 Axios로 DRF에 데이터를 요청하면, 브라우저의 CORS 정책에 의해 차단된다.

## ❓ 더 찾아볼 것
- Axios 공식 문서 (Request Config — method, url, data 옵션)
- Vue Lifecycle Hooks: onMounted, onUpdated, onUnmounted 차이
- pinia-plugin-persistedstate 동작 원리 (localStorage 활용)
- Django `get_object_or_404` vs `get_list_or_404` 차이
