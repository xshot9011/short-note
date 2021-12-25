# README

# Important Part

## removing migration file

1. run

```bash
find . -path “*/migrations/*.py” -not -name “__init__.py” -delete
find . -path “*/migrations/*.pyc” -delete
```

2. drop database

3. re-make migrations and migrate

## Common command

```bash
django-admin startproject [project’s name]
python manage.py startapp [app’s name]  # don't forget to add [app's name] to installed_apps 
python manage.py runserver
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
python manage.py shell
python manage.py collectstatic
```

## Environment file

first install

```bash
pip install python-decouple
```

in .env file

```
KEY=VALUE
```

using

```python
from decouple import config

SECRET_KEY = config('SECRET_KEY')
```

## models.py

```python
def calculate_age(born):
	today = date.today()
	return today.year - born.year - ((today.month, today.day) < (born.month, born.day))

@deconstructible
class MinAgeValidator(BaseValidator):
	compare = lambda self, a, b: calculate_age(a) < b
	message = _("Age must be at least %(limit_value)d.")
	code = 'min_age'
```

```python
from django.db import models
from django.auth.contrib.models import User
from django.core.validators import MinValueValidator, MaxValueValidator # just try
from django.utils.translation import gettext_lazy as _

#MVC's design fat at model thin at controller
class AbstractTimeStamp(models.Model):
	created = models.DateTimeField(auto_now_add=True)    
	updated = models.DateTimeField(auto_now=True)   	

	class Meta:
		abstract = True
		permissions = [('can_deliver_pizzas', 'Can deliver pizzas')]
				     	#(permission_code, human_readable_permission_name)
		unique_together = [['driver', 'restaurant']]
		indexes = [models.Index(fields=['last_name', 'first_name']),
					models.Index(fields=['first_name'], name='first_name_idx'),]
					# index_together may be deprecated in future
		constraints = [models.CheckConstraint(check=model.Q(age__gte=18), name='age_gte_18')
					# constraints will raise error when run .save()

class Board(AbstractTimeStamp):
	
	class YearInSchool(models.TextChoices):  # IntegerChoices
		FRESHMAN = 'FR', _('Freshman')         # 0, _('No') or 1, _('Yes')
		SOPHOMORE = 'SO', _('Sophomore')
		JUNIOR = 'JR', _('Junior')
		SENIOR = 'SR', _('Senior')
		GRADUATE = 'GR', _('Graduate')
		
	year_in_school = models.CharField(max_length=2,
									  choices=YearInSchool.choices,
									  default=YearInSchool.FRESHMAN,)
	name = models.CharField(max_length=100)
	birth_date = models.DateField(validators=[MinAgeValidator(18), ])
	user = models.ForeignKey(User, on_delete=models.CASCADE)    
  
	def __str__(self):        
		return '<id:{}, name:{}>'.format(str(self.pk), str(self.name))
```

## admin.py

```python
from django.contrib import admin
from .models import (Board, List)

@admin.register(Board)
class BoardAdmin(admin.ModelAdmin):    
	pass

@admin.register(List)
class ListAdmin(admin.ModelAdmin):    
	list_filter = ['board_name']  # there are many field just google
```

## serializers.py

```python
from rest_framework import serializers

class OrderSerializer(serializers.ModelSerializer):
	party_id = serializers.IntegerField(min_value=1, write_only=True)
	goods_name = serializers.SerializerMethodField('get_goods_name')

	class Meta:
		model = OrderItem
		fields = [
			'party_id', 'goods_id', 'order_qty', 'price_unit', 'order_price', 'is_serve', 		 
			'created', 'goods_name',
		]]
		extra_kwargs = {
			'price_unit': {'read_only': True},
			'order_price': {'read_only': True},
			'create': {'read_only': True},
			'is_serve': {'read_only': True},
		}

	def get_goods_name(self, obj):
		return obj.[something]
```

## api.py

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

## urls.py (application)

```python
from django.urls import path, include
from rest_framework import routers
from .api import [viewset class]

router = routers.DefaultRouter()
router.register('[path with no /]', [viewset class])
...

app_name = '[app name]'

urlpatterns = [
	path('', include(router.urls)),
]
```

# Basic Setting up

## Pre-require library

