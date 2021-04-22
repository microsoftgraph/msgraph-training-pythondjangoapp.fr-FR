<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="0c86b-101">Dans cet exercice, vous allez étendre l'application de l'exercice précédent pour prendre en charge l'authentification avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="0c86b-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="0c86b-102">Cette étape est nécessaire pour obtenir le jeton d'accès OAuth nécessaire pour appeler Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="0c86b-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="0c86b-103">Dans cette étape, vous allez intégrer la [bibliothèque MSAL pour Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) dans l'application.</span><span class="sxs-lookup"><span data-stu-id="0c86b-103">In this step you will integrate the [MSAL for Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) library into the application.</span></span>

1. <span data-ttu-id="0c86b-104">Créez un fichier à la racine du projet nommé `oauth_settings.yml` et ajoutez le contenu suivant.</span><span class="sxs-lookup"><span data-stu-id="0c86b-104">Create a new file in the root of the project named `oauth_settings.yml`, and add the following content.</span></span>

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. <span data-ttu-id="0c86b-105">Remplacez `YOUR_APP_ID_HERE` par l'ID d'application à partir du portail d'inscription des applications et remplacez-le par le `YOUR_APP_SECRET_HERE` mot de passe que vous avez généré.</span><span class="sxs-lookup"><span data-stu-id="0c86b-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="0c86b-106">Si vous utilisez un contrôle source tel que Git, il est temps d'exclure le fichier **oauth_settings.yml** du contrôle source afin d'éviter toute fuite accidentelle de votre ID d'application et de votre mot de passe.</span><span class="sxs-lookup"><span data-stu-id="0c86b-106">If you're using source control such as git, now would be a good time to exclude the **oauth_settings.yml** file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="0c86b-107">Implémentation de la connexion</span><span class="sxs-lookup"><span data-stu-id="0c86b-107">Implement sign-in</span></span>

1. <span data-ttu-id="0c86b-108">Créez un fichier dans le **répertoire ./tutorial** nommé `auth_helper.py` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="0c86b-108">Create a new file in the **./tutorial** directory named `auth_helper.py` and add the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    <span data-ttu-id="0c86b-109">Ce fichier conservera toutes vos méthodes d'authentification.</span><span class="sxs-lookup"><span data-stu-id="0c86b-109">This file will hold all of your authentication-related methods.</span></span> <span data-ttu-id="0c86b-110">La méthode génère une URL d'autorisation et la méthode échange la réponse d'autorisation `get_sign_in_flow` `get_token_from_code` contre un jeton d'accès.</span><span class="sxs-lookup"><span data-stu-id="0c86b-110">The `get_sign_in_flow` generates an authorization URL, and the `get_token_from_code` method exchanges the authorization response for an access token.</span></span>

1. <span data-ttu-id="0c86b-111">Ajoutez les `import` instructions suivantes en haut de **./tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="0c86b-111">Add the following `import` statements to the top of **./tutorial/views.py**.</span></span>

    ```python
    from dateutil import tz, parser
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code
    ```

1. <span data-ttu-id="0c86b-112">Ajoutez un affichage de sign-in dans **le fichier ./tutorial/views.py.**</span><span class="sxs-lookup"><span data-stu-id="0c86b-112">Add a sign-in view in the **./tutorial/views.py** file.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. <span data-ttu-id="0c86b-113">Ajoutez un affichage de rappel dans **le fichier ./tutorial/views.py.**</span><span class="sxs-lookup"><span data-stu-id="0c86b-113">Add a callback view in the **./tutorial/views.py** file.</span></span>

    ```python
    def callback(request):
      # Make the token request
      result = get_token_from_code(request)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(result) }
      return HttpResponseRedirect(reverse('home'))
    ```

    <span data-ttu-id="0c86b-114">Prenez en compte les possibilités de ces affichages :</span><span class="sxs-lookup"><span data-stu-id="0c86b-114">Consider what these views do:</span></span>

    - <span data-ttu-id="0c86b-115">L'action génère l'URL de la signature Azure AD, enregistre le flux généré par le client OAuth, puis redirige le navigateur vers la page de signature `signin` Azure AD.</span><span class="sxs-lookup"><span data-stu-id="0c86b-115">The `signin` action generates the Azure AD signin URL, saves the flow generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    - <span data-ttu-id="0c86b-116">`callback`L'action est l'endroit où Azure redirige une fois la signature terminée.</span><span class="sxs-lookup"><span data-stu-id="0c86b-116">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="0c86b-117">Cette action utilise le flux enregistré et la chaîne de requête envoyée par Azure pour demander un jeton d'accès.</span><span class="sxs-lookup"><span data-stu-id="0c86b-117">That action uses the saved flow and the query string sent by Azure to request an access token.</span></span> <span data-ttu-id="0c86b-118">Il redirige ensuite vers la page d'accueil avec la réponse dans la valeur d'erreur temporaire.</span><span class="sxs-lookup"><span data-stu-id="0c86b-118">It then redirects back to the home page with the response in the temporary error value.</span></span> <span data-ttu-id="0c86b-119">Vous l'utiliserez pour vérifier que notre connectez-vous fonctionne avant de passer à autre chose.</span><span class="sxs-lookup"><span data-stu-id="0c86b-119">You'll use this to verify that our sign-in is working before moving on.</span></span>

