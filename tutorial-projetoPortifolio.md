# Projeto: sistema web para cadastro e exibição de projetos

Este projeto tem como objetivo desenvolver uma aplicação web utilizando o framework Django, que permita o cadastro e exibição de projetos. A aplicação é construída com HTML e CSS para o front-end, e MySQL como sistema gerenciador de banco de dados. O diferencial está na criação de um sistema de cadastro personalizado, sem o uso do Django Admin, proporcionando ao aluno uma compreensão mais profunda sobre formulários, rotas, views e manipulação de dados em aplicações web modernas.

## Etapa 1: Configurações iniciais.
Crie um novo projeto 'python' usando a IDE PyCharm ou VSCode.

Abra o terminal 'python' e verifique a versão do interpretador python instalada.

Com o terminal aberto instale o framework 'Django' no projeto 'python' através do comando abaixo.

```bash
pip install django 
```

Instala a biblioteca Pillow para trabalharmos com arquivos de imagens em nosso projeto.

```bash
pip install Pillow 
```

Crie um projeto 'Django' com o nome 'portifolio' digitando o comando abaixo no terminal python.

```bash
django-admin startproject portifolio
```

O comando ``` django-admin startproject 'nome_projeto'``` cria a estrutura base do projeto Django.

Navegue até o diretório do projeto 'Django' com o comando:

```bash
cd portifolio
```
Ainda no terminal python, crie uma aplicação (módulo) do projeto 'Djando' com o comando:
```bash
python manage.py startapp projects
```
O comando ``` python manage.py startapp 'nome_app'``` cria um módulo reutilizável dentro do projeto, neste caso, chamado projects.

## Etapa 2: Configurar o projeto
#### Em settings.py
### 2.1 Adicione o app no 'settings.py'
Adicione a linha 'projects' em 'INSTALLED_APPS'. Essa configuração ativa a aplicação (módulo) criada anteriormente.

```python
INSTALLED_APPS = [
    # ...
    'projects',
]
```

### 2.2 Configurar diretórios estáticos e de media
Crie as pastas 'static' e 'media' dentro do diretório raiz do projeto 'Django', e a pasta 'css' dentro de 'static'.
#### Estruturas de pastas

```
portfolio/ 
├─ static/ 
│  └─ css/ 
│     └─ style.css 
├─ media/
```

Adicione as linhas abaixo em 'settings.py'.

```python
STATIC_URL = 'static/'
STATICFILES_DIRS = [BASE_DIR / "static"]

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

### 2.3 Configuração de URLs
#### Em portfolio/urls.py
```python
from django.conf import settings
from django.conf.urls.static import static
from django.urls import path, include

