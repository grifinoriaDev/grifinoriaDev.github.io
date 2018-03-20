---
layout: post
title: "Criando uma aplicação django Parte 1"
description: Vamos criar uma aplicação django de tarefas para servir de api REST
image: '/assets/img/django.jpg'
category: 'django'
---

# Aplicação

Olá pessoal, hoje vou mostrar alguns macetes que utilizo no meu dia a dia para criar uma aplicação.

A aplicação será um controle de tarefas com as seguintes premissas:

1. Adicionar, Editar e Remover Tarefa
2. As tarefas serão por usuários
3. As tarefas terão status Pendente, Resolvida e Em Execução
4. As tarefas terão um agrupamento
5. As tarefas terão um prazo limite

Vamos separar nosso projeto em 3 etapas

1. Criar o backend (UHULLL vamos djangar!!!)
2. Terminar a API
3. Criar o frontEnd (EEEEEEEEBAAAA vamos vuezar!!!)
4. Hospedar (Vixe, vamos configurar um servidor ubuntu na munheca para essa meleca)

## Vamos ao que interessa, Django

Vou desenvolver esse projeto no windows, porém, é muito parecido o que vamos fazer para os outros sistemas operacionais (Mac e Linux), com excessão a instalação do python.

### Instalando o python

[Sugiro ler isso aqui](https://python.org.br/instalacao-windows/)

### Criando o direotrio do projeto

Vamos criar um diretório em um lugar no qual você tenha permissões e possa fazer merda a vontade.

```bash
cd %userprofile%
mkdir tarefas
cd tarefas
mkdir api
cd api
```

Criamos um diretorio ná pasta do seu usuário com nome de `tarefas` e dentro dele criamos um subdiretorio chamado de `api`.

> É muito importante organizar isso ai, pois é a estrutura do projeto


### Vitualenv

Dentro dessa pasta `api` vamos iniciar o virtualenv do python. Simples.

```bash
python -m venv .tarefasEnv
```

Feito isso, vamos iniciar nosso vitualenv e começar a fazer as instalações

Windows:  
```prompt
.tarefasEnv\Scripts\activate.bat
```

Mac ou Linux
```bash
source .tarefasEnv/bin/activate
```

Se tudo tiver correto, você tera a seguinte saida no terminal:

```
(.tarefasEnv) C:\Users\SEUUSUARIO\tarefas\api>
```

### Pipenv

Para deixar as coisas mais profissionais e interessantes, vamos utilizar o `pipenv` como gerenciador de pacotes, então, vamos rodar o comando

```bash
pip install pipenv
```

### Pacotes

Agora estamos quase prontos para iniciar o projeto. Vamos instalar alguns pacotes do django e começar a por a mão na massa

Pacotes para produção
```bash
pipenv install django django-extensions djangorestframework django-rest-auth python-decouple dj-database-url django-cors-headers djangorestframework-jwt django-rest-swagger
```

Pacotes para desenvolvimento
```bash
pipenv install notebook --dev
```

Feito tudo certo, vamos ter o arquivo `Pipfile` dentro da pasta api

```python
[[source]]

url = "https://pypi.python.org/simple"
verify_ssl = true
name = "pypi"


[packages]

django = "*"
django-extensions = "*"
djangorestframework = "*"
django-rest-auth = "*"
python-decouple = "*"
dj-database-url = "*"
django-cors-headers = "*"
djangorestframework-jwt = "*"
django-rest-swagger = "*"


[dev-packages]

notebook = "*"
```

### Iniciando o projeto

Agora, vamos criar o projeto.

```python
django-admin startproject api .
cd api
django-admin startapp core
```

Para testar, volte para a pasta raiz, onde esta o arquivo `manage.py` e vamos iniciar o servidor de desenvolvimento

```bash
python manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).

You have 14 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
March 20, 2018 - 16:04:47
Django version 2.0.3, using settings 'api.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

Maravilha, agora vamos escrever um pouco de código.

#### Configurando o Settings

Entenda que o settings.py do seu projeto possui algumas informações sensíveis, como senha do banco de dados, algumas informações que serão diferentes em produção e desenvolvimento. Para isso, vamos usar o python-decouple pra facilitar nossa vida. O que fazemos é extrair essas informações para um arquivo `.env` que irá instanciar isso como variável de ambiente e utilizar essas variáveis no django.

![supresa](/assets/img/supresa.gif)

Dentro do arquivo .env irão ficar as seguintes configs
```bash
SECRET_KEY=HAHAHAHAHAHAHA
DEBUG=True
ALLOWED_HOSTS=127.0.0.1, .localhost
DATABASE_URL=sqlite:///tarefas.sqlite3
LANGUAGE_CODE=pt-br
TIME_ZONE=America/Sao_Paulo
```

e o settings.py

```python
import os
from decouple import config, Csv
from dj_database_url import parse

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', cast=bool, default=False)
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv(), default=[])

