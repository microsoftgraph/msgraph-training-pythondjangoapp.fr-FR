<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1c530-101">Dans cet exercice, vous allez utiliser [Django](https://www.djangoproject.com/) pour créer une application Web.</span><span class="sxs-lookup"><span data-stu-id="1c530-101">In this exercise you will use [Django](https://www.djangoproject.com/) to build a web app.</span></span> <span data-ttu-id="1c530-102">Si Django n’est pas encore installé, vous pouvez l’installer à partir de votre interface de ligne de commande (CLI) à l’aide de la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="1c530-102">If you don't already have Django installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

```Shell
pip install Django=2.2.5
```

<span data-ttu-id="1c530-103">Ouvrez votre interface CLI, accédez à un répertoire où vous disposez des droits nécessaires pour créer des fichiers, puis exécutez la commande suivante pour créer une nouvelle application Django.</span><span class="sxs-lookup"><span data-stu-id="1c530-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Django app.</span></span>

```Shell
django-admin startproject graph_tutorial
```

<span data-ttu-id="1c530-104">Django crée un nouveau répertoire appelé `graph_tutorial` et génère un échafaudage d’une application Web Django.</span><span class="sxs-lookup"><span data-stu-id="1c530-104">Django creates a new directory called `graph_tutorial` and scaffolds a Django web app.</span></span> <span data-ttu-id="1c530-105">Accédez à ce nouveau répertoire et entrez la commande suivante pour démarrer un serveur Web local.</span><span class="sxs-lookup"><span data-stu-id="1c530-105">Navigate to this new directory and enter the following command to start a local web server.</span></span>

```Shell
python manage.py runserver
```

<span data-ttu-id="1c530-106">Ouvrez votre navigateur et accédez à `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="1c530-106">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="1c530-107">Si tout fonctionne, vous verrez une page d’accueil de Django.</span><span class="sxs-lookup"><span data-stu-id="1c530-107">If everything is working, you will see a Django welcome page.</span></span> <span data-ttu-id="1c530-108">Si ce n’est pas le cas, consultez le [Guide de prise](https://www.djangoproject.com/start/)en main de Django.</span><span class="sxs-lookup"><span data-stu-id="1c530-108">If you don't see that, check the [Django getting started guide](https://www.djangoproject.com/start/).</span></span>

<span data-ttu-id="1c530-109">À présent que vous avez vérifié le projet, ajoutez une application au projet.</span><span class="sxs-lookup"><span data-stu-id="1c530-109">Now that you've verified the project, add an app to the project.</span></span> <span data-ttu-id="1c530-110">Exécutez la commande suivante dans votre interface CLI.</span><span class="sxs-lookup"><span data-stu-id="1c530-110">Run the following command in your CLI.</span></span>

```Shell
python manage.py startapp tutorial
```

<span data-ttu-id="1c530-111">Cela crée une nouvelle application dans le `./tutorial` répertoire.</span><span class="sxs-lookup"><span data-stu-id="1c530-111">This creates a new app in the `./tutorial` directory.</span></span> <span data-ttu-id="1c530-112">Ouvrez le `./graph_tutorial/settings.py` et ajoutez la nouvelle `tutorial` application au `INSTALLED_APPS` paramètre.</span><span class="sxs-lookup"><span data-stu-id="1c530-112">Open the `./graph_tutorial/settings.py` and add the new `tutorial` app to the `INSTALLED_APPS` setting.</span></span>

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

<span data-ttu-id="1c530-113">Dans votre interface CLI, exécutez la commande suivante pour initialiser la base de données pour le projet.</span><span class="sxs-lookup"><span data-stu-id="1c530-113">In your CLI, run the following command to initialize the database for the project.</span></span>

```Shell
python manage.py migrate
```

<span data-ttu-id="1c530-114">Ajoutez un [URLconf](https://docs.djangoproject.com/en/2.1/topics/http/urls/) pour l' `tutorial` application.</span><span class="sxs-lookup"><span data-stu-id="1c530-114">Add a [URLconf](https://docs.djangoproject.com/en/2.1/topics/http/urls/) for the `tutorial` app.</span></span> <span data-ttu-id="1c530-115">Créez un fichier dans le `./tutorial` répertoire nommé `urls.py` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="1c530-115">Create a new file in the `./tutorial` directory named `urls.py` and add the following code.</span></span>

```python
from django.urls import path

