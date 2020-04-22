<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="cf69d-101">Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="cf69d-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="cf69d-102">Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="cf69d-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="cf69d-103">Dans cette étape, vous allez intégrer la bibliothèque [requêtes-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) dans l’application.</span><span class="sxs-lookup"><span data-stu-id="cf69d-103">In this step you will integrate the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library into the application.</span></span>

1. <span data-ttu-id="cf69d-104">Créez un fichier à la racine du projet nommé `oauth_settings.yml`et ajoutez le contenu suivant.</span><span class="sxs-lookup"><span data-stu-id="cf69d-104">Create a new file in the root of the project named `oauth_settings.yml`, and add the following content.</span></span>

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. <span data-ttu-id="cf69d-105">Remplacez `YOUR_APP_ID_HERE` par l’ID de l’application dans le portail d’inscription de `YOUR_APP_SECRET_HERE` l’application et remplacez par le mot de passe que vous avez généré.</span><span class="sxs-lookup"><span data-stu-id="cf69d-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="cf69d-106">Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le fichier **oauth_settings. yml** du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application et de votre mot de passe.</span><span class="sxs-lookup"><span data-stu-id="cf69d-106">If you're using source control such as git, now would be a good time to exclude the **oauth_settings.yml** file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="cf69d-107">Implémentation de la connexion</span><span class="sxs-lookup"><span data-stu-id="cf69d-107">Implement sign-in</span></span>

1. <span data-ttu-id="cf69d-108">Créez un fichier dans le répertoire **./Tutorial** nommé `auth_helper.py` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="cf69d-108">Create a new file in the **./tutorial** directory named `auth_helper.py` and add the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    <span data-ttu-id="cf69d-109">Ce fichier contiendra toutes vos méthodes d’authentification.</span><span class="sxs-lookup"><span data-stu-id="cf69d-109">This file will hold all of your authentication-related methods.</span></span> <span data-ttu-id="cf69d-110">L `get_sign_in_url` 'génère une URL d’autorisation et `get_token_from_code` la méthode échange la réponse d’autorisation pour un jeton d’accès.</span><span class="sxs-lookup"><span data-stu-id="cf69d-110">The `get_sign_in_url` generates an authorization URL, and the `get_token_from_code` method exchanges the authorization response for an access token.</span></span>

1. <span data-ttu-id="cf69d-111">Ajoutez les instructions `import` suivantes en haut de **./Tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="cf69d-111">Add the following `import` statements to the top of **./tutorial/views.py**.</span></span>

    ```python
    from django.urls import reverse
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code
    ```

1. <span data-ttu-id="cf69d-112">Ajoutez une vue de connexion dans le fichier **./Tutorial/views.py** .</span><span class="sxs-lookup"><span data-stu-id="cf69d-112">Add a sign-in view in the **./tutorial/views.py** file.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. <span data-ttu-id="cf69d-113">Ajoutez une vue de rappel dans le fichier **./Tutorial/views.py** .</span><span class="sxs-lookup"><span data-stu-id="cf69d-113">Add a callback view in the **./tutorial/views.py** file.</span></span>

    ```python
    def callback(request):
      # Get the state saved in session
      expected_state = request.session.pop('auth_state', '')
      # Make the token request
      token = get_token_from_code(request.get_full_path(), expected_state)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(token) }
      return HttpResponseRedirect(reverse('home'))
    ```

    <span data-ttu-id="cf69d-114">Tenez compte de ce que ces affichages effectuent :</span><span class="sxs-lookup"><span data-stu-id="cf69d-114">Consider what these views do:</span></span>

    - <span data-ttu-id="cf69d-115">L' `signin` action génère l’URL de connexion Azure ad, enregistre la `state` valeur générée par le client OAuth, puis redirige le navigateur vers la page de connexion Azure ad.</span><span class="sxs-lookup"><span data-stu-id="cf69d-115">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    - <span data-ttu-id="cf69d-116">L' `callback` action est l’endroit où Azure redirige une fois la connexion terminée.</span><span class="sxs-lookup"><span data-stu-id="cf69d-116">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="cf69d-117">Cette action permet de s' `state` assurer que la valeur correspond à la valeur enregistrée, puis utilise le code d’autorisation envoyé par Azure pour demander un jeton d’accès.</span><span class="sxs-lookup"><span data-stu-id="cf69d-117">That action makes sure the `state` value matches the saved value, then uses the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="cf69d-118">Il redirige ensuite vers la page d’accueil avec le jeton d’accès dans la valeur d’erreur temporaire.</span><span class="sxs-lookup"><span data-stu-id="cf69d-118">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="cf69d-119">Vous l’utiliserez pour vérifier que notre connexion fonctionne avant de poursuivre.</span><span class="sxs-lookup"><span data-stu-id="cf69d-119">You'll use this to verify that our sign-in is working before moving on.</span></span>

