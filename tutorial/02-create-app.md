<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6bf02-101">Dans cet exercice, vous allez utiliser [Django](https://www.djangoproject.com/) pour créer une application Web.</span><span class="sxs-lookup"><span data-stu-id="6bf02-101">In this exercise you will use [Django](https://www.djangoproject.com/) to build a web app.</span></span>

1. <span data-ttu-id="6bf02-102">Si Django n’est pas encore installé, vous pouvez l’installer à partir de votre interface de ligne de commande (CLI) à l’aide de la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="6bf02-102">If you don't already have Django installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    pip install Django==3.0.4
    ```

1. <span data-ttu-id="6bf02-103">Ouvrez votre interface CLI, accédez à un répertoire où vous disposez des droits nécessaires pour créer des fichiers, puis exécutez la commande suivante pour créer une nouvelle application Django.</span><span class="sxs-lookup"><span data-stu-id="6bf02-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Django app.</span></span>

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. <span data-ttu-id="6bf02-104">Accédez au répertoire **graph_tutorial** et entrez la commande suivante pour démarrer un serveur Web local.</span><span class="sxs-lookup"><span data-stu-id="6bf02-104">Navigate to the **graph_tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    python manage.py runserver
    ```

1. <span data-ttu-id="6bf02-105">Ouvrez votre navigateur et accédez à `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="6bf02-105">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="6bf02-106">Si tout fonctionne, vous verrez une page d’accueil de Django.</span><span class="sxs-lookup"><span data-stu-id="6bf02-106">If everything is working, you will see a Django welcome page.</span></span> <span data-ttu-id="6bf02-107">Si ce n’est pas le cas, consultez le [Guide de prise](https://www.djangoproject.com/start/)en main de Django.</span><span class="sxs-lookup"><span data-stu-id="6bf02-107">If you don't see that, check the [Django getting started guide](https://www.djangoproject.com/start/).</span></span>

1. <span data-ttu-id="6bf02-108">Ajoutez une application au projet.</span><span class="sxs-lookup"><span data-stu-id="6bf02-108">Add an app to the project.</span></span> <span data-ttu-id="6bf02-109">Exécutez la commande suivante dans votre interface CLI.</span><span class="sxs-lookup"><span data-stu-id="6bf02-109">Run the following command in your CLI.</span></span>

    ```Shell
    python manage.py startapp tutorial
    ```

1. <span data-ttu-id="6bf02-110">Ouvrez **./graph_tutorial/Settings.py** et ajoutez la nouvelle `tutorial` application au `INSTALLED_APPS` paramètre.</span><span class="sxs-lookup"><span data-stu-id="6bf02-110">Open **./graph_tutorial/settings.py** and add the new `tutorial` app to the `INSTALLED_APPS` setting.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. <span data-ttu-id="6bf02-111">Dans votre interface CLI, exécutez la commande suivante pour initialiser la base de données pour le projet.</span><span class="sxs-lookup"><span data-stu-id="6bf02-111">In your CLI, run the following command to initialize the database for the project.</span></span>

    ```Shell
    python manage.py migrate
    ```

1. <span data-ttu-id="6bf02-112">Ajoutez un [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) pour l' `tutorial` application.</span><span class="sxs-lookup"><span data-stu-id="6bf02-112">Add a [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) for the `tutorial` app.</span></span> <span data-ttu-id="6bf02-113">Créez un fichier dans le répertoire **./Tutorial** nommé `urls.py` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="6bf02-113">Create a new file in the **./tutorial** directory named `urls.py` and add the following code.</span></span>

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

1. <span data-ttu-id="6bf02-114">Mettez à jour le projet URLconf pour importer celui-ci.</span><span class="sxs-lookup"><span data-stu-id="6bf02-114">Update the project URLconf to import this one.</span></span> <span data-ttu-id="6bf02-115">Ouvrez **./graph_tutorial/URLs.py** et remplacez le contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="6bf02-115">Open **./graph_tutorial/urls.py** and replace the contents with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. <span data-ttu-id="6bf02-116">Ajoutez une vue temporaire à l' `tutorials` application pour vérifier que le routage des URL fonctionne.</span><span class="sxs-lookup"><span data-stu-id="6bf02-116">Add a temporary view to the `tutorials` app to verify that URL routing is working.</span></span> <span data-ttu-id="6bf02-117">Ouvrez **./Tutorial/views.py** et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="6bf02-117">Open **./tutorial/views.py** and add the following code.</span></span>

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. <span data-ttu-id="6bf02-118">Enregistrez toutes vos modifications et redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="6bf02-118">Save all of your changes and restart the server.</span></span> <span data-ttu-id="6bf02-119">Accédez à `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="6bf02-119">Browse to `http://localhost:8000`.</span></span> <span data-ttu-id="6bf02-120">Vous devriez voir`Welcome to the tutorial.`</span><span class="sxs-lookup"><span data-stu-id="6bf02-120">You should see `Welcome to the tutorial.`</span></span>

## <a name="install-libraries"></a><span data-ttu-id="6bf02-121">Installation des bibliothèques</span><span class="sxs-lookup"><span data-stu-id="6bf02-121">Install libraries</span></span>