from . import views

urlpatterns = [
  # /tutorial
  path('', views.home, name='home'),
]
```

<span data-ttu-id="1c530-116">À présent, mettez à jour le projet URLconf pour l’importer.</span><span class="sxs-lookup"><span data-stu-id="1c530-116">Now update the project URLconf to import this one.</span></span> <span data-ttu-id="1c530-117">Ouvrez le `./graph_tutorial/urls.py` fichier et remplacez le contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="1c530-117">Open the `./graph_tutorial/urls.py` file and replace the contents with the following.</span></span>

```python
from django.contrib import admin
from django.urls import path, include
from tutorial import views

urlpatterns = [
    path('tutorial/', include('tutorial.urls')),
    path('admin/', admin.site.urls),
]
```

<span data-ttu-id="1c530-118">Enfin, ajoutez une vue temporaire à `tutorials` l’application pour vérifier que le routage des URL fonctionne.</span><span class="sxs-lookup"><span data-stu-id="1c530-118">Finally add a temporary view to the `tutorials` app to verify that URL routing is working.</span></span> <span data-ttu-id="1c530-119">Ouvrez le fichier `./tutorial/views.py` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="1c530-119">Open the `./tutorial/views.py` file and add the following code.</span></span>

```python
from django.http import HttpResponse, HttpResponseRedirect

def home(request):
  # Temporary!
  return HttpResponse("Welcome to the tutorial.")
```

<span data-ttu-id="1c530-120">Enregistrez toutes vos modifications et redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="1c530-120">Save all of your changes and restart the server.</span></span> <span data-ttu-id="1c530-121">Accédez à `http://localhost:8000/tutorial`.</span><span class="sxs-lookup"><span data-stu-id="1c530-121">Browse to `http://localhost:8000/tutorial`.</span></span> <span data-ttu-id="1c530-122">Vous devriez voir`Welcome to the tutorial.`</span><span class="sxs-lookup"><span data-stu-id="1c530-122">You should see `Welcome to the tutorial.`</span></span>