1. <span data-ttu-id="0c86b-120">Ouvrez **./tutorial/urls.py** et remplacez les `path` instructions existantes `signin` par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="0c86b-120">Open **./tutorial/urls.py** and replace the existing `path` statements for `signin` with the following.</span></span>

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. <span data-ttu-id="0c86b-121">Ajoutez une nouvelle `path` valeur pour `callback` l'affichage.</span><span class="sxs-lookup"><span data-stu-id="0c86b-121">Add a new `path` for the `callback` view.</span></span>

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. <span data-ttu-id="0c86b-122">Démarrez le serveur et accédez à `https://localhost:8000` .</span><span class="sxs-lookup"><span data-stu-id="0c86b-122">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="0c86b-123">Cliquez sur le bouton de connexion. vous serez redirigé vers `https://login.microsoftonline.com`.</span><span class="sxs-lookup"><span data-stu-id="0c86b-123">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="0c86b-124">Connectez-vous avec votre compte Microsoft et consentez aux autorisations demandées.</span><span class="sxs-lookup"><span data-stu-id="0c86b-124">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="0c86b-125">Le navigateur redirige vers l'application, affichant la réponse, y compris le jeton d'accès.</span><span class="sxs-lookup"><span data-stu-id="0c86b-125">The browser redirects to the app, showing the response, including the access token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="0c86b-126">Obtenir les détails de l’utilisateur</span><span class="sxs-lookup"><span data-stu-id="0c86b-126">Get user details</span></span>

1. <span data-ttu-id="0c86b-127">Créez un fichier dans le **répertoire ./tutorial** nommé `graph_helper.py` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="0c86b-127">Create a new file in the **./tutorial** directory named `graph_helper.py` and add the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    <span data-ttu-id="0c86b-128">La méthode effectue une requête GET au point de terminaison Microsoft Graph pour obtenir le profil de l'utilisateur, à l'aide du jeton d'accès que vous `get_user` `/me` avez acquis précédemment.</span><span class="sxs-lookup"><span data-stu-id="0c86b-128">The `get_user` method makes a GET request to the Microsoft Graph `/me` endpoint to get the user's profile, using the access token you acquired previously.</span></span>

1. <span data-ttu-id="0c86b-129">Mettez à `callback` jour la méthode dans **./tutorial/views.py** pour obtenir le profil de l'utilisateur à partir de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="0c86b-129">Update the `callback` method in **./tutorial/views.py** to get the user's profile from Microsoft Graph.</span></span> <span data-ttu-id="0c86b-130">Ajoutez l’instruction `import` suivante en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="0c86b-130">Add the following `import` statement to the top of the file.</span></span>

    ```python
    from tutorial.graph_helper import *
    ```

1. <span data-ttu-id="0c86b-131">Remplacez la méthode `callback` par le code ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="0c86b-131">Replace the `callback` method with the following code.</span></span>

    ```python
    def callback(request):
      # Make the token request
      result = get_token_from_code(request)

      #Get the user's profile
      user = get_user(result['access_token'])
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': 'User: {0}\nToken: {1}'.format(user, result) }
      return HttpResponseRedirect(reverse('home'))
    ```

    <span data-ttu-id="0c86b-132">Le nouveau code appelle la `get_user` méthode pour demander le profil de l'utilisateur.</span><span class="sxs-lookup"><span data-stu-id="0c86b-132">The new code calls the `get_user` method to request the user's profile.</span></span> <span data-ttu-id="0c86b-133">Il ajoute l'objet utilisateur à la sortie temporaire pour le test.</span><span class="sxs-lookup"><span data-stu-id="0c86b-133">It adds the user object to the temporary output for testing.</span></span>

