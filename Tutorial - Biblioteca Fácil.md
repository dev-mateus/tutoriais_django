# Projeto: Sistema web de reserva de livros

Este projeto tem como objetivo desenvolver uma aplicação web utilizando o *framework Django*, que permita o cadastros de livros e usuarios, e o controle de reserva de livros. A aplicação é construída com HTML e CSS para o front-end, e MySQL como sistema gerenciador de banco de dados. O diferencial está na criação de um sistema de cadastro personalizado, sem o uso do Django Admin, proporcionando ao aluno uma compreensão mais profunda sobre formulários, rotas, views e manipulação de dados em aplicações web modernas.

## Passo 1: Configurações iniciais.
Crie um novo projeto **Python** com o nome **'rp_bibliotecafacil'** usando a IDE PyCharm ou crie um diretório no seu local preferido do seu computador com o mesmo nome indicado acima e abra esse diretótio com o VSCode.

Abra o terminal da IDE e instale as bibliotecas necessárias através dos comandos:

```bash
pip install django 
```

```bash
pip install Pillow 
```

Crie um projeto *'Django'* com o nome *'bibliotecafacil'* digitando o comando abaixo no terminal python.

```bash
django-admin startproject bibliotecafacil
```

O comando ``` django-admin startproject 'nome_projeto'``` cria a estrutura base do projeto.

Navegue até o diretório do projeto com o comando:

```bash
cd bibliotecafacil
```

Ainda no terminal python, crie uma aplicação (módulo) do projeto com o comando:

```bash
python manage.py startapp reserva
```

## Passo 2: Configurar o projeto

### 2.1 Adicionando o app ao projeto
Abra o arquivo *settings.py* e adicione a linha *'reserva'* em *'INSTALLED_APPS'*. Essa configuração ativa a aplicação (módulo) criada anteriormente.

```python
INSTALLED_APPS = [
    # ...
    'reserva',
]
```

### 2.2 Configurar diretórios estáticos e de media
Crie os diretórios 'static' e 'media' dentro do diretório raiz do projeto, e o diretório 'css' dentro de 'static'.
Crie o arquivo 'style.css' dentro do diretório 'css'.
Após essas modificações a estrutura das pastas ficará assim:

```
bibliotecafacil/ 
├─ static/ 
│  └─ css/ 
│     └─ style.css 
├─ media/
```

Adicione as linhas abaixo em *'settings.py'*.

```python
STATIC_URL = 'static/' # essa linha já existe
STATICFILES_DIRS = [BASE_DIR / "static"] # <- adicione essa

MEDIA_URL = '/media/' # <- adicione essa
MEDIA_ROOT = BASE_DIR / 'media' # <- adicione essa
```

### 2.3 Configuração de URLs
Abra o arquivo *'bibliotecafacil/urls.py'* e apague todo o conteúdo e adicione o código abaixo.

```python
from django.conf import settings
from django.conf.urls.static import static
from django.urls import path, include

urlpatterns = [
    path('', include('reserva.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## Passo 3: Configurar a conexão com o banco de dados.

**Opção 1: SQLite (padrão)**
Não é necessário alterar nada, pois o SGBD default do Django é o *SQLite3*.

**Opção 2: MySQL**
Abra o terminal e instale a biblioteca *mysqlclient* que é responsável pela conexão com o *SGBD MySQL*. 

```bash
pip install mysqlclient
```

Atualize o arquivo de configuração *'settings.py'* adcionando as linhas abaixo na entrada *'DATABASES'*:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', # <- SGDB
        'NAME': 'bibliotecafacil_db', # <- nome do banco de dados
        'USER': 'root', # <- usuário do banco de dados
        'PASSWORD': 'sua_senha',# <- senha do usuário do banco de dados
        'HOST': 'localhost',# <- local do servidor de banco de dados
        'PORT': '3306',# <- porta TCP da con
    }
}
```

## Passo 4: Definindo o modelo de dados

Esse passo estabelece como os dados serão estruturados e armazenados no banco de dados. No Django, isso é feito criando modelos (*models*), que são classes *Python* que representam as tabelas e seus atribitos no banco de dados.