urlpatterns = [
    path('', include('projects.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## Etapa 3: Configurar a conexão com o banco de dados.

### Opção 1: SQLite (padrão)
Não é necessário alterar nada.

### Opção 2: MySQL
#### Instale o conector

```bash
pip install mysqlclient
```

#### Atualize o settings.py

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'portfolio_db',
        'USER': 'root',
        'PASSWORD': 'sua_senha',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
```

## Etapa 4: Definir modelo de dados da aplicação 'projects'
#### Em projects/models.py

```python
from django.db import models

class Project(models.Model):
    titulo = models.CharField(max_length=200)
    descricao = models.TextField()
    tecnologia = models.CharField(max_length=100)
    imagem = models.ImageField(upload_to='projetos/', null=True, blank=True)

    def __str__(self):
        return self.titulo
```

#### Crie o banco de dados no MySQL, pode usar o editor de SQL do Workbench para isso.

``` sql
CREATE DATABASE portfolio_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## Etapa 5: Criar formulário
Crie o arquivo 'forms.py' dentro do diretório 'projects' e adicione as linhas abaixo:

``` python
from django import forms
from .models import Project

class ProjectForm(forms.ModelForm):
    class Meta:
        model = Project
        fields = ['titulo', 'descricao', 'tecnologia', 'imagem']
```

## Etapa 6: Criar as views
#### Em projects/views.py

``` python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Project
from .forms import ProjectForm

def index(request):
    projetos = Project.objects.all()
    return render(request, 'projects/index.html', {'projetos': projetos, 'titulo_pagina': 'Início'})

def detalhes(request, projeto_id):
    projeto = get_object_or_404(Project, id=projeto_id)
    return render(request, 'projects/detalhes.html', {'projeto': projeto, 'titulo_pagina': projeto.titulo})

def cadastro(request):
    if request.method == 'POST':
        form = ProjectForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('index')
    else:
        form = ProjectForm()
        return render(request, 'projects/cadastro.html', {'form': form, 'titulo_pagina': 'Novo Projeto'})
```

Controla o fluxo de dados entre o modelo, o formulário e os templates.

## Etapa 7: Criar as rotas do app
Crie o arquivo 'urls.py' dentro do diretório 'projects' e adicione as linhas abaixo:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('projeto/<int:projeto_id>/', views.detalhes, name='detalhes'),
    path('cadastro/', views.cadastro, name='cadastro'),
]
```

## Etapa 8: Criar os templates

Estrutura:

```
projects/
├─ templates/
│  └─ projects/
│     ├─ base.html
│     ├─ index.html
│     ├─ detalhes.html
│     └─ cadastro.html
```
#### base.html

```html
<!DOCTYPE html>
!DOCTYPE html> 
<html lang="pt-br"> 
{% load static %} 
<head> 
    <meta charset="UTF-8"> 
    <title>{{ titulo_pagina }}</title> 
    <link rel="stylesheet" href="{% static 'css/style.css' %}"> 
</head> 
<body> 
    <header> 
        <h1>Portfólio</h1> 
        <nav> 
            <a href="{% url 'index' %}">Início</a> | 
            <a href="{% url 'cadastro' %}">Novo Projeto</a> 
        </nav> 
    </header> 
    <main> 
        {% block content %}{% endblock %} 
    </main> 
</body> 
</html> 
```
#### index.html
``` html
{% extends 'projects/base.html' %} 
 
{% block content %} 
<h2>Projetos</h2> 
<ul> 
  {% for projeto in projetos %} 
    <li> 
        <a href="{% url 'detalhes' projeto.id %}">{{ projeto.titulo }}</a> 
    </li> 
  {% empty %} 
    <li>Nenhum projeto cadastrado.</li> 
  {% endfor %} 
</ul> 
{% endblock %} 
```

#### detalhes.html
```html
{% extends 'projects/base.html' %} 
 
{% block content %} 
<h2>{{ projeto.titulo }}</h2> 
<p>{{ projeto.descricao }}</p> 
<p><strong>Tecnologia:</strong> {{ projeto.tecnologia }}</p> 
{% if projeto.imagem %} 
  <img src="{{ projeto.imagem.url }}" alt="{{ projeto.titulo }}" style="max-width:300px;"> 
{% endif %} 
<a href="{% url 'index' %}">Voltar</a> 
{% endblock %} 
```

#### cadastro.html
```html
{% extends 'projects/base.html' %} 
 
{% block content %} 
<h2>Cadastrar Novo Projeto</h2> 
<form method="post" enctype="multipart/form-data"> 
    {% csrf_token %} 
    {{ form.as_p }} 
    <button type="submit">Salvar</button> 
</form> 
{% endblock %} 
```

##  Criar o CSS
#### Em static/css/style.css: 
```css
body { 
    font-family: sans-serif; 
    margin: 20px; 
    background-color: #f2f2f2; 
} 
header { 
    background: #444; 
    color: white; 
    padding: 15px; 
} 
nav a { 
    color: white; 
    margin-right: 15px; 
    text-decoration: none; 
} 
main { 
    background: white; 
    padding: 20px; 
    border-radius: 5px; 
} 
form { 
    display: flex; 
    flex-direction: column; 
} 
form input, form textarea, form select { 
    margin-bottom: 10px; 
    padding: 8px; 
} 
```
## Etapa 9: Migrar o banco de dados
```bash
python manage.py makemigrations
```

```bash
python manage.py migrate
```
Essa etapa, cria as tabelas no servidor de banco de dados, tabelas essas que foram definidas em projects/models.py.
Obs: Sempre que fizer qualquer alteração em models.py, executar os comandos acima.

## Etapa 10: Iniciar o servidor e testar

```bash
python manage.py runserver 
```
Acesse para testar a aplicação: http://localhost:8000
