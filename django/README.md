# basic setup

note: env with python3.7 is required
```bash

```
## command

```bash
django-admin startproject [project’s name]
python .\manage.py startapp [app’s name]
python .\manage.py runserver
python .\manage.py makemigrations
python .\manage.py migrate
python .\manage.py createsuperuser
python .\manage.py collectstatic
```
## step 1:

install basic lib

```bash
pip install django
pip install djangorestframework
pip install djangorestframework-simplejwt
pip install django-filter
pip install pillow
pip install psycopg2
```

## step 2:

config in settings.py

```python
INSTALLED_APPS = [
	'rest_framework',
	...
]
...
...
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'manager/static/')
]

MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'

# to use different setting from github when deploy
try:
    from .local_settings import *
except ImportError:
    pass
```

## step 3:

### urls.py

```python
from django.urls import path, include
from django.conf import settings
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView
...
...
urlpatterns = [
	...
	path('login/token/', Token.as_view(), name='token_obtain_pair'),
	path('login/token/refresh/', TokenRefreshView.as_view(),name='token_refresh'),
]+ static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### serializers.py

```python
from rest_framework import serializers
```

### views.py

```python
from rest_framework import viewsets, status, mixins
from rest_framework.response import Response
from rest_framework.views import APIView
```

### urls.py

```python
from django.urls import path, include
from rest_framework import routers
...
...
router = routers.DefaultRouter()
app_name = '[app name]'
...
...
urlpatterns = [
    path('', include(router.urls)),
]
```
