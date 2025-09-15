# ✅ Passo a Passo: Projeto Django - Lista de Tarefas (ToDo List)

## Objetivo
Construir um CRUD completo de tarefas (listar, criar, editar, excluir e marcar como concluída) usando **Django** e **SQLite**.

---

## 0) Pré-requisitos
- **Python 3.10+**
- **pip** e **venv**
- Terminal / PowerShell

> Obs: Funciona em Django 4.x ou 5.x.

---

## 1) Criar o projeto e o ambiente virtual

### macOS / Linux
```bash
mkdir django-todo
cd django-todo
python3 -m venv .venv
source .venv/bin/activate
pip install django
django-admin startproject todo .
python manage.py startapp tasks
code .
```

### Windows (PowerShell)
```powershell
mkdir django-todo
cd django-todo
python -m venv .venv
.\.venv\Scripts\Activate
pip install django
django-admin startproject todo .
python manage.py startapp tasks
code .
```

---

## 2) Registrar o app no projeto
Edite `todo/settings.py` e adicione `tasks` em `INSTALLED_APPS`:
```python
INSTALLED_APPS = [
    # ...
    'tasks',
]
```

---

## 3) Modelo (Model)
Crie o modelo da tarefa em `tasks/models.py`:
```python
from django.db import models

class Task(models.Model):
    title = models.CharField('Título', max_length=200)
    description = models.TextField('Descrição', blank=True)
    completed = models.BooleanField('Concluída', default=False)
    created_at = models.DateTimeField('Criada em', auto_now_add=True)
    updated_at = models.DateTimeField('Atualizada em', auto_now=True)

    class Meta:
        ordering = ['completed', '-created_at']  # Não concluídas primeiro
        verbose_name = 'Tarefa'
        verbose_name_plural = 'Tarefas'

    def __str__(self):
        return self.title
```

---

## 4) Migrar o banco e criar superusuário
```bash
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
# siga as instruções do terminal, e na duvida sobre as instruções pesquise antes de prosseguir.
```

---

## 5) Admin (para testar rápido)
Em `tasks/admin.py`:
```python
from django.contrib import admin
from .models import Task

@admin.register(Task)
class TaskAdmin(admin.ModelAdmin):
    list_display = ('title', 'completed', 'created_at')
    list_filter = ('completed', 'created_at')
    search_fields = ('title',)
```

Inicie e acesse `/admin`:
```bash
python manage.py runserver
```

---

## 6) Formulário (ModelForm)
Crie o arquivo `tasks/forms.py`:
```python
from django import forms
from .models import Task

class TaskForm(forms.ModelForm):
    class Meta:
        model = Task
        fields = ['title', 'description']
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Título da tarefa'}),
            'description': forms.Textarea(attrs={'class': 'form-control', 'rows': 3, 'placeholder': 'Descrição (opcional)'}),
        }
```

---

## 7) Views (CRUD + marcar como concluída)
Em `tasks/views.py`:
```python
from django.shortcuts import render, redirect, get_object_or_404
from django.views.decorators.http import require_POST
from .models import Task
from .forms import TaskForm

def task_list(request):
    q = request.GET.get('q', '')
    tasks = Task.objects.all()
    if q:
        tasks = tasks.filter(title__icontains=q)
    # Form na própria listagem para criar rápido
    form = TaskForm()
    return render(request, 'tasks/task_list.html', {'tasks': tasks, 'form': form, 'q': q})

@require_POST
def create_task(request):
    form = TaskForm(request.POST)
    if form.is_valid():
        form.save()
    return redirect('task_list')

def update_task(request, pk):
    task = get_object_or_404(Task, pk=pk)
    if request.method == 'POST':
        form = TaskForm(request.POST, instance=task)
        if form.is_valid():
            form.save()
            return redirect('task_list')
    else:
        form = TaskForm(instance=task)
    return render(request, 'tasks/task_form.html', {'form': form, 'task': task})

def delete_task(request, pk):
    task = get_object_or_404(Task, pk=pk)
    if request.method == 'POST':
        task.delete()
        return redirect('task_list')
    return render(request, 'tasks/task_confirm_delete.html', {'task': task})

@require_POST
def toggle_complete(request, pk):
    task = get_object_or_404(Task, pk=pk)
    task.completed = not task.completed
    task.save()
    return redirect('task_list')
```

---

## 8) URLs
### Projeto: `todo/urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('tasks.urls')),
]
```

### App: `tasks/urls.py` (crie o arquivo)
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.task_list, name='task_list'),
    path('add/', views.create_task, name='create_task'),
    path('<int:pk>/edit/', views.update_task, name='update_task'),
    path('<int:pk>/delete/', views.delete_task, name='delete_task'),
    path('<int:pk>/toggle/', views.toggle_complete, name='toggle_complete'),
]
```

---

## 9) Templates
Crie a pasta `templates/tasks/` e os arquivos abaixo.

### `templates/tasks/base.html`
```html
<!doctype html>
<html lang="pt-br">
<head>
  <meta charset="utf-8">
  <title>{% block title %}ToDo List{% endblock %}</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <!-- Bootstrap (CDN para estudo) -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <style>
    .done { text-decoration: line-through; color: #777; }
  </style>
</head>
<body class="bg-light">
  <nav class="navbar navbar-dark bg-dark mb-4">
    <div class="container">
      <a class="navbar-brand" href="{% url 'task_list' %}">ToDo</a>
    </div>
  </nav>
  <main class="container">
    {% block content %}{% endblock %}
  </main>
</body>
</html>
```