1. <span data-ttu-id="cf69d-120">Ouvrez **./Tutorial/URLs.py** et remplacez les instructions `path` existantes par `signin` , comme suit.</span><span class="sxs-lookup"><span data-stu-id="cf69d-120">Open **./tutorial/urls.py** and replace the existing `path` statements for `signin` with the following.</span></span>

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. <span data-ttu-id="cf69d-121">Ajoutez un nouveau `path` pour l' `callback` affichage.</span><span class="sxs-lookup"><span data-stu-id="cf69d-121">Add a new `path` for the `callback` view.</span></span>

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. <span data-ttu-id="cf69d-122">Démarrez le serveur et accédez à `https://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="cf69d-122">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="cf69d-123">Cliquez sur le bouton de connexion. vous serez redirigé vers `https://login.microsoftonline.com`.</span><span class="sxs-lookup"><span data-stu-id="cf69d-123">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="cf69d-124">Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées.</span><span class="sxs-lookup"><span data-stu-id="cf69d-124">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="cf69d-125">Le navigateur vous redirige vers l’application, affichant le jeton.</span><span class="sxs-lookup"><span data-stu-id="cf69d-125">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="cf69d-126">Obtenir les détails de l’utilisateur</span><span class="sxs-lookup"><span data-stu-id="cf69d-126">Get user details</span></span>

1. <span data-ttu-id="cf69d-127">Créez un fichier dans le répertoire **./Tutorial** nommé `graph_helper.py` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="cf69d-127">Create a new file in the **./tutorial** directory named `graph_helper.py` and add the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    <span data-ttu-id="cf69d-128">La `get_user` méthode effectue une requête get au point de terminaison `/me` Microsoft Graph pour obtenir le profil de l’utilisateur à l’aide du jeton d’accès que vous avez acquis précédemment.</span><span class="sxs-lookup"><span data-stu-id="cf69d-128">The `get_user` method makes a GET request to the Microsoft Graph `/me` endpoint to get the user's profile, using the access token you acquired previously.</span></span>

1. <span data-ttu-id="cf69d-129">Mettez à `callback` jour la méthode dans **./Tutorial/views.py** pour obtenir le profil de l’utilisateur à partir de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="cf69d-129">Update the `callback` method in **./tutorial/views.py** to get the user's profile from Microsoft Graph.</span></span> <span data-ttu-id="cf69d-130">Ajoutez l’instruction `import` suivante en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="cf69d-130">Add the following `import` statement to the top of the file.</span></span>

    ```python
    from tutorial.graph_helper import get_user
    ```

1. <span data-ttu-id="cf69d-131">Remplacez la méthode `callback` par le code ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="cf69d-131">Replace the `callback` method with the following code.</span></span>

    ```python
    def callback(request):
      # Get the state saved in session
      expected_state = request.session.pop('auth_state', '')
      # Make the token request
      token = get_token_from_code(request.get_full_path(), expected_state)

      # Get the user's profile
      user = get_user(token)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved',
        'debug': 'User: {0}\nToken: {1}'.format(user, token) }
      return HttpResponseRedirect(reverse('home'))
    ```

<span data-ttu-id="cf69d-132">Le nouveau code appelle la `get_user` méthode pour demander le profil de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="cf69d-132">The new code calls the `get_user` method to request the user's profile.</span></span> <span data-ttu-id="cf69d-133">Il ajoute l’objet User à la sortie temporaire à des fins de test.</span><span class="sxs-lookup"><span data-stu-id="cf69d-133">It adds the user object to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="cf69d-134">Stockage des jetons</span><span class="sxs-lookup"><span data-stu-id="cf69d-134">Storing the tokens</span></span>

<span data-ttu-id="cf69d-135">Maintenant que vous pouvez obtenir des jetons, nous vous conseillons d’implémenter un moyen de les stocker dans l’application.</span><span class="sxs-lookup"><span data-stu-id="cf69d-135">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="cf69d-136">Étant donné qu’il s’agit d’un exemple d’application, pour des raisons de simplicité, vous les stockerez dans la session.</span><span class="sxs-lookup"><span data-stu-id="cf69d-136">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="cf69d-137">Une application réelle utilise une solution de stockage sécurisé plus fiable, comme une base de données.</span><span class="sxs-lookup"><span data-stu-id="cf69d-137">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="cf69d-138">Ajoutez les nouvelles méthodes suivantes à **./tutorial/auth_helper. py**.</span><span class="sxs-lookup"><span data-stu-id="cf69d-138">Add the following new methods to **./tutorial/auth_helper.py**.</span></span>

    ```python
    def store_token(request, token):
      request.session['oauth_token'] = token

    def store_user(request, user):
      request.session['user'] = {
        'is_authenticated': True,
        'name': user['displayName'],
        'email': user['mail'] if (user['mail'] != None) else user['userPrincipalName']
      }

    def get_token(request):
      token = request.session['oauth_token']
      return token

    def remove_user_and_token(request):
      if 'oauth_token' in request.session:
        del request.session['oauth_token']

      if 'user' in request.session:
        del request.session['user']
    ```