```bash
pip install djangorestframework
pip install djangorestframework-simplejwt
pip install django-filter
pip install pillow
pip install psycopg2
pip install gunicorn
pip install whitenoise
pip install python-decouple

# Only NoSQL
# if you are using NoSQL don't fking run migrate before setting NoSQL database
pip install djongo
```

## Step1: settings.py

### INSTALLED_APPS

```python
INSTALLED_APPS = [    
	'[app name]',
	'rest_framework',    
	'django_filters',    
	...
]
```

### DATABASES

to s3 [database](https://testdriven.io/blog/storing-django-static-and-media-files-on-amazon-s3/) media file

```python
# up to use case
DATABASES = {    
	'default': {        
		'ENGINE': 'djongo',        
		'NAME': '[mongoDB database name]',    
	}
}

DATABASES['default'] = dj_database_url.config(default='[url]')
```

### STATIC AND MEDIA

```python
STATIC_URL = '/static/'
STATIC_ROOT_PATH = os.path.join(BASE_DIR, 'static')
STATICFILES_DIRS = [    
	os.path.join(BASE_DIR, '[ชื่อ project]/static/')
]

MEDIA_URL = '/media/'
MEDIA_ROOT_PATH = os.path.join(BASE_DIR, 'media')
```

### MIDDLEWARE

```python
MIDDLEWARE = [    
	'whitenoise.middleware.WhiteNoiseMiddleware',    
    ...
]
```

### JWT

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

#### in case of custome token

Token.py

```python
class MyTokenObtainPairSerializer(TokenObtainPairSerializer):
	@classmethod
	def get_token(cls, user):
		token = super().get_token(user)

		# Add custom claims
		token['name'] = user.name
		# ...

		return token

class MyTokenObtainPairView(TokenObtainPairView):
	serializer_class = MyTokenObtainPairSerializer
```

urls.py

```python

	path('login/token/', MyTokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('login/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
```

### RESR_FRAMEWORK

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

## Step2: urls.py

### in case of serving static file by web server

```python
from django.contrib import admin
from django.urls import path, include
from django.conf.urls.static import static
from django.conf import settings
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView
from decouple import config

urlpatterns = [    
	path(config('ADMIN_PATH'), admin.site.urls),    
	path('login/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),    
	path('login/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),    
	# app url    
	path('app/', include('board.api.urls')),
] 

if settings.DEBUG:
	urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### in case of above nothing to serve static file

```python
from django.contrib import admin
from django.urls import path, include, re_path
from django.conf import settings
from django.views.static import serve
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

urlpatterns = [    
	path('admin/', admin.site.urls),    
	path('login/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),    
	path('login/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),    
	# app url    
	path('board/', include('board.api.urls')),        
	re_path(r'^media/(?P<path>.*)$', serve, {'document_root': settings.MEDIA_ROOT}),    
	re_path(r'^static/(?P<path>.*)$', serve, {'document_root': settings.STATIC_ROOT}),
]
```

# deploy on heroku

## 1. download and install heroku cli

follow on youtube

## 2. add heorku remote to git

```bash
heroku create
```

get the remote repo’s heroku

## 3. install requirement

```bash
pip install dj-database-urlpip freeze > requirements.txt
```

## 4. add more file

filename: Procfile

```
web gunicorn [project].wsgi --log-file -
```

filename: runtime.txt

```
python-3.7.3
```

## 5. set up settings

```python
DEBUG = False
ALLOWED_HOST = ['*']

STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATIC_URL = '/static/'
```

## 6. connect free database on heroku

heroku not allow you to use sqlite on server so you need to use others, providing by heroku

```bash
heroku addons:create heroku-postgresql:hobby-dev
```

go to heroku url in your application app > setting > config var > create new config var name ‘SECRET_KEY’ and get the key’s value in settings.py

you will notice DATABASE_URL >>> [some url] copy it !

```python
# in settings.py
import dj_database_url

...
DATABASES = {    
	...
}
# put it below
DATABASES['default'] = dj_database_url.config(default='[some url]')
```

## 7. push it to server

```
git add .
git commit -m "deploy"
git push origin master
heroku config:set DISABLE_COLLECTSSTATIC=1
git push heroku master
heroku run python manage.py migrate
heroku run python manage.py createsuperuser
```