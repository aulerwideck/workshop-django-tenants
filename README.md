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

## Criar o tenant principal
```
from customers.models import Client, Domain

# create your public tenant
tenant = Client(schema_name='public',
                name='Primario',
                paid_until='2025-12-31',
                on_trial=False)
tenant.save()

# Adicionar um domínio para acessar o tenant principal
domain = Domain()
domain.domain = 'teste.localhost'
domain.tenant = tenant
domain.is_primary = True
domain.save()
```

## Criar um novo tenant
```
from customers.models import Client, Domain

# create your public tenant
tenant = Client(schema_name='secundario',
                name='Secundario',
                paid_until='2025-12-31',
                on_trial=False)
tenant.save()

# Adicionar um domínio para acessar o tenant principal
domain = Domain()
domain.domain = 'secundario.localhost'
domain.tenant = tenant
domain.is_primary = True
domain.save()
```


# Separando as informações por Tenant

## Configurando um novo app
python manage.py startapp core

## Configure o app
```
TENANT_APPS = (
    ...,
    'core',
)
```

## View para exibir o tenant
Em core/views.py:
```
from django.shortcuts import render
from django_tenants.utils import get_tenant_model

def tenant_info(request):
    tenant = request.tenant
    return render(request, 'core/tenant_info.html', {'tenant_name': tenant.name})
```

## Template
Crie o template: core/templates/core/tenant_info.html:
```
from django.shortcuts import render
from django_tenants.utils import get_tenant_model

def tenant_info(request):
    tenant = request.tenant
    return render(request, 'core/tenant_info.html', {'tenant_name': tenant.name})
```

## URL
No core/urls.py (crie este arquivo se ainda não existir):
```
from django.urls import path
from .views import tenant_info

urlpatterns = [
    path('', tenant_info, name='tenant_info'),
]

```
E em projeto_tenant/urls.py:
```
from django.urls import path, include

urlpatterns = [
    path('', include('core.urls')),
]
```

## Acesse os dominios:
http://localhost:8000/
http://secundario.localhost:8000/