### Passo 4.1: Criando o model 'Livro'
Abra o arquivo *'reserva/models.py'*. Crie a classe Livro, que representa a tabela livros e seus atributos no banco de dados.

```python
from django.db import models

class Livro(models.Model):
    titulo = models.CharField(max_length=200)
    autores = models.CharField(max_length=100)
    resumo = models.TextField()
    ano = models.IntegerField()
    categoria = models.CharField(max_length=20)
    isbn = models.IntegerField()
    imagem = models.ImageField(upload_to='livros/', null=True, blank=True)

    def __str__(self):
        return self.titulo
```

### 4.2 Criando o banco de dados no MySQL
Abra o editor de script SQL do Workbench ou o de sua preferência, e digite o comando abaixo para criar o banco de dados no *'MySQL Server'*

``` sql
CREATE DATABASE bibliotecafacil_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
**Por que essa etapa é importante?**

* Define a estrutura do banco de dados:
    * Cada atributo da classe Livro vira uma coluna na tabela.
    * O Django usa essa estrutura para criar as tabelas automaticamente com os comandos makemigrations e migrate.
* Facilita a manipulação de dados:
    * Com os modelos, você pode usar comandos Python para criar, ler, atualizar e deletar registros no banco (CRUD), sem escrever SQL diretamente.
* Integração com formulários e views:
    * O modelo é usado para gerar formulários automaticamente (ModelForm) e para exibir dados nas views e templates.
* Organização e reutilização:
    * Separar a lógica de dados em modelos torna o código mais limpo, organizado e fácil de manter.

## Passo 5: Criar formulário

Esse passo consiste em criar um formulário personalizado para que o usuário possa cadastrar livros no sistema. No Django, isso é feito com a classe ModelForm, que gera automaticamente um formulário com base no modelo de dados definido (no caso, o model 'Livro' do Passo 4).

Crie o arquivo *'forms.py'* dentro do diretório *'reserva'* e adicione as linhas abaixo:

``` python
from django import forms
from .models import Livro

class LivroForm(forms.ModelForm):
    class Meta:
        model = Livro
        fields = ['titulo', 'autores', 'resumo', 'ano', 'categoria', 'isbn', 'imagem']

```
**Por que essa etapa é importante?**
* Facilita a criação de formulários:
    * O ModelForm gera automaticamente os campos do formulário com base no modelo Livro, economizando tempo e evitando erros.
* Garante validação automática:
    * O Django valida os dados inseridos com base nos tipos definidos no modelo (por exemplo, IntegerField, CharField, etc.).
* Integração com views e templates:
    * O formulário pode ser facilmente renderizado em um template HTML e processado em uma view, como será feito na Passo 6.
* Segurança:
    * O uso de ModelForm ajuda a evitar problemas como injeção de SQL ou manipulação indevida de dados, pois o Django trata a entrada do usuário de forma segura.

## Passo 6: Criar as views

As views são funções que definem o que o sistema deve fazer quando o usuário acessa uma determinada URL (*Faz uma requisição*). As views são responsáveis por:
* Processar requisições (GET, POST, etc.)
* Buscar ou salvar dados no banco de dados
* Exibir páginas HTML

No arquivo *'reserva/views.py'* adicione as linhas abaixo:

``` python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Livro
from .forms import LivroForm

def index(request):
    livros = Livro.objects.all()
    return render(request, 'index.html', {'livros': livros, 'titulo_pagina': 'Início'})

def detalhes(request, livro_id):
    livro = get_object_or_404(Livro, id=livro_id)
    return render(request, 'detalhes.html', {'livro': livro, 'titulo_pagina': livro.titulo})

def cadastro(request):
    if request.method == 'POST':
        form = LivroForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('index')
    else:
        form = LivroForm()
        return render(request, 'cadastro.html', {'form': form, 'titulo_pagina': 'Novo Livro'})
