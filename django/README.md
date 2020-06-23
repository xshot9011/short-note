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
pip install gunicorn
pip install whitenoise
```

## step 2:

config in settings.py

```python
INSTALLED_APPS = [
	'rest_framework',
	'django_filters',
	...
]
...
...
MIDDLEWARE = [
    # third-party
    'whitenoise.middleware.WhiteNoiseMiddleware',
    # default
    ...
]
...
...
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, '[ชื่อapp]/static/')
]

MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'

# to use different setting from github when deploy
try:
    from .local_settings import *
except ImportError:
    pass
```
### rest_framework

```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}
```

### basic simple jwt

```python
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=100000),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
    'ROTATE_REFRESH_TOKENS': False,
    'BLACKLIST_AFTER_ROTATION': True,

    'ALGORITHM': 'HS512',
    'SIGNING_KEY': SECRET_KEY,
    'VERIFYING_KEY': None,
    'AUDIENCE': None,
    'ISSUER': None,

    'AUTH_HEADER_TYPES': ('Bearer',),
    'USER_ID_FIELD': 'id',
    'USER_ID_CLAIM': 'user_id',

    'AUTH_TOKEN_CLASSES': ('rest_framework_simplejwt.tokens.AccessToken',),
    'TOKEN_TYPE_CLAIM': 'token_type',

    'JTI_CLAIM': 'jti',

    'SLIDING_TOKEN_REFRESH_EXP_CLAIM': 'refresh_exp',
    'SLIDING_TOKEN_LIFETIME': timedelta(minutes=5),
    'SLIDING_TOKEN_REFRESH_LIFETIME': timedelta(days=1),
}
```

## step 3:

### urls.py

```python
from django.contrib import admin
from django.urls import path, include
from django.conf.urls.static import static
from django.conf import settings
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('login/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('login/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    # app url
    path('app/', include('board.api.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### serializers.py

```python
from rest_framework import serializers

class OrderSerializer(serializers.ModelSerializer):
    party_id = serializers.IntegerField(min_value=1, write_only=True)
    goods_name = serializers.SerializerMethodField('get_goods_name')

    class Meta:
        model = OrderItem
        fields = ['party_id', 'goods_id', 'order_qty', 'price_unit', 'order_price', 'is_serve', 'created',
                  'goods_name']
        extra_kwargs = {
            'price_unit': {'read_only': True},
            'order_price': {'read_only': True},
            'create': {'read_only': True},
            'is_serve': {'read_only': True},
        }

    def get_goods_name(self, obj):
        return obj.[something]
```

### api.py

```python
from rest_framework import viewsets, status, mixins
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework.decorators import action
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework import filters

class MyOrderViewSet(mixins.RetrieveModelMixin,
                     mixins.ListModelMixin,
                     viewsets.GenericViewSet):
    queryset = Party.objects.all()
    serializer_class = MyOrderSerializer
    filter_backends = [filters.SearchFilter, DjangoFilterBackend]
    search_fields = ['party_name', ]
    filterset_fields = ['is_active', ]
```

### urls.py

```python
from django.urls import path, include
from rest_framework import routers
from .api import [viewset class]
...
...
router = routers.DefaultRouter()
router.register('[path with no /]', [viewset class])
app_name = '[app name]'
...
...
urlpatterns = [
    path('', include(router.urls)),
]
```
