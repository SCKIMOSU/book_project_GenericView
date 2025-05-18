# DRF APIView, GenericAPIView, ViewSet

- Django REST Framework(DRF)에서 API를 만드는 3가지 방식
    - **`APIView`**, **`GenericAPIView`**, `ViewSet`은 모두 클래스 기반이지만 **구조, 역할, 사용 목적이 다름.**
- **차이점, 공통점, 용도, 예제 비교표**

---

## ✅ 1. 요약 비교표

| 항목 | `APIView` | `GenericAPIView` | `ViewSet` (특히 `ModelViewSet`) |
| --- | --- | --- | --- |
| 기반 클래스 | Django `View` | `APIView` + Generic 기능 | `GenericAPIView` + `Mixin` 조합 |
| 코드 구성 | 메서드(`get`, `post`) 직접 작성 | queryset, serializer 제공 | `.list()`, `.retrieve()` 등 이름 기반 메서드 |
| 라우팅 방식 | 수동(`urls.py`에 path 등록) | 수동 | 자동 (`router.register()`) |
| CRUD 처리 편의성 | 👎 불편 (직접 작성 필요) | 👍 `ListAPIView`, `CreateAPIView` 등 제공 | 👍👍 `ModelViewSet` 하나로 전체 CRUD 자동화 |
| 커스터마이징 | ✅ 자유로움 | ✅ 비교적 쉬움 | ✅ 커스텀 액션(`@action`) 사용 가능 |
| 대표 용도 | 간단한 커스텀 API | CRUD 중 일부만 구현할 때 | 전체 CRUD API + 자동 URL |

---

## ✅ 2. 실전 예제 비교

---

### 📌 `APIView` 예제

```python
from rest_framework.views import APIView
from rest_framework.response import Response

class HelloAPIView(APIView):
    def get(self, request):
        return Response({"message": "GET hello"})

    def post(self, request):
        return Response({"message": "POST hello"})

```

- **모든 HTTP 메서드를 직접 정의**해야 함
- 구조는 Django View와 유사

---

### 📌 `GenericAPIView` + Mixin 예제

```python
from rest_framework.generics import GenericAPIView
from rest_framework.mixins import ListModelMixin, CreateModelMixin
from .models import Book
from .serializers import BookSerializer

class BookListCreateView(GenericAPIView, ListModelMixin, CreateModelMixin):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get(self, request):
        return self.list(request)

    def post(self, request):
        return self.create(request)

```

- CRUD 중 필요한 기능만 Mixin으로 조합 가능
- **유연하지만 조금 장황**

---

### 📌 `ModelViewSet` + Router 예제

```python
from rest_framework import viewsets
from .models import Book
from .serializers import BookSerializer

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

```

```python
# urls.py
router = DefaultRouter()
router.register('books', BookViewSet)

```

- 전체 CRUD (`list`, `retrieve`, `create`, `update`, `destroy`) 자동 생성
- URL도 자동 생성 → 유지보수 효율 최고

---

## ✅ 3. 언제 무엇을 써야 할까?

| 상황 | 추천 방식 |
| --- | --- |
| API를 아주 세밀하게 직접 제어하고 싶을 때 | `APIView` |
| 단일 기능 (예: 목록 조회, 단일 등록)만 필요할 때 | `GenericAPIView` + Mixin |
| 전체 CRUD API를 빠르게 만들고 싶을 때 | `ModelViewSet` + `Router` |
| 커스텀 액션을 URL에 추가하고 싶을 때 | `ViewSet` + `@action` |
| Django REST 구조에 익숙하지 않은 초급자 | `GenericAPIView` 계열 추천 |
| Swagger 자동 문서화, RESTful 설계 중시 | `ViewSet` + Router 추천 |

---

## ✅ 4. 코드 복잡도/자동화 정도 비교

```
많이 작성해야 함 (유연) ←───────────── 자동화됨 (빠름)
      APIView   →   GenericAPIView   →   ModelViewSet

```

---

## ✅ 결론 요약

| View 종류 | 특징 및 사용처 |
| --- | --- |
| `APIView` | 가장 유연함. 완전 수동으로 제어할 때 사용 |
| `GenericAPIView` | CRUD 기능을 일부 조합해서 쓰고 싶을 때 |
| `ModelViewSet` | CRUD 전체 + 라우팅 자동 + 가장 추천되는 구조 |

---

## **Generic View**

Django REST framework의 **주요 Generic View**들을 사용하여 `Book` 모델에 대한 **CRUD API** 구성

---

## ✅ 0. 전제: `Book` 모델

```python
# models.py
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=100)
    published_year = models.IntegerField()
    is_available = models.BooleanField(default=True)

    def __str__(self):
        return self.title

```

---

## ✅ 1. Serializer

```python
# serializers.py
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = '__all__'

```

---

## ✅ 2. View (GenericAPIView + 다양한 뷰)

