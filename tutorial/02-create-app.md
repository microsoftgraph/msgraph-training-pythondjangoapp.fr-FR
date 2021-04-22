<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="8c6c6-101">Dans cet exercice, vous allez utiliser [Django](https://www.djangoproject.com/) pour créer une application web.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-101">In this exercise you will use [Django](https://www.djangoproject.com/) to build a web app.</span></span>

1. <span data-ttu-id="8c6c6-102">Si Django n'est pas déjà installé, vous pouvez l'installer à partir de votre interface de ligne de commande (CLI) avec la commande suivante.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-102">If you don't already have Django installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    pip install Django==3.2
    ```

1. <span data-ttu-id="8c6c6-103">Ouvrez votre CLI, accédez à un répertoire dans lequel vous avez le droit de créer des fichiers et exécutez la commande suivante pour créer une application Django.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Django app.</span></span>

    ```Shell
    django-admin startproject graph_tutorial
    ```

1. <span data-ttu-id="8c6c6-104">Accédez au **répertoire graph_tutorial** et entrez la commande suivante pour démarrer un serveur web local.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-104">Navigate to the **graph_tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    python manage.py runserver
    ```

1. <span data-ttu-id="8c6c6-105">Ouvrez votre navigateur et accédez à `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-105">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="8c6c6-106">Si tout fonctionne, vous verrez une page d'accueil Django.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-106">If everything is working, you will see a Django welcome page.</span></span> <span data-ttu-id="8c6c6-107">Si ce n'est pas le cas, consultez le guide de mise en [place de Django.](https://www.djangoproject.com/start/)</span><span class="sxs-lookup"><span data-stu-id="8c6c6-107">If you don't see that, check the [Django getting started guide](https://www.djangoproject.com/start/).</span></span>

1. <span data-ttu-id="8c6c6-108">Ajoutez une application au projet.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-108">Add an app to the project.</span></span> <span data-ttu-id="8c6c6-109">Exécutez la commande suivante dans votre CLI.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-109">Run the following command in your CLI.</span></span>

    ```Shell
    python manage.py startapp tutorial
    ```

1. <span data-ttu-id="8c6c6-110">Ouvrez **./graph_tutorial/settings.py** et ajoutez la nouvelle `tutorial` application au `INSTALLED_APPS` paramètre.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-110">Open **./graph_tutorial/settings.py** and add the new `tutorial` app to the `INSTALLED_APPS` setting.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/settings.py" id="InstalledAppsSnippet" highlight="8":::

1. <span data-ttu-id="8c6c6-111">Dans votre CLI, exécutez la commande suivante pour initialiser la base de données du projet.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-111">In your CLI, run the following command to initialize the database for the project.</span></span>

    ```Shell
    python manage.py migrate
    ```

1. <span data-ttu-id="8c6c6-112">Ajoutez [un URNfnf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) pour l'application. `tutorial`</span><span class="sxs-lookup"><span data-stu-id="8c6c6-112">Add a [URLconf](https://docs.djangoproject.com/en/3.0/topics/http/urls/) for the `tutorial` app.</span></span> <span data-ttu-id="8c6c6-113">Créez un fichier dans le **répertoire ./tutorial** nommé `urls.py` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-113">Create a new file in the **./tutorial** directory named `urls.py` and add the following code.</span></span>

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

1. <span data-ttu-id="8c6c6-114">Mettez à jour l'URNf du projet pour importer celui-ci.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-114">Update the project URLconf to import this one.</span></span> <span data-ttu-id="8c6c6-115">Ouvrez **./graph_tutorial/urls.py** et remplacez le contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-115">Open **./graph_tutorial/urls.py** and replace the contents with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/graph_tutorial/urls.py" id="UrlConfSnippet":::

1. <span data-ttu-id="8c6c6-116">Ajoutez une vue temporaire à `tutorials` l'application pour vérifier que le routage d'URL fonctionne.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-116">Add a temporary view to the `tutorials` app to verify that URL routing is working.</span></span> <span data-ttu-id="8c6c6-117">Ouvrez **./tutorial/views.py** et remplacez tout son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-117">Open **./tutorial/views.py** and replace its entire contents with the following code.</span></span>

    ```python
    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect
    from django.urls import reverse
    from datetime import datetime, timedelta

    def home(request):
      # Temporary!
      return HttpResponse("Welcome to the tutorial.")
    ```

1. <span data-ttu-id="8c6c6-118">Enregistrez toutes vos modifications et redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-118">Save all of your changes and restart the server.</span></span> <span data-ttu-id="8c6c6-119">Accédez `http://localhost:8000` à .</span><span class="sxs-lookup"><span data-stu-id="8c6c6-119">Browse to `http://localhost:8000`.</span></span> <span data-ttu-id="8c6c6-120">Vous devriez voir `Welcome to the tutorial.`</span><span class="sxs-lookup"><span data-stu-id="8c6c6-120">You should see `Welcome to the tutorial.`</span></span>

## <a name="install-libraries"></a><span data-ttu-id="8c6c6-121">Installation des bibliothèques</span><span class="sxs-lookup"><span data-stu-id="8c6c6-121">Install libraries</span></span>

<span data-ttu-id="8c6c6-122">Avant de passer à la suite, installez certaines bibliothèques supplémentaires que vous utiliserez ultérieurement :</span><span class="sxs-lookup"><span data-stu-id="8c6c6-122">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="8c6c6-123">[Bibliothèque d'authentification Microsoft (MSAL) pour Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) pour la gestion des flux de jeton OAuth et de la signature.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-123">[Microsoft Authentication Library (MSAL) for Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="8c6c6-124">[PyYAML pour le chargement](https://pyyaml.org/wiki/PyYAMLDocumentation) de la configuration à partir d'un fichier YAML.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-124">[PyYAML](https://pyyaml.org/wiki/PyYAMLDocumentation) for loading configuration from a YAML file.</span></span>
- <span data-ttu-id="8c6c6-125">[python-dateutil pour](https://pypi.org/project/python-dateutil/) l'utilisation des chaînes de date ISO 8601 renvoyées par Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-125">[python-dateutil](https://pypi.org/project/python-dateutil/) for parsing ISO 8601 date strings returned from Microsoft Graph.</span></span>

1. <span data-ttu-id="8c6c6-126">Exécutez la commande suivante dans votre CLI.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-126">Run the following command in your CLI.</span></span>

    ```Shell
    pip install msal==1.10.0
    pip install pyyaml==5.4.1
    pip install python-dateutil==2.8.1
    ```

## <a name="design-the-app"></a><span data-ttu-id="8c6c6-127">Concevoir l’application</span><span class="sxs-lookup"><span data-stu-id="8c6c6-127">Design the app</span></span>

1. <span data-ttu-id="8c6c6-128">Créez un répertoire dans le **répertoire ./tutorial** nommé `templates` .</span><span class="sxs-lookup"><span data-stu-id="8c6c6-128">Create a new directory in the **./tutorial** directory named `templates`.</span></span>

1. <span data-ttu-id="8c6c6-129">Dans le **répertoire ./tutorial/templates,** créez un répertoire nommé `tutorial` .</span><span class="sxs-lookup"><span data-stu-id="8c6c6-129">In the **./tutorial/templates** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="8c6c6-130">Dans le **répertoire ./tutorial/templates/tutorial,** créez un fichier nommé `layout.html` .</span><span class="sxs-lookup"><span data-stu-id="8c6c6-130">In the **./tutorial/templates/tutorial** directory, create a new file named `layout.html`.</span></span> <span data-ttu-id="8c6c6-131">Ajoutez le code suivant dans ce fichier.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-131">Add the following code in that file.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/layout.html" id="LayoutSnippet":::

    <span data-ttu-id="8c6c6-132">Ce code ajoute [Bootstrap pour](http://getbootstrap.com/) un style simple et [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) pour certaines icônes simples.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-132">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) for some simple icons.</span></span> <span data-ttu-id="8c6c6-133">Il définit également une disposition globale avec une barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-133">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="8c6c6-134">Créez un répertoire dans le **répertoire ./tutorial** nommé `static` .</span><span class="sxs-lookup"><span data-stu-id="8c6c6-134">Create a new directory in the **./tutorial** directory named `static`.</span></span>

1. <span data-ttu-id="8c6c6-135">Dans le **répertoire ./tutorial/static,** créez un répertoire nommé `tutorial` .</span><span class="sxs-lookup"><span data-stu-id="8c6c6-135">In the **./tutorial/static** directory, create a new directory named `tutorial`.</span></span>

1. <span data-ttu-id="8c6c6-136">Dans le **répertoire ./tutorial/static/tutorial,** créez un fichier nommé `app.css` .</span><span class="sxs-lookup"><span data-stu-id="8c6c6-136">In the **./tutorial/static/tutorial** directory, create a new file named `app.css`.</span></span> <span data-ttu-id="8c6c6-137">Ajoutez le code suivant dans ce fichier.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-137">Add the following code in that file.</span></span>

    :::code language="css" source="../demo/graph_tutorial/tutorial/static/tutorial/app.css":::

1. <span data-ttu-id="8c6c6-138">Créez un modèle pour la page d'accueil qui utilise la mise en page.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-138">Create a template for the home page that uses the layout.</span></span> <span data-ttu-id="8c6c6-139">Créez un fichier dans le répertoire **./tutorial/templates/tutorial** nommé `home.html` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-139">Create a new file in the **./tutorial/templates/tutorial** directory named `home.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/home.html" id="HomeSnippet":::

1. <span data-ttu-id="8c6c6-140">Ouvrez `./tutorial/views.py` le fichier et ajoutez la nouvelle fonction suivante.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-140">Open the `./tutorial/views.py` file and add the following new function.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="InitializeContextSnippet":::

1. <span data-ttu-id="8c6c6-141">Remplacez l'affichage `home` existant par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-141">Replace the existing `home` view with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="HomeViewSnippet":::

1. <span data-ttu-id="8c6c6-142">Ajoutez un fichier PNG nommé **no-profile-photo.png** dans le répertoire **./tutorial/static/tutorial.**</span><span class="sxs-lookup"><span data-stu-id="8c6c6-142">Add a PNG file named **no-profile-photo.png** in the **./tutorial/static/tutorial** directory.</span></span>

1. <span data-ttu-id="8c6c6-143">Enregistrez toutes vos modifications et redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-143">Save all of your changes and restart the server.</span></span> <span data-ttu-id="8c6c6-144">L'application doit maintenant avoir une apparence très différente.</span><span class="sxs-lookup"><span data-stu-id="8c6c6-144">Now, the app should look very different.</span></span>

    ![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
