<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez utiliser [Django](https://www.djangoproject.com/) pour créer une application Web. Si Django n’est pas encore installé, vous pouvez l’installer à partir de votre interface de ligne de commande (CLI) à l’aide de la commande suivante.

```Shell
pip install Django=2.2.2
```

Ouvrez votre interface CLI, accédez à un répertoire où vous disposez des droits nécessaires pour créer des fichiers, puis exécutez la commande suivante pour créer une nouvelle application Django.

```Shell
django-admin startproject graph_tutorial
```

Django crée un nouveau répertoire appelé `graph_tutorial` et génère un échafaudage d’une application Web Django. Accédez à ce nouveau répertoire et entrez la commande suivante pour démarrer un serveur Web local.

```Shell
python manage.py runserver
```

Ouvrez votre navigateur et accédez à `http://localhost:8000`. Si tout fonctionne, vous verrez une page d’accueil de Django. Si ce n’est pas le cas, consultez le [Guide de prise](https://www.djangoproject.com/start/)en main de Django.

À présent que vous avez vérifié le projet, ajoutez une application au projet. Exécutez la commande suivante dans votre interface CLI.

```Shell
python manage.py startapp tutorial
```

Cela crée une nouvelle application dans le `./tutorial` répertoire. Ouvrez le `./graph_tutorial/settings.py` et ajoutez la nouvelle `tutorial` application au `INSTALLED_APPS` paramètre.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'tutorial',
]
```

Dans votre interface CLI, exécutez la commande suivante pour initialiser la base de données pour le projet.

```Shell
python manage.py migrate
```

Ajoutez un [URLconf](https://docs.djangoproject.com/en/2.1/topics/http/urls/) pour l' `tutorial` application. Créez un fichier dans le `./tutorial` répertoire nommé `urls.py` et ajoutez le code suivant.

```python
from django.urls import path

from . import views

urlpatterns = [
  # /tutorial
  path('', views.home, name='home'),
]
```

À présent, mettez à jour le projet URLconf pour l’importer. Ouvrez le `./graph_tutorial/urls.py` fichier et remplacez le contenu par ce qui suit.

```python
from django.contrib import admin
from django.urls import path, include
from tutorial import views

urlpatterns = [
    path('tutorial/', include('tutorial.urls')),
    path('admin/', admin.site.urls),
]
```

Enfin, ajoutez une vue temporaire à `tutorials` l’application pour vérifier que le routage des URL fonctionne. Ouvrez le fichier `./tutorial/views.py` et ajoutez le code suivant.

```python
from django.http import HttpResponse, HttpResponseRedirect

def home(request):
  # Temporary!
  return HttpResponse("Welcome to the tutorial.")
```

Enregistrez toutes vos modifications et redémarrez le serveur. Accédez à `http://localhost:8000/tutorial`. Vous devriez voir`Welcome to the tutorial.`

Avant de poursuivre, installez des bibliothèques supplémentaires que vous utiliserez plus tard:

- [Demandes-OAuthlib: OAuth pour les êtres humains](https://requests-oauthlib.readthedocs.io/en/latest/) pour la gestion des flux de connexion et de jetons OAuth, et pour passer des appels à Microsoft Graph.
- [PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) pour le chargement de la configuration à partir d’un fichier YAML.
- [python-dateutil](https://pypi.org/project/python-dateutil/) pour l’analyse des chaînes de date ISO 8601 renvoyées par Microsoft Graph.

Exécutez la commande suivante dans votre interface CLI.

```Shell
pip install requests_oauthlib==1.2.0
pip install pyyaml==5.1
pip install python-dateutil==2.8.0
```

## <a name="design-the-app"></a>Concevoir l’application

Commencez par créer un répertoire de modèles et définissez une disposition globale pour l’application. Créez un répertoire dans le `./tutorial` répertoire nommé. `templates` Dans le `templates` répertoire, créez un répertoire nommé `tutorial`. Enfin, créez un fichier dans ce répertoire nommé `layout.html`. Le chemin d’accès relatif à partir de la racine de `./tutorial/templates/tutorial/layout.html`votre projet doit être. Ajoutez le code suivant dans ce fichier.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Python Graph Tutorial</title>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
      integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
      integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    {% load static %}
    <link rel="stylesheet" href="{% static "tutorial/app.css" %}">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
      integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
      integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <a href="{% url 'home' %}" class="navbar-brand">Python Graph Tutorial</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
          aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <a href="{% url 'home' %}" class="nav-link{% if request.resolver_match.view_name == 'home' %} active{% endif %}">Home</a>
            </li>
            {% if user.is_authenticated %}
              <li class="nav-item" data-turbolinks="false">
                <a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="#">Calendar</a>
              </li>
            {% endif %}
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            {% if user.is_authenticated %}
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                  {% if user.avatar %}
                    <img src="{{ user.avatar }}" class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  {% else %}
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  {% endif %}
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0">{{ user.name }}</h5>
                  <p class="dropdown-item-text text-muted mb-0">{{ user.email }}</p>
                  <div class="dropdown-divider"></div>
                  <a href="#" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            {% else %}
              <li class="nav-item">
                <a href="#" class="nav-link">Sign In</a>
              </li>
            {% endif %}
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      {% if errors %}
        {% for error in errors %}
          <div class="alert alert-danger" role="alert">
            <p class="mb-3">{{ error.message }}</p>
            {% if error.debug %}
              <pre class="alert-pre border bg-light p-2"><code>{{ error.debug }}</code></pre>
            {% endif %}
          </div>
        {% endfor %}
      {% endif %}
      {% block content %}{% endblock %}
    </main>
  </body>
</html>
```

Ce code ajoute [bootstrap](http://getbootstrap.com/) pour les styles simples et [font Isard](https://fontawesome.com/) pour certaines icônes simples. Il définit également une disposition globale avec une barre de navigation.

Créez maintenant un nouveau répertoire dans le `./tutorial` répertoire nommé `static`. Dans le `static` répertoire, créez un répertoire nommé `tutorial`. Enfin, créez un fichier dans ce répertoire nommé `app.css`. Le chemin d’accès relatif à partir de la racine de `./tutorial/static/tutorial/app.css`votre projet doit être. Ajoutez le code suivant dans ce fichier.

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

Ensuite, créez un modèle pour la page d’accueil qui utilise la mise en page. Créez un fichier dans le `./tutorial/templates/tutorial` répertoire nommé `home.html` et ajoutez le code suivant.

```html
{% extends "tutorial/layout.html" %}
{% block content %}
<div class="jumbotron">
  <h1>Python Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from Python</p>
  {% if user.is_authenticated %}
    <h4>Welcome {{ user.name }}!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  {% else %}
    <a href="#" class="btn btn-primary btn-large">Click here to sign in</a>
  {% endif %}
</div>
{% endblock %}
```

Mettez à `home` jour l’affichage pour utiliser ce modèle. Ouvrez le `./tutorial/views.py` fichier et ajoutez la nouvelle fonction suivante.

```python
def initialize_context(request):
  context = {}

  # Check for any errors in the session
  error = request.session.pop('flash_error', None)

  if error != None:
    context['errors'] = []
    context['errors'].append(error)

  # Check for user in the session
  context['user'] = request.session.get('user', {'is_authenticated': False})
  return context
```

Remplacez ensuite la vue `home` existante par la suivante.

```python
def home(request):
  context = initialize_context(request)

  return render(request, 'tutorial/home.html', context)
```

Enregistrez toutes vos modifications et redémarrez le serveur. À présent, l’application doit être très différente.

![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
