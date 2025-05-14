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


# Crie um novo APP
python3 manage.py startapp clientes

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