1. <span data-ttu-id="cf69d-139">Mettez à `callback` jour la fonction dans **./Tutorial/views.py** pour stocker les jetons dans la session et rediriger vers la page principale.</span><span class="sxs-lookup"><span data-stu-id="cf69d-139">Update the `callback` function in **./tutorial/views.py** to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="cf69d-140">Remplacez la `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` ligne par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="cf69d-140">Replace the `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` line with the following.</span></span>

    ```python
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
    ```

1. <span data-ttu-id="cf69d-141">Remplacez la `callback` méthode par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="cf69d-141">Replace the `callback` method with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="cf69d-142">Mettre en œuvre la déconnexion</span><span class="sxs-lookup"><span data-stu-id="cf69d-142">Implement sign-out</span></span>

1. <span data-ttu-id="cf69d-143">Ajoutez un nouvel `sign_out` affichage dans **./Tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="cf69d-143">Add a new `sign_out` view in **./tutorial/views.py**.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. <span data-ttu-id="cf69d-144">Ouvrez **./Tutorial/URLs.py** et remplacez les instructions `path` existantes par `signout` , comme suit.</span><span class="sxs-lookup"><span data-stu-id="cf69d-144">Open **./tutorial/urls.py** and replace the existing `path` statements for `signout` with the following.</span></span>

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. <span data-ttu-id="cf69d-145">Redémarrez le serveur et suivez le processus de connexion.</span><span class="sxs-lookup"><span data-stu-id="cf69d-145">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="cf69d-146">Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.</span><span class="sxs-lookup"><span data-stu-id="cf69d-146">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. <span data-ttu-id="cf69d-148">Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour accéder au lien **déconnexion** .</span><span class="sxs-lookup"><span data-stu-id="cf69d-148">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="cf69d-149">Le fait de cliquer sur **Se déconnecter** réinitialise la session et vous ramène à la page d’accueil.</span><span class="sxs-lookup"><span data-stu-id="cf69d-149">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Capture d’écran du menu déroulant avec le lien de déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="cf69d-151">Actualisation des jetons</span><span class="sxs-lookup"><span data-stu-id="cf69d-151">Refreshing tokens</span></span>

<span data-ttu-id="cf69d-152">À ce stade, votre application a un jeton d’accès, qui est envoyé `Authorization` dans l’en-tête des appels d’API.</span><span class="sxs-lookup"><span data-stu-id="cf69d-152">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="cf69d-153">Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="cf69d-153">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="cf69d-154">Cependant, ce jeton est de courte durée.</span><span class="sxs-lookup"><span data-stu-id="cf69d-154">However, this token is short-lived.</span></span> <span data-ttu-id="cf69d-155">Le jeton expire une heure après son émission.</span><span class="sxs-lookup"><span data-stu-id="cf69d-155">The token expires an hour after it is issued.</span></span> <span data-ttu-id="cf69d-156">C’est là que le jeton d’actualisation devient utile.</span><span class="sxs-lookup"><span data-stu-id="cf69d-156">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="cf69d-157">Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans obliger l’utilisateur à se reconnecter.</span><span class="sxs-lookup"><span data-stu-id="cf69d-157">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="cf69d-158">Mettez à jour le code de gestion des jetons pour implémenter l’actualisation des jetons.</span><span class="sxs-lookup"><span data-stu-id="cf69d-158">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="cf69d-159">Remplacez la méthode `get_token` existante dans **./Tutorial/auth_helper. py** par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="cf69d-159">Replace the existing `get_token` method in **./tutorial/auth_helper.py** with the following.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="GetTokenSnippet":::

    <span data-ttu-id="cf69d-160">Cette méthode vérifie d’abord si le jeton d’accès a expiré ou s’il arrive à expiration.</span><span class="sxs-lookup"><span data-stu-id="cf69d-160">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="cf69d-161">Si c’est le cas, il utilise le jeton d’actualisation pour obtenir de nouveaux jetons, puis il met à jour le cache et renvoie le nouveau jeton d’accès.</span><span class="sxs-lookup"><span data-stu-id="cf69d-161">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