<span data-ttu-id="1c530-123">Avant de poursuivre, installez des bibliothèques supplémentaires que vous utiliserez plus tard :</span><span class="sxs-lookup"><span data-stu-id="1c530-123">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="1c530-124">[Demandes-OAuthlib : OAuth pour les êtres humains](https://requests-oauthlib.readthedocs.io/en/latest/) pour la gestion des flux de connexion et de jetons OAuth, et pour passer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1c530-124">[Requests-OAuthlib: OAuth for Humans](https://requests-oauthlib.readthedocs.io/en/latest/) for handling sign-in and OAuth token flows, and for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="1c530-125">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) pour le chargement de la configuration à partir d’un fichier YAML.</span><span class="sxs-lookup"><span data-stu-id="1c530-125">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) for loading configuration from a YAML file.</span></span>
- <span data-ttu-id="1c530-126">[python-dateutil](https://pypi.org/project/python-dateutil/) pour l’analyse des chaînes de date ISO 8601 renvoyées par Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1c530-126">[python-dateutil](https://pypi.org/project/python-dateutil/) for parsing ISO 8601 date strings returned from Microsoft Graph.</span></span>

<span data-ttu-id="1c530-127">Exécutez la commande suivante dans votre interface CLI.</span><span class="sxs-lookup"><span data-stu-id="1c530-127">Run the following command in your CLI.</span></span>

```Shell
pip install requests_oauthlib==1.2.0
pip install pyyaml==5.1
pip install python-dateutil==2.8.0
```

## <a name="design-the-app"></a><span data-ttu-id="1c530-128">Concevoir l’application</span><span class="sxs-lookup"><span data-stu-id="1c530-128">Design the app</span></span>

<span data-ttu-id="1c530-129">Commencez par créer un répertoire de modèles et définissez une disposition globale pour l’application.</span><span class="sxs-lookup"><span data-stu-id="1c530-129">Start by creating a templates directory and defining a global layout for the app.</span></span> <span data-ttu-id="1c530-130">Créez un répertoire dans le `./tutorial` répertoire nommé. `templates`</span><span class="sxs-lookup"><span data-stu-id="1c530-130">Create a new directory in the `./tutorial` directory named `templates`.</span></span> <span data-ttu-id="1c530-131">Dans le `templates` répertoire, créez un répertoire nommé `tutorial`.</span><span class="sxs-lookup"><span data-stu-id="1c530-131">In the `templates` directory, create a new directory named `tutorial`.</span></span> <span data-ttu-id="1c530-132">Enfin, créez un fichier dans ce répertoire nommé `layout.html`.</span><span class="sxs-lookup"><span data-stu-id="1c530-132">Finally, create a new file in this directory named `layout.html`.</span></span> <span data-ttu-id="1c530-133">Le chemin d’accès relatif à partir de la racine de `./tutorial/templates/tutorial/layout.html`votre projet doit être.</span><span class="sxs-lookup"><span data-stu-id="1c530-133">The relative path from the root of your project should be `./tutorial/templates/tutorial/layout.html`.</span></span> <span data-ttu-id="1c530-134">Ajoutez le code suivant dans ce fichier.</span><span class="sxs-lookup"><span data-stu-id="1c530-134">Add the following code in that file.</span></span>

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

<span data-ttu-id="1c530-135">Ce code ajoute [bootstrap](http://getbootstrap.com/) pour les styles simples et [font Isard](https://fontawesome.com/) pour certaines icônes simples.</span><span class="sxs-lookup"><span data-stu-id="1c530-135">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="1c530-136">Il définit également une disposition globale avec une barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="1c530-136">It also defines a global layout with a nav bar.</span></span>

<span data-ttu-id="1c530-137">Créez maintenant un nouveau répertoire dans le `./tutorial` répertoire nommé `static`.</span><span class="sxs-lookup"><span data-stu-id="1c530-137">Now create a new directory in the `./tutorial` directory named `static`.</span></span> <span data-ttu-id="1c530-138">Dans le `static` répertoire, créez un répertoire nommé `tutorial`.</span><span class="sxs-lookup"><span data-stu-id="1c530-138">In the `static` directory, create a new directory named `tutorial`.</span></span> <span data-ttu-id="1c530-139">Enfin, créez un fichier dans ce répertoire nommé `app.css`.</span><span class="sxs-lookup"><span data-stu-id="1c530-139">Finally, create a new file in this directory named `app.css`.</span></span> <span data-ttu-id="1c530-140">Le chemin d’accès relatif à partir de la racine de `./tutorial/static/tutorial/app.css`votre projet doit être.</span><span class="sxs-lookup"><span data-stu-id="1c530-140">The relative path from the root of your project should be `./tutorial/static/tutorial/app.css`.</span></span> <span data-ttu-id="1c530-141">Ajoutez le code suivant dans ce fichier.</span><span class="sxs-lookup"><span data-stu-id="1c530-141">Add the following code in that file.</span></span>

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

<span data-ttu-id="1c530-142">Ensuite, créez un modèle pour la page d’accueil qui utilise la mise en page.</span><span class="sxs-lookup"><span data-stu-id="1c530-142">Next, create a template for the home page that uses the layout.</span></span> <span data-ttu-id="1c530-143">Créez un fichier dans le `./tutorial/templates/tutorial` répertoire nommé `home.html` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="1c530-143">Create a new file in the `./tutorial/templates/tutorial` directory named `home.html` and add the following code.</span></span>

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

<span data-ttu-id="1c530-144">Mettez à `home` jour l’affichage pour utiliser ce modèle.</span><span class="sxs-lookup"><span data-stu-id="1c530-144">Update the `home` view to use this template.</span></span> <span data-ttu-id="1c530-145">Ouvrez le `./tutorial/views.py` fichier et ajoutez la nouvelle fonction suivante.</span><span class="sxs-lookup"><span data-stu-id="1c530-145">Open the `./tutorial/views.py` file and add the following new function.</span></span>

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

<span data-ttu-id="1c530-146">Remplacez ensuite la vue `home` existante par la suivante.</span><span class="sxs-lookup"><span data-stu-id="1c530-146">Then replace the existing `home` view with the following.</span></span>

```python
def home(request):
  context = initialize_context(request)

  return render(request, 'tutorial/home.html', context)
```

<span data-ttu-id="1c530-147">Enregistrez toutes vos modifications et redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="1c530-147">Save all of your changes and restart the server.</span></span> <span data-ttu-id="1c530-148">À présent, l’application doit être très différente.</span><span class="sxs-lookup"><span data-stu-id="1c530-148">Now, the app should look very different.</span></span>

![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