<span data-ttu-id="6bf02-122">Avant de poursuivre, installez des bibliothèques supplémentaires que vous utiliserez plus tard :</span><span class="sxs-lookup"><span data-stu-id="6bf02-122">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="6bf02-123">[Demandes-OAuthlib : OAuth pour les êtres humains](https://requests-oauthlib.readthedocs.io/en/latest/) pour la gestion des flux de connexion et de jetons OAuth, et pour passer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="6bf02-123">[Requests-OAuthlib: OAuth for Humans](https://requests-oauthlib.readthedocs.io/en/latest/) for handling sign-in and OAuth token flows, and for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="6bf02-124">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) pour le chargement de la configuration à partir d’un fichier YAML.</span><span class="sxs-lookup"><span data-stu-id="6bf02-124">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) for loading configuration from a YAML file.</span></span>
- <span data-ttu-id="6bf02-125">[python-dateutil](https://pypi.org/project/python-dateutil/) pour l’analyse des chaînes de date ISO 8601 renvoyées par Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="6bf02-125">[python-dateutil](https://pypi.org/project/python-dateutil/) for parsing ISO 8601 date strings returned from Microsoft Graph.</span></span>

1. <span data-ttu-id="6bf02-126">Exécutez la commande suivante dans votre interface CLI.</span><span class="sxs-lookup"><span data-stu-id="6bf02-126">Run the following command in your CLI.</span></span>

    ```Shell
    pip install requests_oauthlib==1.3.0
    pip install pyyaml==5.3.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a><span data-ttu-id="6bf02-127">Concevoir l’application</span><span class="sxs-lookup"><span data-stu-id="6bf02-127">Design the app</span></span>

1. <span data-ttu-id="6bf02-128">Créez un répertoire dans le répertoire **./Tutorial** nommé `templates`.</span><span class="sxs-lookup"><span data-stu-id="6bf02-128">Create a new directory in the **./tutorial** directory named `templates`.</span></span>

1. <span data-ttu-id="6bf02-129">Dans le répertoire **./Tutorial/templates** , créez un répertoire nommé `tutorial`.</span><span class="sxs-lookup"><span data-stu-id="6bf02-129">In the **./tutorial/templates** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="6bf02-130">Dans le répertoire **./Tutorial/templates/Tutorial** , créez un fichier nommé `layout.html`.</span><span class="sxs-lookup"><span data-stu-id="6bf02-130">In the **./tutorial/templates/tutorial** directory, create a new file named `layout.html`.</span></span> <span data-ttu-id="6bf02-131">Ajoutez le code suivant dans ce fichier.</span><span class="sxs-lookup"><span data-stu-id="6bf02-131">Add the following code in that file.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    <span data-ttu-id="6bf02-132">Ce code ajoute [Bootstrap](http://getbootstrap.com/) pour la mise en forme simple et [Font Awesome](https://fontawesome.com/) pour certaines icônes simples.</span><span class="sxs-lookup"><span data-stu-id="6bf02-132">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="6bf02-133">Il définit également une disposition globale avec une barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="6bf02-133">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="6bf02-134">Créez un répertoire dans le répertoire **./Tutorial** nommé `static`.</span><span class="sxs-lookup"><span data-stu-id="6bf02-134">Create a new directory in the **./tutorial** directory named `static`.</span></span>

1. <span data-ttu-id="6bf02-135">Dans le répertoire **./Tutorial/static** , créez un répertoire nommé `tutorial`.</span><span class="sxs-lookup"><span data-stu-id="6bf02-135">In the **./tutorial/static** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="6bf02-136">Dans le répertoire **./Tutorial/static/Tutorial** , créez un fichier nommé `app.css`.</span><span class="sxs-lookup"><span data-stu-id="6bf02-136">In the **./tutorial/static/tutorial** directory, create a new file named `app.css`.</span></span> <span data-ttu-id="6bf02-137">Ajoutez le code suivant dans ce fichier.</span><span class="sxs-lookup"><span data-stu-id="6bf02-137">Add the following code in that file.</span></span>

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. <span data-ttu-id="6bf02-138">Créez un modèle pour la page d’accueil qui utilise la mise en page.</span><span class="sxs-lookup"><span data-stu-id="6bf02-138">Create a template for the home page that uses the layout.</span></span> <span data-ttu-id="6bf02-139">Créez un fichier dans le répertoire **./Tutorial/templates/Tutorial** nommé `home.html` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="6bf02-139">Create a new file in the **./tutorial/templates/tutorial** directory named `home.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. <span data-ttu-id="6bf02-140">Ouvrez le `./tutorial/views.py` fichier et ajoutez la nouvelle fonction suivante.</span><span class="sxs-lookup"><span data-stu-id="6bf02-140">Open the `./tutorial/views.py` file and add the following new function.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. <span data-ttu-id="6bf02-141">Remplacez l’affichage `home` existant par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="6bf02-141">Replace the existing `home` view with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. <span data-ttu-id="6bf02-142">Enregistrez toutes vos modifications et redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="6bf02-142">Save all of your changes and restart the server.</span></span> <span data-ttu-id="6bf02-143">À présent, l’application doit être très différente.</span><span class="sxs-lookup"><span data-stu-id="6bf02-143">Now, the app should look very different.</span></span>

    ![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
