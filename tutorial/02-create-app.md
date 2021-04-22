<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez utiliser [Django](https://www.djangoproject.com/) pour créer une application web.

1. Si Django n'est pas déjà installé, vous pouvez l'installer à partir de votre interface de ligne de commande (CLI) avec la commande suivante.

    ```Shell
    pip install Django==3.2
    ```

1. Ouvrez votre CLI, accédez à un répertoire dans lequel vous avez le droit de créer des fichiers et exécutez la commande suivante pour créer une application Django.

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. Accédez au **répertoire graph_tutorial** et entrez la commande suivante pour démarrer un serveur web local.

    ```Shell
    python manage.py runserver
    ```

1. Ouvrez votre navigateur et accédez à `http://localhost:8000`. Si tout fonctionne, vous verrez une page d'accueil Django. Si ce n'est pas le cas, consultez le guide de mise en [place de Django.](https://www.djangoproject.com/start/)

1. Ajoutez une application au projet. Exécutez la commande suivante dans votre CLI.

    ```Shell
    python manage.py startapp tutorial
    ```

1. Ouvrez **./graph_tutorial/settings.py** et ajoutez la nouvelle `tutorial` application au `INSTALLED_APPS` paramètre.

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. Dans votre CLI, exécutez la commande suivante pour initialiser la base de données du projet.

    ```Shell
    python manage.py migrate
    ```

1. Ajoutez [un URNfnf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) pour l'application. `tutorial` Créez un fichier dans le **répertoire ./tutorial** nommé `urls.py` et ajoutez le code suivant.

    ```python
    from django.urls import path

    from . import views

    urlpatterns = [
      # /
      path('', views.home, name='home'),
      # TEMPORARY
      path('signin', views.home, name='signin'),
      path('signout', views.home, name='signout'),
      path('calendar', views.home, name='calendar'),
    ]
    ```

1. Mettez à jour l'URNf du projet pour importer celui-ci. Ouvrez **./graph_tutorial/urls.py** et remplacez le contenu par ce qui suit.

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. Ajoutez une vue temporaire à `tutorials` l'application pour vérifier que le routage d'URL fonctionne. Ouvrez **./tutorial/views.py** et remplacez tout son contenu par le code suivant.

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect
    from django.urls import reverse
    from datetime import datetime, timedelta

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. Enregistrez toutes vos modifications et redémarrez le serveur. Accédez `http://localhost:8000` à . Vous devriez voir `Welcome to the tutorial.`

## <a name="install-libraries"></a>Installation des bibliothèques

Avant de passer à la suite, installez certaines bibliothèques supplémentaires que vous utiliserez ultérieurement :

- [Bibliothèque d'authentification Microsoft (MSAL) pour Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) pour la gestion des flux de jeton OAuth et de la signature.
- [PyYAML pour le chargement](https://pyyaml.org/wiki/PyYAMLDocumentation) de la configuration à partir d'un fichier YAML.
- [python-dateutil pour](https://pypi.org/project/python-dateutil/) l'utilisation des chaînes de date ISO 8601 renvoyées par Microsoft Graph.

1. Exécutez la commande suivante dans votre CLI.

    ```Shell
    pip install msal==1.10.0
    pip install pyyaml==5.4.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a>Concevoir l’application

1. Créez un répertoire dans le **répertoire ./tutorial** nommé `templates` .

1. Dans le **répertoire ./tutorial/templates,** créez un répertoire nommé `tutorial` .

1. Dans le **répertoire ./tutorial/templates/tutorial,** créez un fichier nommé `layout.html` . Ajoutez le code suivant dans ce fichier.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    Ce code ajoute [Bootstrap pour](http://getbootstrap.com/) un style simple et [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) pour certaines icônes simples. Il définit également une disposition globale avec une barre de navigation.

1. Créez un répertoire dans le **répertoire ./tutorial** nommé `static` .

1. Dans le **répertoire ./tutorial/static,** créez un répertoire nommé `tutorial` .

1. Dans le **répertoire ./tutorial/static/tutorial,** créez un fichier nommé `app.css` . Ajoutez le code suivant dans ce fichier.

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. Créez un modèle pour la page d'accueil qui utilise la mise en page. Créez un fichier dans le répertoire **./tutorial/templates/tutorial** nommé `home.html` et ajoutez le code suivant.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. Ouvrez `./tutorial/views.py` le fichier et ajoutez la nouvelle fonction suivante.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. Remplacez l'affichage `home` existant par ce qui suit.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. Ajoutez un fichier PNG nommé **no-profile-photo.png** dans le répertoire **./tutorial/static/tutorial.**

1. Enregistrez toutes vos modifications et redémarrez le serveur. L'application doit maintenant avoir une apparence très différente.

    ![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
