<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez utiliser [Django](https://www.djangoproject.com/) pour créer une application Web.

1. Si Django n’est pas encore installé, vous pouvez l’installer à partir de votre interface de ligne de commande (CLI) à l’aide de la commande suivante.

    ```Shell
    pip install Django==3.0.4
    ```

1. Ouvrez votre interface CLI, accédez à un répertoire où vous disposez des droits nécessaires pour créer des fichiers, puis exécutez la commande suivante pour créer une nouvelle application Django.

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. Accédez au répertoire **graph_tutorial** et entrez la commande suivante pour démarrer un serveur Web local.

    ```Shell
    python manage.py runserver
    ```

1. Ouvrez votre navigateur et accédez à `http://localhost:8000`. Si tout fonctionne, vous verrez une page d’accueil de Django. Si ce n’est pas le cas, consultez le [Guide de prise](https://www.djangoproject.com/start/)en main de Django.

1. Ajoutez une application au projet. Exécutez la commande suivante dans votre interface CLI.

    ```Shell
    python manage.py startapp tutorial
    ```

1. Ouvrez **./graph_tutorial/Settings.py** et ajoutez la nouvelle `tutorial` application au `INSTALLED_APPS` paramètre.

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. Dans votre interface CLI, exécutez la commande suivante pour initialiser la base de données pour le projet.

    ```Shell
    python manage.py migrate
    ```

1. Ajoutez un [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) pour l' `tutorial` application. Créez un fichier dans le répertoire **./Tutorial** nommé `urls.py` et ajoutez le code suivant.

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

1. Mettez à jour le projet URLconf pour importer celui-ci. Ouvrez **./graph_tutorial/URLs.py** et remplacez le contenu par ce qui suit.

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. Ajoutez une vue temporaire à l' `tutorials` application pour vérifier que le routage des URL fonctionne. Ouvrez **./Tutorial/views.py** et ajoutez le code suivant.

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. Enregistrez toutes vos modifications et redémarrez le serveur. Accédez à `http://localhost:8000`. Vous devriez voir`Welcome to the tutorial.`

## <a name="install-libraries"></a>Installation des bibliothèques

Avant de poursuivre, installez des bibliothèques supplémentaires que vous utiliserez plus tard :

- [Demandes-OAuthlib : OAuth pour les êtres humains](https://requests-oauthlib.readthedocs.io/en/latest/) pour la gestion des flux de connexion et de jetons OAuth, et pour passer des appels à Microsoft Graph.
- [PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) pour le chargement de la configuration à partir d’un fichier YAML.
- [python-dateutil](https://pypi.org/project/python-dateutil/) pour l’analyse des chaînes de date ISO 8601 renvoyées par Microsoft Graph.

1. Exécutez la commande suivante dans votre interface CLI.

    ```Shell
    pip install requests_oauthlib==1.3.0
    pip install pyyaml==5.3.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a>Concevoir l’application

1. Créez un répertoire dans le répertoire **./Tutorial** nommé `templates`.

1. Dans le répertoire **./Tutorial/templates** , créez un répertoire nommé `tutorial`.

1. Dans le répertoire **./Tutorial/templates/Tutorial** , créez un fichier nommé `layout.html`. Ajoutez le code suivant dans ce fichier.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    Ce code ajoute [Bootstrap](http://getbootstrap.com/) pour la mise en forme simple et [Font Awesome](https://fontawesome.com/) pour certaines icônes simples. Il définit également une disposition globale avec une barre de navigation.

1. Créez un répertoire dans le répertoire **./Tutorial** nommé `static`.

1. Dans le répertoire **./Tutorial/static** , créez un répertoire nommé `tutorial`.

1. Dans le répertoire **./Tutorial/static/Tutorial** , créez un fichier nommé `app.css`. Ajoutez le code suivant dans ce fichier.

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. Créez un modèle pour la page d’accueil qui utilise la mise en page. Créez un fichier dans le répertoire **./Tutorial/templates/Tutorial** nommé `home.html` et ajoutez le code suivant.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. Ouvrez le `./tutorial/views.py` fichier et ajoutez la nouvelle fonction suivante.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. Remplacez l’affichage `home` existant par ce qui suit.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. Enregistrez toutes vos modifications et redémarrez le serveur. À présent, l’application doit être très différente.

    ![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