```python
# views.py
from rest_framework import generics
from .models import Book
from .serializers import BookSerializer

# 1. 목록 조회 (GET) + 생성 (POST)
class BookListCreateView(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# 2. 단일 조회 (GET)
class BookRetrieveView(generics.RetrieveAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# 3. 수정 (PUT/PATCH)
class BookUpdateView(generics.UpdateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# 4. 삭제 (DELETE)
class BookDestroyView(generics.DestroyAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# 5. 단일 조회 + 수정 + 삭제
class BookRetrieveUpdateDestroyView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

```

---

## ✅ 3. URL 설정

```python
# urls.py
from django.urls import path
from .views import (
    BookListCreateView,
    BookRetrieveView,
    BookUpdateView,
    BookDestroyView,
    BookRetrieveUpdateDestroyView,
)

urlpatterns = [
    path('api/books/', BookListCreateView.as_view(), name='book-list-create'),
    path('api/books/<int:pk>/', BookRetrieveView.as_view(), name='book-retrieve'),
    path('api/books/<int:pk>/update/', BookUpdateView.as_view(), name='book-update'),
    path('api/books/<int:pk>/delete/', BookDestroyView.as_view(), name='book-delete'),
    path('api/books/<int:pk>/detail/', BookRetrieveUpdateDestroyView.as_view(), name='book-detail'),
]

```

---

## ✅ 4. 주요 Generic View 요약

| 뷰 클래스 | 지원 HTTP 메서드 | 설명 |
| --- | --- | --- |
| `ListCreateAPIView` | GET, POST | 전체 조회 + 생성 |
| `RetrieveAPIView` | GET | 단일 조회 |
| `UpdateAPIView` | PUT, PATCH | 단일 수정 |
| `DestroyAPIView` | DELETE | 단일 삭제 |
| `RetrieveUpdateDestroyAPIView` | GET, PUT/PATCH, DELETE | 단일 조회 + 수정 + 삭제 |

---

## ✅ 5. Postman 테스트 예시

### ➤ `GET /api/books/`

```json
[
  {
    "id": 1,
    "title": "DRF 기초",
    "author": "홍길동",
    "published_year": 2023,
    "is_available": true
  }
]

```

### ➤ `POST /api/books/`

```json
{
  "title": "DRF 실전",
  "author": "이몽룡",
  "published_year": 2024,
  "is_available": false
}

```

---

## `ViewSet`

- `ViewSet`은 Django REST Framework(DRF)에서 제공하는 강력한 클래스 기반 뷰
- 하나의 클래스에 CRUD 로직을 모아두고,
- 라우터(Router)를 통해 자동으로 URL 매핑까지 처리할 수 있도록 돕는 구조.

---

## ✅ ViewSet이란?

> ViewSet은 APIView나 GenericAPIView와 달리, list(), create(), retrieve(), update() 등 HTTP 메서드에 대응하는 로직만 작성하면 되고, URL 라우팅은 Router가 자동으로 처리.
> 

---

## ✅ ViewSet의 핵심 특징

| 특징 | 설명 |
| --- | --- |
| CRUD 처리 통합 | 하나의 클래스에서 목록 조회, 생성, 수정, 삭제 등을 모두 정의 가능 |
| URL 자동 매핑 지원 | `Router`를 사용하면 URLConf를 작성하지 않아도 자동으로 라우팅 |
| 메서드 이름 기반 매핑 | `.list()`, `.create()` 등 메서드 이름으로 HTTP 메서드 매핑됨 |
| `@action` 데코레이터 지원 | 커스텀 액션 추가 가능 (예: 좋아요 버튼 등) |

---

## ✅ ViewSet 종류

DRF는 상황에 맞게 다양한 ViewSet을 제공:

| 클래스 | 특징 |
| --- | --- |
| `ViewSet` | 가장 기본 ViewSet, CRUD 수동 처리 |
| `GenericViewSet` | `GenericAPIView` + Mixin 조합용 |
| `ModelViewSet` | 가장 많이 사용, CRUD 자동 처리 |
| `ReadOnlyModelViewSet` | 목록 + 단일 조회만 지원 |

---

## ✅ `ModelViewSet` 예시 (CRUD 자동 처리)

```python
# views.py
from rest_framework import viewsets
from .models import Book
from .serializers import BookSerializer

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

```

> .list(), .retrieve(), .create(), .update(), .destroy() 가 자동 처리됨
> 

---

## ✅ URL 연결 (Router 사용)

```python
# urls.py
from rest_framework.routers import DefaultRouter
from .views import BookViewSet

router = DefaultRouter()
router.register('books', BookViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]

```

---

## ✅ 자동 생성되는 URL 예시

| URL Path | HTTP Method | ViewSet 메서드 |
| --- | --- | --- |
| `/api/books/` | GET | `list()` |
| `/api/books/` | POST | `create()` |
| `/api/books/{pk}/` | GET | `retrieve()` |
| `/api/books/{pk}/` | PUT/PATCH | `update()` |
| `/api/books/{pk}/` | DELETE | `destroy()` |