```

**Explicação:**

Cada view é associada a uma URL específica no arquivo *'urls.py'*. Quando o usuário acessa essa URL no navegador, o Django executa a função correspondente. Como mostrado abaixo:
* Quando o usuário acessa a página inicial do sistema (ex: http://localhost:8000/) a view *'index'* é chamada.
* Quando o usuário clica em um livro da lista da pagina inicial  e acessa a página de detalhes desse livro (ex: http://localhost:8000/livro/3/) a view *'detalhes'* é chamada.
* Quando o usuário acessa a página de cadastro de novo livro (ex: http://localhost:8000/cadastro/) a view 'cadastro' é chamada

**View index:**
* Busca todos os livros cadastrados. 
* Em *Livro.objects.all()*, busca todos os registros da tabela *Livro* no banco de dados. Semelhante ao comando "SELECT" do SQL. Mas usa o Model para isso.
* Envia os dados para o template index.html.

**View detalhes:**
* Busca um livro específico pelo id.
    * *livro_id*: é o identificador do livro passado pela URL.
    * *get_object_or_404(...)*: busca o livro pelo ID no banco de dados, se não encontrar, retorna erro 404. 
* Envia os dados para o template detalhes.html.
    * *render(...)*: envia os dados para o template *detalhes.html*, que mostra todas as informações do livro.

**View cadastro:**
```python
    request.method == 'POST':
``` 
Verifica se o formulário foi enviado. O método "POST" é usado quando a página web envia dados, por exemplo quando o usuário clica no botão "Salvar". Se o método for POST, significa que o formulário está sendo enviado com dados preenchidos.

```python
    form = LivroForm(request.POST, request.FILES)
```
* Cria um objeto do formulário *LivroForm*, preenchido com os dados enviados pelo usuário.
    * *request.POST*: contém os dados de texto do formulário (título, autores, etc.).
    * request.FILES: contém os arquivos enviados (como a imagem do livro).
 

```python
    if form.is_valid():
            form.save()
            return redirect('index')if form.is_valid():
```
* Verifica se os dados enviados são válidos de acordo com as regras do model Livro.  
* Se os dados forem válidos, o Django salva automaticamente o novo livro no banco de dados.
* Após salvar, o usuário é redirecionado para a página inicial *'index'*. Isso evita que o formulário seja reenviado se o usuário atualizar a página.

```python
    else:
        form = LivroForm()
        return render(request, 'cadastro.html', {'form': form, 'titulo_pagina': 'Novo Livro'})
```
Se a requisição não for POST (ou seja, for GET), significa que o usuário está apenas acessando a página de cadastro. Então a view:
* Cria um objeto *'vazio'* do formulário *LivroForm*.
* Renderiza o template *'cadastro.html'*, passando o formulário vazio e o título da página como *'Novo Livro'*. O template usará *{{ form.as_p }}* para exibir os campos do formulário automaticamente.

**Por que essa etapa é importante?**
* Controla o fluxo da aplicação:
    * As **views são o “cérebro”** da aplicação. Elas decidem o que fazer com cada requisição.
* Integra modelo, formulário e template:
    * Cada view conecta os dados (models), a entrada do usuário (forms) e a interface (templates).
* Permite lógica personalizada:
    * Você pode adicionar validações, filtros, mensagens, redirecionamentos, etc.
* Organização e clareza:
    * Separar a lógica de cada funcionalidade em views específicas torna o código mais limpo e fácil de manter.

## Passo 7: Criar as rotas do app

As rotas *(ou URLs)* definem quais caminhos o usuário pode acessar no navegador e qual função *(view)* será executada quando ele acessar cada um desses caminhos.
No Django, isso é feito no arquivo *'urls.py'*.

Crie o arquivo *'urls.py'* dentro do diretório *'reserva'* e adicione as linhas abaixo:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('livro/<int:livro_id>/', views.detalhes, name='detalhes'),
    path('cadastro/', views.cadastro, name='cadastro'),
]
```

*path('', views.index, name='index')*

* URL: '' (raiz do site)
* View chamada: 'index'
* name='index': permite usar {% url 'index' %} nos templates para gerar o link para a página index.

*path('livro/<<int:livro_id>>/', views.detalhes, name='detalhes')*
* URL: /livro/3/, /livro/10/, etc.
* View chamada: detalhes
* Parâmetro: *livro_id* (inteiro) é passado para a view.
* name='detalhes': usado nos templates para criar links dinâmicos para cada livro na pagina de detalhes.

*path('cadastro/', views.cadastro, name='cadastro')*
* URL: /cadastro/
* View chamada: cadastro
* name='cadastro': usado nos templates para criar o link para a página de cadastro.

