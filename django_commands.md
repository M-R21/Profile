# Django REST API Design Cheat Sheet

This cheat sheet serves as a guide to designing and implementing a RESTful API using Django and Django REST Framework (DRF). It covers the fundamental concepts, best practices, and common patterns you should follow when building REST APIs.

## Table of Contents

1. [Getting Started](#getting-started)
    - [Installation](#installation)
    - [Creating a Django Project](#creating-a-django-project)
    - [Setting Up Django REST Framework](#setting-up-django-rest-framework)
2. [Basic Concepts](#basic-concepts)
    - [Models](#models)
    - [Serializers](#serializers)
    - [Views](#views)
    - [URLs](#urls)
3. [Authentication & Authorization](#authentication--authorization)
    - [Token Authentication](#token-authentication)
    - [Session Authentication](#session-authentication)
    - [Custom Authentication](#custom-authentication)
4. [Pagination](#pagination)
5. [Filtering and Searching](#filtering-and-searching)
6. [Throttling](#throttling)
7. [Versioning](#versioning)
8. [Error Handling](#error-handling)
9. [Best Practices](#best-practices)
10. [Advanced Topics](#advanced-topics)
    - [ViewSets and Routers](#viewsets-and-routers)
    - [Permissions](#permissions)
    - [Custom Responses](#custom-responses)
    - [Swagger/OpenAPI Documentation](#swaggeropenapi-documentation)
11. [Deployment](#deployment)
    - [Deploying with Docker](#deploying-with-docker)
    - [Deploying to Heroku](#deploying-to-heroku)

---

## Getting Started

### Installation

1. **Install Django and Django REST Framework:**

    ```bash
    pip install django djangorestframework
    ```

2. **Create a new Django project:**

    ```bash
    django-admin startproject myproject
    cd myproject
    ```

3. **Create a new Django app:**

    ```bash
    python manage.py startapp myapp
    ```

4. **Add `rest_framework` to `INSTALLED_APPS` in `settings.py`:**

    ```python
    INSTALLED_APPS = [
        ...
        'rest_framework',
        'myapp',
    ]
    ```

### Creating a Django Project

1. **Create the project structure:**

    ```bash
    django-admin startproject myproject
    ```

2. **Run migrations and create the database:**

    ```bash
    python manage.py migrate
    ```

3. **Start the development server:**

    ```bash
    python manage.py runserver
    ```

### Setting Up Django REST Framework

1. **Create a `views.py` file in your app directory.**

2. **Create a `serializers.py` file in your app directory.**

3. **Add basic settings in `settings.py`:**

    ```python
    REST_FRAMEWORK = {
        'DEFAULT_AUTHENTICATION_CLASSES': [
            'rest_framework.authentication.SessionAuthentication',
            'rest_framework.authentication.TokenAuthentication',
        ],
        'DEFAULT_PERMISSION_CLASSES': [
            'rest_framework.permissions.IsAuthenticated',
        ],
        'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
        'PAGE_SIZE': 10,
    }
    ```

## Basic Concepts

### Models

Define your data structure using Django models.

```python
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=255)
    author = models.CharField(max_length=255)
    published_date = models.DateField()

    def __str__(self):
        return self.title
```

### Serializers

Serializers convert complex data types like querysets and model instances to native Python datatypes that can then be easily rendered into JSON or other content types.

```python
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = '__all__'
```

### Views

Views handle the logic behind the API endpoints.

```python
from rest_framework import generics
from .models import Book
from .serializers import BookSerializer

class BookListCreate(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

### URLs

Map the API views to specific URLs.

```python
from django.urls import path
from .views import BookListCreate

urlpatterns = [
    path('books/', BookListCreate.as_view(), name='book-list-create'),
]
```

## Authentication & Authorization

### Token Authentication

Use Token Authentication for stateless session management.

1. **Install `djangorestframework.authtoken`:**

    ```bash
    pip install djangorestframework-authtoken
    ```

2. **Add to `INSTALLED_APPS`:**

    ```python
    INSTALLED_APPS = [
        ...
        'rest_framework.authtoken',
    ]
    ```

3. **Run migrations:**

    ```bash
    python manage.py migrate
    ```

4. **Add Token Authentication to settings:**

    ```python
    REST_FRAMEWORK = {
        'DEFAULT_AUTHENTICATION_CLASSES': (
            'rest_framework.authentication.TokenAuthentication',
        ),
    }
    ```

5. **Obtain Auth Token via API:**

    ```python
    from rest_framework.authtoken.views import obtain_auth_token

    urlpatterns = [
        path('api-token-auth/', obtain_auth_token),
    ]
    ```

### Session Authentication

Session Authentication is useful for web clients that have session management enabled.

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.SessionAuthentication',
    ),
}
```

### Custom Authentication

You can implement custom authentication classes by subclassing `BaseAuthentication`.

```python
from rest_framework.authentication import BaseAuthentication

class CustomAuthentication(BaseAuthentication):
    def authenticate(self, request):
        # Custom authentication logic here
        return (user, None)  # Return a user or None if authentication fails
```

## Pagination

Use pagination to control the amount of data sent in a single response.

### Basic Pagination Setup

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}
```

### Custom Pagination Class

```python
from rest_framework.pagination import PageNumberPagination

class CustomPagination(PageNumberPagination):
    page_size = 5
    page_size_query_param = 'page_size'
    max_page_size = 100
```

## Filtering and Searching

### Django Filter

Install and set up Django Filter for filtering querysets.

```bash
pip install django-filter
```

```python
INSTALLED_APPS = [
    ...
    'django_filters',
]
```

```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': (
        'django_filters.rest_framework.DjangoFilterBackend',
    ),
}
```

### Basic Filtering Example

```python
from django_filters import rest_framework as filters

class BookFilter(filters.FilterSet):
    author = filters.CharFilter(lookup_expr='iexact')
    
    class Meta:
        model = Book
        fields = ['author', 'published_date']
```

### Searching

Use `SearchFilter` to search across fields.

```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': (
        'rest_framework.filters.SearchFilter',
    ),
    'SEARCH_PARAM': 'q',
}
```

### Basic Search Example

```python
from rest_framework import generics
from rest_framework.filters import SearchFilter
from .models import Book
from .serializers import BookSerializer

class BookListCreate(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filter_backends = [SearchFilter]
    search_fields = ['title', 'author']
```

## Throttling

Throttle requests to prevent abuse and rate-limit users.

### Basic Throttling Setup

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'user': '100/day',
        'anon': '20/day',
    }
}
```

### Custom Throttling

```python
from rest_framework.throttling import UserRateThrottle

class BurstRateThrottle(UserRateThrottle):
    rate = '5/minute'
```

## Versioning

Version your API to manage changes over time.

### URL Path Versioning

```python
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.URLPathVersioning',
}
```

### Example URL Versioning

```python
urlpatterns = [
    path('v1/books/', BookListCreate.as_view(), name='book-list-create'),
]
```

## Error Handling

### Custom Exception Handling

Use Django's exception handling to customize error responses.

```python
from rest_framework.views import exception_handler

def custom_exception_handler(exc, context):
    response = exception_handler(exc, context)

    if response is not None:
        response.data['status_code'] = response.status_code

    return response
```

### Update Settings for Custom Exceptions

```python
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'myproject.myapp.utils.custom_exception_handler'
}
```

## Best Practices

- **Use ViewSets and Routers**: Simplify URL routing by using ViewSets and Routers.
- **Utilize Django's ORM effectively**: Avoid N+1 query problems, use `select_related` and `prefetch_related`.
- **Pagination**: Always paginate list

 responses.
- **Validation**: Use serializer validation methods (`validate_fieldname` and `validate`) for data integrity.
- **Security**: Implement SSL, strong authentication, and authorization.
- **Documentation**: Always document your API using tools like Swagger or Redoc.

## Advanced Topics

### ViewSets and Routers

ViewSets group common actions together, while Routers automatically generate URL patterns.

```python
from rest_framework import viewsets
from rest_framework.routers import DefaultRouter

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

router = DefaultRouter()
router.register(r'books', BookViewSet)

urlpatterns = router.urls
```

### Permissions

Control access to your API using permissions.

```python
from rest_framework.permissions import IsAuthenticated

class BookViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]
    ...
```

### Custom Responses

Customize the response format for your API.

```python
from rest_framework.response import Response

def custom_response(data=None, status=200):
    return Response({
        'status': status,
        'data': data
    }, status=status)
```

### Swagger/OpenAPI Documentation

Use tools like `drf-yasg` or `django-rest-swagger` to automatically generate API documentation.

1. **Install `drf-yasg`:**

    ```bash
    pip install drf-yasg
    ```

2. **Add to `INSTALLED_APPS`:**

    ```python
    INSTALLED_APPS = [
        ...
        'drf_yasg',
    ]
    ```

3. **Generate Swagger Docs:**

    ```python
    from rest_framework import permissions
    from drf_yasg.views import get_schema_view
    from drf_yasg import openapi

    schema_view = get_schema_view(
        openapi.Info(
            title="My API",
            default_version='v1',
        ),
        public=True,
        permission_classes=(permissions.AllowAny,),
    )

    urlpatterns = [
        path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
    ]
    ```

## Deployment

### Deploying with Docker

1. **Create a `Dockerfile`:**

    ```Dockerfile
    FROM python:3.9
    ENV PYTHONUNBUFFERED 1
    WORKDIR /code
    COPY requirements.txt /code/
    RUN pip install -r requirements.txt
    COPY . /code/
    ```

2. **Create a `docker-compose.yml`:**

    ```yaml
    version: '3'

    services:
      db:
        image: postgres
        environment:
          POSTGRES_DB: mydb
          POSTGRES_USER: myuser
          POSTGRES_PASSWORD: mypassword
      web:
        build: .
        command: python manage.py runserver 0.0.0.0:8000
        volumes:
          - .:/code
        ports:
          - "8000:8000"
        depends_on:
          - db
    ```

3. **Build and run the containers:**

    ```bash
    docker-compose up --build
    ```

### Deploying to Heroku

1. **Install Heroku CLI and log in:**

    ```bash
    heroku login
    ```

2. **Create a `Procfile`:**

    ```Procfile
    web: gunicorn myproject.wsgi
    ```

3. **Push to Heroku:**

    ```bash
    git add .
    git commit -m "Deploy"
    heroku create
    git push heroku master
    ```

4. **Set up the database:**

    ```bash
    heroku run python manage.py migrate
    ```

---

This cheat sheet provides a structured and detailed approach to creating RESTful APIs using Django and Django REST Framework. By following these guidelines and best practices, you'll be able to design robust, scalable, and secure APIs.