---

## ✅ 커스텀 동작 추가 (`@action` 사용)

```python
from rest_framework.decorators import action
from rest_framework.response import Response

class BookViewSet(viewsets.ModelViewSet):
    ...

    @action(detail=True, methods=['post'])
    def toggle(self, request, pk=None):
        book = self.get_object()
        book.is_available = not book.is_available
        book.save()
        return Response({'is_available': book.is_available})

```

➡ 자동 URL: `POST /api/books/{pk}/toggle/`

---

## ✅ 요약

| 항목 | 설명 |
| --- | --- |
| 목적 | CRUD API를 한 클래스에서 처리 |
| 장점 | 코드 간결, 유지보수 편리, 라우팅 자동화 |
| 필수 구성 | `queryset`, `serializer_class`, `Router` 등록 |
| 확장성 | `@action`으로 커스텀 메서드 쉽게 추가 가능 |

---

## Django REST Framework(DRF)의 **Router**

- Django REST Framework(DRF)의 **Router**는 `ViewSet`과 함께 사용되어 **RESTful API의 URL 패턴을 자동으로 생성**해주는 강력한 도구.

---

## ✅ 1. Router란?

- `ViewSet`은 `get`, `post`, `put`, `delete` 메서드를 직접 URL에 매핑하지 않음
- `Router`는 이런 `ViewSet`을 보고 적절한 URL들을 자동으로 생성
- `urls.py`에서 `router.register()`를 통해 하나의 라인으로 CRUD API 등록 가능

---

## ✅ 2. 기본 사용법

```python
# urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import BookViewSet

router = DefaultRouter()
router.register('books', BookViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]

```

---

## ✅ 3. 자동으로 생성되는 URL 목록

- 예: `router.register('books', BookViewSet)` 기준

| HTTP Method | URL Path | Action | ViewSet 메서드 |
| --- | --- | --- | --- |
| GET | `/api/books/` | 목록 조회 | `.list()` |
| POST | `/api/books/` | 새 객체 생성 | `.create()` |
| GET | `/api/books/{pk}/` | 단일 조회 | `.retrieve()` |
| PUT | `/api/books/{pk}/` | 전체 수정 | `.update()` |
| PATCH | `/api/books/{pk}/` | 부분 수정 | `.partial_update()` |
| DELETE | `/api/books/{pk}/` | 삭제 | `.destroy()` |

---

## ✅ 4. `DefaultRouter` vs `SimpleRouter`

| Router 종류 | 설명 |
| --- | --- |
| `DefaultRouter` | 기본 RESTful URL + 자동으로 `/`에 브라우저 UI root 추가 |
| `SimpleRouter` | UI root 없이 순수 URL 패턴만 생성 |

```python
# 차이 예시
DefaultRouter() → `/api/` 진입 시 API root 페이지 있음
SimpleRouter()  → 없음

```

---

## ✅ 5. `.register()` 파라미터

```python
router.register(prefix, viewset, basename=None)

```

| 파라미터 | 설명 |
| --- | --- |
| `prefix` | URL의 접두사 (예: `'books'` → `/books/`) |
| `viewset` | 연결할 ViewSet 클래스 |
| `basename` | 뷰셋에 `queryset`이 없을 때 수동으로 이름 지정 필요 |

---

## ✅ 6. 커스터마이징 예시 (lookup, extra actions 등)

```python
# views.py
from rest_framework.decorators import action
from rest_framework.response import Response

class BookViewSet(viewsets.ModelViewSet):
    ...

    @action(detail=True, methods=['post'])
    def toggle_availability(self, request, pk=None):
        book = self.get_object()
        book.is_available = not book.is_available
        book.save()
        return Response({"available": book.is_available})

```

```python
# 자동 등록됨
POST /api/books/1/toggle_availability/

```

---

## ✅ 전체 흐름 그림

```
                ┌──────────────────────────┐
URLs.py  ──────► router.register("books", BookViewSet)
                └────────────┬─────────────┘
                             ▼
     /api/books/                → list(), create()
     /api/books/<pk>/           → retrieve(), update(), destroy()

```

---

## ✅ 장점 요약

| 기능 | 설명 |
| --- | --- |
| URL 자동 생성 | CRUD URL을 자동으로 생성해줌 |
| ViewSet과 강력한 조합 | 여러 메서드가 하나의 클래스로 관리됨 |
| 유지보수 용이 | URL 수동 등록이 필요 없고 구조가 깔끔함 |
| `@action`으로 확장 기능 쉽게 구현 | 좋아요, 북마크 등 커스텀 액션 추가 용이 |

---

## ✅ 요약

- `Router`는 `ViewSet`의 URL 라우팅을 자동으로 처리
- `DefaultRouter()`를 사용하면 API Root UI 페이지까지 제공
- `router.register()` 한 줄로 전체 CRUD API가 구성됨

---