**Explicação sobre o parâmetro _'name'_**

Quando você define uma rota no Django e informa o *'name'*, você está nomeando ou apelidando essa rota. É um apelido simbólico que pode ser usado em outras partes do projeto, especialmente nos templates HTML e em redirecionamentos.

**Por que usar _name='nome_da_rota'_?**

Evita hardcoding de URLs. Em vez de escrever diretamente a URL no HTML, como:

```HTML
<a href="/cadastro/">Cadastrar Livro</a>
```

Você pode usar:
```HTML
<a href="{% url 'cadastro' %}">Cadastrar Livro</a>
```

Isso é muito mais seguro e flexível, porque:
* Se a URL mudar no 'urls.py', você não precisa alterar o HTML. O Django vai gerar automaticamente a URL correta com base no nome da rota.

Exemplo prático:
Em *'urls.py'*:

```python
path('', views.index, name='index')
```
Em *'base.html'*:

```HTML
<a href="{% url 'index' %}">Início</a>
```
Em *'views.py'*:

```python
    return redirect('index')
```

Todos esses trechos estão conectados pelo nome da rota, e não pela URL em si. Isso torna o código mais limpo, reutilizável e fácil de manter.

## Passo 8: Criar os templates

**O que são templates no Django?**
Templates são arquivos HTML que definem a interface visual da aplicação. Eles são responsáveis por exibir os dados que vêm das views e permitir a interação do usuário com o sistema. No Django, os templates usam uma linguagem de marcação chamada Django Template Language (DTL), que é usada nos arquivos HTML do Django para inserir lógica de apresentação. Ela permite:
* Exibir variáveis do Python (vindas das views)
* Criar estruturas de controle (como if, for, etc.)
* Carregar arquivos estáticos (CSS, JS, imagens)
* Estender templates (herança)
* Criar links dinâmicos com base nas rotas nomeadas

Resumo dos DTLs usados: 
* {% extends %} - Herança de templates
* {% block %}	- Definir blocos de conteúdo
* {% load static %} - Carregar arquivos estáticos
* {% url %} - Geração dinâmica de URLs
* {{ variavel }} - Exibir dados de uma variável vindos da view que carregou o template
* {% for %} - Laço de repetição
* {% empty %} - Conteúdo alternativo para listas vazias
* {% if %} - Condicional
* {% csrf_token %} - Segurança em formulários
* {{ form.as_p }}	Renderização automática de formulários

### Passo 8.1: Criando as páginas HTML
Criar o diretério *'templates* dentro de *'reservas'* e os arquivos: *base.html, index.html, detalhes.htmle  cadastro.html*. 
```
reserva/
├─ templates/
│  ├─ base.html
│  ├─ index.html
│  ├─ detalhes.html
│  └─ cadastro.html
```
**base.html**

Esse é o template principal, que serve como base para os outros. Ele define a estrutura comum da página (como cabeçalho, navegação e layout geral).

Importante:
* {% load static %}: carrega os arquivos estáticos (como CSS).
* {% block content %}{% endblock %}: define um bloco que será preenchido pelos templates filhos.
* Links de navegação usam {% url 'nome_da_rota' %} para gerar as URLs.

```html
<!DOCTYPE html>
<html lang="pt-br"> 
{% load static %} 
<head> 
    <meta charset="UTF-8"> 
    <title>{{ titulo_pagina }}</title> 
    <link rel="stylesheet" href="{% static 'css/style.css' %}"> 
</head> 
<body> 
    <header> 
        <h1>Biblioteca Fácil</h1> 
        <nav> 
            <a href="{% url 'index' %}">Início</a> | 
            <a href="{% url 'cadastro' %}">Cadastro Livro</a> 
        </nav> 
    </header> 
    <main> 
        {% block content %}{% endblock %} 
    </main> 
</body> 
</html> 
```
**index.html**

Esse template herda de *base.html* e mostra a lista de livros cadastrados.
Importante:
* {% for livro in livros %}: percorre a lista de livros enviada pela view.
* {% url 'detalhes' livro.id %}: cria um link para a página de detalhes de cada livro.
* {% empty %}: mostra uma mensagem se não houver livros cadastrados.

