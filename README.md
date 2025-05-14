# workshop-django-tenants

## Instale o django e demais pacotes
pip install django django-tenants psycopg

## Iniciar novo projeto
django-admin startproject projeto_tenant

#Configure o banco de dados em setting.py
```
DATABASES = {
    'default': {
        'ENGINE': 'django_tenants.postgresql_backend',
        'NAME': 'db_principal',
        'USER': 'foo',
        'PASSWORD': 'bar',
        'HOST': 'localhost',
        'PORT': '5433',
    }
}

DATABASE_ROUTERS = (
    'django_tenants.routers.TenantSyncRouter',
)
```

## Adicione o middleware
```
MIDDLEWARE = (
    'django_tenants.middleware.main.TenantMainMiddleware',
    #...
)
```

## Crie um novo APP
python3 manage.py startapp customers

## Criar o modelo "Client"
```
from django.db import models
from django_tenants.models import TenantMixin, DomainMixin

class Client(TenantMixin):
    name = models.CharField(max_length=100)
    paid_until =  models.DateField()
    on_trial = models.BooleanField()
    created_on = models.DateField(auto_now_add=True)

    # default true, schema will be automatically created and synced when it is saved
    auto_create_schema = True

class Domain(DomainMixin):
    pass
```

## Configurar o admin
```
from django.contrib import admin
from django_tenants.admin import TenantAdminMixin

from myapp.models import Client

@admin.register(Client)
class ClientAdmin(TenantAdminMixin, admin.ModelAdmin):
        list_display = ('name', 'paid_until')
```

## Configurar os APPS
```
SHARED_APPS = (
    'django_tenants',  # mandatory
    'customers', # you must list the app where your tenant model resides in

    'django.contrib.contenttypes',

    # everything below here is optional
    'django.contrib.auth',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.admin',
)

TENANT_APPS = (
    # your tenant-specific apps
    'myapp.hotels',
    'myapp.houses',
)

INSTALLED_APPS = list(SHARED_APPS) + [app for app in TENANT_APPS if app not in SHARED_APPS]
```

## Configurar os modelos usados pelo Django-tenants
```
TENANT_MODEL = "customers.Client" # app.Model

TENANT_DOMAIN_MODEL = "customers.Domain"  # app.Model
```

## Adicione o arquivo docker-compose.yml na pasta
```
version: '3.8'

services:
  db:
    image: postgres:15
    container_name: postgress
    environment:
      POSTGRES_DB: db_principal
      POSTGRES_USER: foo
      POSTGRES_PASSWORD: bar
    ports:
      - "5433:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Rodar o container
```
docker compose up -d
```

## Gerar as migrations
```
python3 manage.py makemigrations
python3 manage.py migrate
```