### `templates/tasks/task_list.html`
```html
{% extends 'tasks/base.html' %}
{% block title %}Minhas Tarefas{% endblock %}
{% block content %}
<div class="row">
  <div class="col-lg-7">
    <div class="d-flex justify-content-between align-items-center mb-3">
      <h3 class="mb-0">Tarefas</h3>
      <form class="d-flex" method="get">
        <input class="form-control me-2" type="search" name="q" value="{{ q }}" placeholder="Buscar...">
        <button class="btn btn-outline-secondary" type="submit">Buscar</button>
      </form>
    </div>

    <ul class="list-group">
      {% for task in tasks %}
        <li class="list-group-item d-flex justify-content-between align-items-center">
          <div>
            <span class="{% if task.completed %}done{% endif %}"><strong>{{ task.title }}</strong></span>
            {% if task.description %}
              <div class="text-muted small">{{ task.description }}</div>
            {% endif %}
            <div class="small text-secondary">Criada: {{ task.created_at|date:"d/m/Y H:i" }}</div>
          </div>
          <div class="d-flex gap-2">
            <form method="post" action="{% url 'toggle_complete' task.pk %}">
              {% csrf_token %}
              <button class="btn btn-sm {% if task.completed %}btn-warning{% else %}btn-success{% endif %}">
                {% if task.completed %}Desmarcar{% else %}Concluir{% endif %}
              </button>
            </form>
            <a class="btn btn-sm btn-primary" href="{% url 'update_task' task.pk %}">Editar</a>
            <a class="btn btn-sm btn-outline-danger" href="{% url 'delete_task' task.pk %}">Excluir</a>
          </div>
        </li>
      {% empty %}
        <li class="list-group-item">Nenhuma tarefa encontrada.</li>
      {% endfor %}
    </ul>
  </div>

  <div class="col-lg-5">
    <div class="card">
      <div class="card-body">
        <h5 class="card-title">Nova Tarefa</h5>
        <form method="post" action="{% url 'create_task' %}">
          {% csrf_token %}
          {{ form.as_p }}
          <button class="btn btn-dark">Adicionar</button>
        </form>
      </div>
    </div>
  </div>
</div>
{% endblock %}
```

### `templates/tasks/task_form.html`
```html
{% extends 'tasks/base.html' %}
{% block title %}Editar Tarefa{% endblock %}
{% block content %}
<div class="col-lg-8 mx-auto">
  <div class="card">
    <div class="card-body">
      <h5 class="card-title">Editar: {{ task.title }}</h5>
      <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <div class="d-flex gap-2">
          <button class="btn btn-primary">Salvar</button>
          <a href="{% url 'task_list' %}" class="btn btn-outline-secondary">Cancelar</a>
        </div>
      </form>
    </div>
  </div>
</div>
{% endblock %}
```

### `templates/tasks/task_confirm_delete.html`
```html
{% extends 'tasks/base.html' %}
{% block title %}Excluir Tarefa{% endblock %}
{% block content %}
<div class="col-lg-8 mx-auto">
  <div class="alert alert-danger">
    <h5>Tem certeza que deseja excluir a tarefa <strong>"{{ task.title }}"</strong>?</h5>
    <form method="post">
      {% csrf_token %}
      <div class="d-flex gap-2 mt-3">
        <button class="btn btn-danger">Sim, excluir</button>
        <a class="btn btn-outline-secondary" href="{% url 'task_list' %}">Cancelar</a>
      </div>
    </form>
  </div>
</div>
{% endblock %}
```

---

## 10) Executar
```bash
python manage.py runserver
```
Abra: `http://127.0.0.1:8000/`

---

## 11) Estrutura final (referência)
```
django-todo/
├─ .venv/
├─ manage.py
├─ todo/
│  ├─ __init__.py
│  ├─ asgi.py
│  ├─ settings.py
│  ├─ urls.py
│  └─ wsgi.py
└─ tasks/
   ├─ __init__.py
   ├─ admin.py
   ├─ apps.py
   ├─ forms.py
   ├─ migrations/
   │  └─ 0001_initial.py
   ├─ models.py
   ├─ tests.py
   ├─ urls.py
   ├─ views.py
   └─ templates/
      └─ tasks/
         ├─ base.html
         ├─ task_list.html
         ├─ task_form.html
         └─ task_confirm_delete.html
```

---

## 12) O projeto está funcionando, mas… faça este teste:
Teste:
* 1) Acesse o Admin em http://127.0.0.1:8000/admin/.

* 2) Faça login no Admin com o usuario criado no passo 4, adcione uma terafa. Faça o logout.

* 3) Acesse a aplicação em http://127.0.0.1:8000/.

* 4) Perceba que você pode adicionar tarefas mesmo sem estar logado na aplicação.

Perguntas:
* Por que isso acontece?
* Como você faria para corrigir isso?

Dicas para sua pesquisa:

* Como restringir acesso às views para usuários autenticados?
* Como redirecionar para login quando o usuário não está autenticado?
* Como associar tarefas ao usuário logado?
Palavras-chave para buscar:

* django login_required
* django redirect after login
* django associar objeto ao usuário
Correção:
* Permitir que somente usuários logados acessem a lista de tarefas.
* Se não estiver logado, redirecionar para a página de login. Depois do login, voltar para a view task_list.

## 13) Melhorias, algumas são opcionais, outras são necessárias para fazer a correção pedida acima:
- **Autenticação & Usuário**  
  - Adicionar um campo `owner = ForeignKey(User, ...)` no modelo e filtrar por `request.user`.  
  - Proteger views com `login_required`.
- **Campos extras**: `due_date` (prazo), `priority` (prioridade).
- **Paginação e busca avançada** (por concluída/não concluída).
- **Mensagens de feedback** com `django.contrib.messages`.