``` html
{% extends 'base.html' %}

{% block content %}
<h2>Acervo</h2>
<ul>
  {% for livro in livros %}
    <li>
        <a href="{% url 'detalhes' livro.id %}">{{ livro.titulo }}</a>
    </li>
  {% empty %}
    <li>Nenhum livro cadastrado.</li>
  {% endfor %}
</ul>
{% endblock %}
```

**detalhes.html**

Mostra as informações completas de um livro específico.
Importante:
* Usa {{ livro.atributos_do_livro }} para exibir os dados do livro.
* Verifica se há imagem com {% if livro.imagem %} e exibe a imagem com {{ livro.imagem.url }}.
* Usa {% extends 'base.html' %} para herdar o layout principal.

```html
{% extends 'base.html' %}

{% block content %}
<h2>{{ livro.titulo }}</h2>
<p>Autor(es): {{ livro.autores }}</p>
<p>Resumo: {{ livro.resumo }}</p>
<p>Ano: {{ livro.ano }}</p>
<p>Categoria: {{ livro.categoria }}</p>
<p>ISBN: {{ livro.isbn }}</p>
{% if livro.imagem %}
  <img src="{{ livro.imagem.url }}" alt="{{ livro.titulo }}" style="max-width:300px;">
{% endif %}
<a href="{% url 'index' %}">Voltar</a>
{% endblock %} 
```

**cadastro.html**

Exibe o formulário para cadastrar um novo livro.
Importante:
* Usa {% csrf_token %} para proteger contra ataques CSRF.
* {{ form.as_p }} renderiza o formulário com os campos definidos em *LivroForm*.
* O formulário usa *method='POST'* para enviar os dados do formulário preenchido e *enctype='multipart/form-data'* para permitir envio de arquivos (como imagens).

```html
{% extends 'base.html' %}

{% block content %}
<h2>Cadastrar Novo Livro</h2>
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Salvar</button>
</form>
{% endblock %} 
```

### Passo 8.2: Estilizando as páginas com CSS
Após criar os arquivos HTML, vamos estilizar com o CSS. Em *static/css/style.css* adicione o código abaixo: 
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

#### Por que essa etapa é importante?
* Separa a lógica da apresentação: o Django segue o padrão MVC (Model-View-Controller), e os templates representam a "View" (interface).
* Facilita a manutenção: com base.html, você evita repetir código em todas as páginas.
* Organização: cada template tem uma função clara.
* Integra com as views: os dados enviados pelas views são exibidos nos templates.

## Passo 9: Migrar o banco de dados

Esse passo é fundamental para que o Django crie fisicamente as tabelas no banco de dados, com base nos modelos definidos em 'models.py'

Migrar é o processo de:
* Detectar alterações nos modelos (como criação de novas classes ou campos).
* Gerar arquivos de migração que descrevem essas alterações.
* Aplicar essas alterações no banco de dados, criando ou modificando tabelas

```bash
python manage.py makemigrations
```

Esse comando:
* Analisa os modelos (models.py) do seu projeto.
* Cria um arquivo de migração dentro do diretório migrations/ do app (reserva).
* Esse arquivo descreve, em Python, as instruções para criar ou alterar tabelas no banco.

*saída esperada:*

```bash
Migrations for 'reserva':
  reserva/migrations/0001_initial.py
    - Create model Livro
```    

```bash
python manage.py migrate
```

Esse comando:
* Executa os arquivos de migração criados.
* Cria as tabelas no banco de dados (MySQL ou SQLite, dependendo da sua configuração).
* Também aplica as migrações padrão do Django (como autenticação, sessões, etc.).

*saída esperada:*

```bash
Applying reserva.0001_initial... OK
```
#### Quando repetir essa passo?
Sempre que você:
* Criar um novo modelo.
* Adicionar, remover ou alterar campos em um modelo existente
* Alterar relacionamentos entre modelos.

## Passo 10: Iniciar o servidor e testar

Neste passo o servidor de desenvolvimento do Django é executado com a finalidade de testar a aplicação localmente no navegador.

```bash
python manage.py runserver 
```
Esse comando:
* Inicia o servidor web embutido do Django.
* Permite acessar a aplicação no navegador, geralmente pelo endereço: http://localhost:8000