DJANGO_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

THIRD_APPS = [
    'django_extensions',
    'corsheaders',
    'rest_framework',
    'rest_auth',
    'rest_framework_jwt',
]

LOCAL_APPS = [
    'api.core',
]

INSTALLED_APPS = DJANGO_APPS + THIRD_APPS + LOCAL_APPS

CORS_ORIGIN_ALLOW_ALL = True

REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': (
        'django_filters.rest_framework.DjangoFilterBackend',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ),
    'DEFAULT_PARSER_CLASSES': (
        'rest_framework.parsers.JSONParser',
    ),
    'COERCE_DECIMAL_TO_STRING': False,
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
}

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'api.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'api.wsgi.application'

DATABASES = dict()
DATABASES['default'] = config('DATABASE_URL', cast=parse, default='sqlite:///' + os.path.join(BASE_DIR, 'db.sqlite3'))

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

LANGUAGE_CODE = config('LANGUAGE_CODE', default='en-us')

TIME_ZONE = config('TIME_ZONE', default='UTC')

USE_I18N = True

USE_L10N = True

USE_TZ = True

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

Para desencargo, pare o rode o servidor para ver se esta tudo certo.

```bash
python manage.py runserver
```

#### Criando nosso modelo

Seguindo as premissas do projeto de que

1. Adicionar, Editar e Remover Tarefa
2. As tarefas serão por usuários
3. As tarefas terão status Pendente, Resolvida e Em Execução
4. As tarefas terão um agrupamento
5. As tarefas terão um prazo limite

Vamos editar o arquivo dentro da pasta `api/core/models.py` e deixa-lo lindão desse jeito

```python
from django.db import models


class Agrupamento(models.Model):
    """
    Modelo de aguepamento de tarefa
    """

    nome = models.CharField(max_length=100)

    def __str__(self):
        return self.nome


class Tarefa(models.Model):
    """
    Modelo de Tarefa
    """
    STATUS = (
        (1, 'Pendente'),
        (2, 'Execução'),
        (3, 'Resolvida'),
    )

    titulo = models.CharField(max_length=100)
    descricao = models.TextField('descrição')
    limite = models.DateTimeField()
    agrupamento = models.ForeignKey('core.Agrupamento', on_delete=models.CASCADE)
    status = models.IntegerField(choices=STATUS, default=1)
    usuario = models.ForeignKey('auth.User', on_delete=models.CASCADE)

    def __str__(self):
        return self.titulo
```

Feito isso, vamos criar nossas migrações e envia-las ao nosso banco de dados

```prompt
python manage.py makemigrations

Migrations for 'core':
  api\core\migrations\0001_initial.py
    - Create model Agrupamento
    - Create model Tarefa

python manage.py migrate

Operations to perform:
  Apply all migrations: admin, auth, contenttypes, core, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying core.0001_initial... OK
  Applying sessions.0001_initial... OK
```

#### Criando nosso admin

Massa. Agora que temos a tabela, temos a classe do modelo, vamos aproveitar o django-admin para fazer nosso crud.

Vamos editar o arquivo dentro da pasta `api/core/models.py` e deixa-lo lindão desse jeito

```python
from django.contrib import admin
from .models import Agrupamento, Tarefa

admin.site.register(Agrupamento)

@admin.register(Tarefa)
class TarefaAdmin(admin.ModelAdmin):
    """
    Admin para modelo de Tarefa
    """

    list_display = ('titulo', 'limite', 'agrupamento', 'status', 'usuario')
    search_fields = ('titulo', 'descricao')
    list_filter = ('limite', 'agrupamento')
```

Vamos criar um super usuário e levantar o servidor para testar.

```prompt
python manage.py createsuperuser
```

Levantar o servidor

```prompt
python manage.py runserver
```

Acesse o site http://localhost:8000/admin/