1. <span data-ttu-id="0c86b-134">Ajoutez les nouvelles méthodes suivantes **à ./tutorial/auth_helper.py**.</span><span class="sxs-lookup"><span data-stu-id="0c86b-134">Add the following new methods to **./tutorial/auth_helper.py**.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="SecondCodeSnippet":::

1. <span data-ttu-id="0c86b-135">Mettez à `callback` jour la fonction dans **./tutorial/views.py** pour stocker l'utilisateur dans la session et rediriger vers la page principale.</span><span class="sxs-lookup"><span data-stu-id="0c86b-135">Update the `callback` function in **./tutorial/views.py** to store the user in the session and redirect back to the main page.</span></span> <span data-ttu-id="0c86b-136">Remplacez `from tutorial.auth_helper import get_sign_in_flow, get_token_from_code` la ligne par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="0c86b-136">Replace the `from tutorial.auth_helper import get_sign_in_flow, get_token_from_code` line with the following.</span></span>

    ```python
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code, store_user, remove_user_and_token, get_token
    ```

1. <span data-ttu-id="0c86b-137">Remplacez `callback` la méthode par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="0c86b-137">Replace the `callback` method with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="0c86b-138">Implémenter la signature</span><span class="sxs-lookup"><span data-stu-id="0c86b-138">Implement sign-out</span></span>

1. <span data-ttu-id="0c86b-139">Ajoutez un nouvel `sign_out` affichage **dans ./tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="0c86b-139">Add a new `sign_out` view in **./tutorial/views.py**.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. <span data-ttu-id="0c86b-140">Ouvrez **./tutorial/urls.py** et remplacez les `path` instructions existantes `signout` par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="0c86b-140">Open **./tutorial/urls.py** and replace the existing `path` statements for `signout` with the following.</span></span>

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. <span data-ttu-id="0c86b-141">Redémarrez le serveur et traversez le processus de sign-in.</span><span class="sxs-lookup"><span data-stu-id="0c86b-141">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="0c86b-142">Vous devez revenir sur la page d'accueil, mais l'interface utilisateur doit changer pour indiquer que vous êtes bien inscrit.</span><span class="sxs-lookup"><span data-stu-id="0c86b-142">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. <span data-ttu-id="0c86b-144">Cliquez sur l'avatar de l'utilisateur dans le coin supérieur droit pour accéder au lien **de** connexion.</span><span class="sxs-lookup"><span data-stu-id="0c86b-144">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="0c86b-145">Le fait de cliquer sur **Se déconnecter** réinitialise la session et vous ramène à la page d’accueil.</span><span class="sxs-lookup"><span data-stu-id="0c86b-145">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Capture d’écran du menu déroulant avec le lien de déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="0c86b-147">Actualisation des jetons</span><span class="sxs-lookup"><span data-stu-id="0c86b-147">Refreshing tokens</span></span>

<span data-ttu-id="0c86b-148">À ce stade, votre application dispose d'un jeton d'accès, qui est envoyé dans l'en-tête des `Authorization` appels d'API.</span><span class="sxs-lookup"><span data-stu-id="0c86b-148">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="0c86b-149">Il s'agit du jeton qui permet à l'application d'accéder à Microsoft Graph au nom de l'utilisateur.</span><span class="sxs-lookup"><span data-stu-id="0c86b-149">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="0c86b-150">Cependant, ce jeton est de courte durée.</span><span class="sxs-lookup"><span data-stu-id="0c86b-150">However, this token is short-lived.</span></span> <span data-ttu-id="0c86b-151">Le jeton expire une heure après son émission.</span><span class="sxs-lookup"><span data-stu-id="0c86b-151">The token expires an hour after it is issued.</span></span> <span data-ttu-id="0c86b-152">C’est là que le jeton d’actualisation devient utile.</span><span class="sxs-lookup"><span data-stu-id="0c86b-152">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="0c86b-153">Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans obliger l’utilisateur à se reconnecter.</span><span class="sxs-lookup"><span data-stu-id="0c86b-153">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="0c86b-154">Étant donné que cet exemple utilise MSAL, vous n'avez pas besoin d'écrire de code spécifique pour actualiser le jeton.</span><span class="sxs-lookup"><span data-stu-id="0c86b-154">Because this sample uses MSAL, you do not have to write any specific code to refresh the token.</span></span> <span data-ttu-id="0c86b-155">La méthode MSAL gère `acquire_token_silent` l'actualisation du jeton si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="0c86b-155">MSAL's `acquire_token_silent` method handles refreshing the token if needed.</span></span>
