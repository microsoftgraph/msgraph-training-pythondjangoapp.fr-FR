<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="8fc37-101">Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="8fc37-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="8fc37-102">Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="8fc37-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="8fc37-103">Dans cette étape, vous allez intégrer la bibliothèque [requêtes-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) dans l’application.</span><span class="sxs-lookup"><span data-stu-id="8fc37-103">In this step you will integrate the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library into the application.</span></span>

<span data-ttu-id="8fc37-104">Créez un fichier à la racine du projet nommé `oauth_settings.yml`et ajoutez le contenu suivant.</span><span class="sxs-lookup"><span data-stu-id="8fc37-104">Create a new file in the root of the project named `oauth_settings.yml`, and add the following content.</span></span>

```text
app_id: YOUR_APP_ID_HERE
app_secret: YOUR_APP_PASSWORD_HERE
redirect: http://localhost:8000/tutorial/callback
scopes: openid profile offline_access user.read calendars.read
authority: https://login.microsoftonline.com/common
authorize_endpoint: /oauth2/v2.0/authorize
token_endpoint: /oauth2/v2.0/token
```

<span data-ttu-id="8fc37-105">Remplacez `YOUR_APP_ID_HERE` par l’ID de l’application dans le portail d’inscription de `YOUR_APP_SECRET_HERE` l’application et remplacez par le mot de passe que vous avez généré.</span><span class="sxs-lookup"><span data-stu-id="8fc37-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="8fc37-106">Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le `oauth_settings.yml` fichier du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application et de votre mot de passe.</span><span class="sxs-lookup"><span data-stu-id="8fc37-106">If you're using source control such as git, now would be a good time to exclude the `oauth_settings.yml` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="8fc37-107">Mettre en œuvre la connexion</span><span class="sxs-lookup"><span data-stu-id="8fc37-107">Implement sign-in</span></span>

<span data-ttu-id="8fc37-108">Créez un fichier dans le `./tutorial` répertoire nommé `auth_helper.py` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="8fc37-108">Create a new file in the `./tutorial` directory named `auth_helper.py` and add the following code.</span></span>

```python
import yaml
from requests_oauthlib import OAuth2Session
import os
import time

# This is necessary for testing with non-HTTPS localhost
# Remove this if deploying to production
os.environ['OAUTHLIB_INSECURE_TRANSPORT'] = '1'

# This is necessary because Azure does not guarantee
# to return scopes in the same case and order as requested
os.environ['OAUTHLIB_RELAX_TOKEN_SCOPE'] = '1'
os.environ['OAUTHLIB_IGNORE_SCOPE_CHANGE'] = '1'

# Load the oauth_settings.yml file
stream = open('oauth_settings.yml', 'r')
settings = yaml.load(stream)
authorize_url = '{0}{1}'.format(settings['authority'], settings['authorize_endpoint'])
token_url = '{0}{1}'.format(settings['authority'], settings['token_endpoint'])

# Method to generate a sign-in url
def get_sign_in_url():
  # Initialize the OAuth client
  aad_auth = OAuth2Session(settings['app_id'],
    scope=settings['scopes'],
    redirect_uri=settings['redirect'])

  sign_in_url, state = aad_auth.authorization_url(authorize_url, prompt='login')

  return sign_in_url, state

# Method to exchange auth code for access token
def get_token_from_code(callback_url, expected_state):
  # Initialize the OAuth client
  aad_auth = OAuth2Session(settings['app_id'],
    state=expected_state,
    scope=settings['scopes'],
    redirect_uri=settings['redirect'])

  token = aad_auth.fetch_token(token_url,
    client_secret = settings['app_secret'],
    authorization_response=callback_url)

  return token
```

<span data-ttu-id="8fc37-109">Ce fichier contiendra toutes vos méthodes d’authentification.</span><span class="sxs-lookup"><span data-stu-id="8fc37-109">This file will hold all of your authentication-related methods.</span></span> <span data-ttu-id="8fc37-110">L `get_sign_in_url` 'génère une URL d’autorisation et `get_token_from_code` la méthode échange la réponse d’autorisation pour un jeton d’accès.</span><span class="sxs-lookup"><span data-stu-id="8fc37-110">The `get_sign_in_url` generates an authorization URL, and the `get_token_from_code` method exchanges the authorization response for an access token.</span></span>

<span data-ttu-id="8fc37-111">Ajoutez les instructions `import` suivantes en haut de `./tutorial/views.py`.</span><span class="sxs-lookup"><span data-stu-id="8fc37-111">Add the following `import` statements to the top of `./tutorial/views.py`.</span></span>

```python
from django.urls import reverse
from tutorial.auth_helper import get_sign_in_url, get_token_from_code
```

<span data-ttu-id="8fc37-112">À présent, ajoutez quelques nouvelles vues dans le `./tutorial/views.py` fichier.</span><span class="sxs-lookup"><span data-stu-id="8fc37-112">Now add a couple of new views in the `./tutorial/views.py` file.</span></span>

```python
def sign_in(request):
  # Get the sign-in URL
  sign_in_url, state = get_sign_in_url()
  # Save the expected state so we can validate in the callback
  request.session['auth_state'] = state
  # Redirect to the Azure sign-in page
  return HttpResponseRedirect(sign_in_url)

def callback(request):
  # Get the state saved in session
  expected_state = request.session.pop('auth_state', '')
  # Make the token request
  token = get_token_from_code(request.get_full_path(), expected_state)
  # Temporary! Save the response in an error so it's displayed
  request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(token) }
  return HttpResponseRedirect(reverse('home'))
```

<span data-ttu-id="8fc37-113">Cette définition définit deux nouvelles vues `signin` : `callback`et.</span><span class="sxs-lookup"><span data-stu-id="8fc37-113">This defines two new views: `signin` and `callback`.</span></span>

<span data-ttu-id="8fc37-114">L' `signin` action génère l’URL de connexion Azure ad, enregistre la `state` valeur générée par le client OAuth, puis redirige le navigateur vers la page de connexion Azure ad.</span><span class="sxs-lookup"><span data-stu-id="8fc37-114">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

<span data-ttu-id="8fc37-115">L' `callback` action est l’endroit où Azure redirige une fois la connexion terminée.</span><span class="sxs-lookup"><span data-stu-id="8fc37-115">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="8fc37-116">Cette action permet de s' `state` assurer que la valeur correspond à la valeur enregistrée, puis utilise le code d’autorisation envoyé par Azure pour demander un jeton d’accès.</span><span class="sxs-lookup"><span data-stu-id="8fc37-116">That action makes sure the `state` value matches the saved value, then uses the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="8fc37-117">Il redirige ensuite vers la page d’accueil avec le jeton d’accès dans la valeur d’erreur temporaire.</span><span class="sxs-lookup"><span data-stu-id="8fc37-117">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="8fc37-118">Vous l’utiliserez pour vérifier que notre connexion fonctionne avant de poursuivre.</span><span class="sxs-lookup"><span data-stu-id="8fc37-118">You'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="8fc37-119">Avant de procéder au test, vous devez ajouter les vues `./tutorial/urls.py`à.</span><span class="sxs-lookup"><span data-stu-id="8fc37-119">Before you test, you need to add the views to `./tutorial/urls.py`.</span></span>

```python
path('signin', views.sign_in, name='signin'),
path('callback', views.callback, name='callback'),
```

<span data-ttu-id="8fc37-120">Remplacez la `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` ligne `./tutorial/templates/tutorial/home.html` par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="8fc37-120">Replace the `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` line in `./tutorial/templates/tutorial/home.html` with the following.</span></span>

```html
<a href="{% url 'signin' %}" class="btn btn-primary btn-large">Click here to sign in</a>
```

<span data-ttu-id="8fc37-121">Remplacez la `<a href="#" class="nav-link">Sign In</a>` ligne `./tutorial/templates/tutorial/layout.html` par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="8fc37-121">Replace the `<a href="#" class="nav-link">Sign In</a>` line in `./tutorial/templates/tutorial/layout.html` with the following.</span></span>

```html
<a href="{% url 'signin' %}" class="nav-link">Sign In</a>
```

<span data-ttu-id="8fc37-122">Démarrez le serveur et accédez à `https://localhost:8000/tutorial`.</span><span class="sxs-lookup"><span data-stu-id="8fc37-122">Start the server and browse to `https://localhost:8000/tutorial`.</span></span> <span data-ttu-id="8fc37-123">Cliquez sur le bouton de connexion et vous devez être redirigé vers `https://login.microsoftonline.com`.</span><span class="sxs-lookup"><span data-stu-id="8fc37-123">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="8fc37-124">Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées.</span><span class="sxs-lookup"><span data-stu-id="8fc37-124">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="8fc37-125">Le navigateur redirige vers l’application en affichant le jeton.</span><span class="sxs-lookup"><span data-stu-id="8fc37-125">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="8fc37-126">Obtenir les détails de l’utilisateur</span><span class="sxs-lookup"><span data-stu-id="8fc37-126">Get user details</span></span>

<span data-ttu-id="8fc37-127">Créez un fichier dans le `./tutorial` répertoire nommé `graph_helper.py` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="8fc37-127">Create a new file in the `./tutorial` directory named `graph_helper.py` and add the following code.</span></span>

```python
from requests_oauthlib import OAuth2Session

graph_url = 'https://graph.microsoft.com/v1.0'

def get_user(token):
  graph_client = OAuth2Session(token=token)
  # Send GET to /me
  user = graph_client.get('{0}/me'.format(graph_url))
  # Return the JSON result
  return user.json()
```

<span data-ttu-id="8fc37-128">La `get_user` méthode effectue une requête get au point de terminaison `/me` Microsoft Graph pour obtenir le profil de l’utilisateur à l’aide du jeton d’accès que vous avez acquis précédemment.</span><span class="sxs-lookup"><span data-stu-id="8fc37-128">The `get_user` method makes a GET request to the Microsoft Graph `/me` endpoint to get the user's profile, using the access token you acquired previously.</span></span>

<span data-ttu-id="8fc37-129">Mettez à `callback` jour la `./tutorial/views.py` méthode dans pour obtenir le profil de l’utilisateur à partir de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="8fc37-129">Update the `callback` method in `./tutorial/views.py` to get the user's profile from Microsoft Graph.</span></span>

<span data-ttu-id="8fc37-130">Tout d’abord, ajoutez `import` l’instruction suivante en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="8fc37-130">First, add the following `import` statement to the top of the file.</span></span>

```python
from tutorial.graph_helper import get_user
```

<span data-ttu-id="8fc37-131">Remplacez la `callback` méthode par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="8fc37-131">Replace the `callback` method with the following code.</span></span>

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

<span data-ttu-id="8fc37-132">Le nouveau code appelle la `get_user` méthode pour demander le profil de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="8fc37-132">The new code calls the `get_user` method to request the user's profile.</span></span> <span data-ttu-id="8fc37-133">Il ajoute l’objet User à la sortie temporaire à des fins de test.</span><span class="sxs-lookup"><span data-stu-id="8fc37-133">It adds the user object to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="8fc37-134">Stockage des jetons</span><span class="sxs-lookup"><span data-stu-id="8fc37-134">Storing the tokens</span></span>

<span data-ttu-id="8fc37-135">Maintenant que vous pouvez obtenir des jetons, il est temps de mettre en œuvre un moyen de les stocker dans l’application.</span><span class="sxs-lookup"><span data-stu-id="8fc37-135">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="8fc37-136">Étant donné qu’il s’agit d’un exemple d’application, pour des raisons de simplicité, vous les stockerez dans la session.</span><span class="sxs-lookup"><span data-stu-id="8fc37-136">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="8fc37-137">Une application réelle utiliserait une solution de stockage sécurisé plus fiable, comme une base de données.</span><span class="sxs-lookup"><span data-stu-id="8fc37-137">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="8fc37-138">Ajoutez les nouvelles méthodes suivantes à `./tutorial/auth_helper.py`.</span><span class="sxs-lookup"><span data-stu-id="8fc37-138">Add the following new methods to `./tutorial/auth_helper.py`.</span></span>

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

<span data-ttu-id="8fc37-139">Ensuite, mettez à `callback` jour la `./tutorial/views.py` fonction dans pour stocker les jetons dans la session et rediriger vers la page principale.</span><span class="sxs-lookup"><span data-stu-id="8fc37-139">Then, update the `callback` function in `./tutorial/views.py` to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="8fc37-140">Remplacez la `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` ligne par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="8fc37-140">Replace the `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` line with the following.</span></span>

```python
from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
```

<span data-ttu-id="8fc37-141">Remplacez la `callback` méthode par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="8fc37-141">Replace the `callback` method with the following.</span></span>

```python
def callback(request):
  # Get the state saved in session
  expected_state = request.session.pop('auth_state', '')
  # Make the token request
  token = get_token_from_code(request.get_full_path(), expected_state)

  # Get the user's profile
  user = get_user(token)

  # Save token and user
  store_token(request, token)
  store_user(request, user)

  return HttpResponseRedirect(reverse('home'))
```

## <a name="implement-sign-out"></a><span data-ttu-id="8fc37-142">Mettre en œuvre la déconnexion</span><span class="sxs-lookup"><span data-stu-id="8fc37-142">Implement sign-out</span></span>

<span data-ttu-id="8fc37-143">Avant de tester cette nouvelle fonctionnalité, ajoutez une méthode pour vous déconnecter. Ajoutez un nouvel `sign_out` affichage dans `./tutorial/views.py`.</span><span class="sxs-lookup"><span data-stu-id="8fc37-143">Before you test this new feature, add a way to sign out. Add a new `sign_out` view in `./tutorial/views.py`.</span></span>

```python
def sign_out(request):
  # Clear out the user and token
  remove_user_and_token(request)

  return HttpResponseRedirect(reverse('home'))
```

<span data-ttu-id="8fc37-144">À présent, ajoutez cette `./tutorial/urls.py`vue à.</span><span class="sxs-lookup"><span data-stu-id="8fc37-144">Now add this view to `./tutorial/urls.py`.</span></span>

```python
path('signout', views.sign_out, name='signout'),
```

<span data-ttu-id="8fc37-145">Mettez à \*\*\*\* jour le lien Déconnexion dans `./tutorial/templates/tutorial/layout.html` pour utiliser cette nouvelle vue.</span><span class="sxs-lookup"><span data-stu-id="8fc37-145">Update the **Sign Out** link in `./tutorial/templates/tutorial/layout.html` to use this new view.</span></span> <span data-ttu-id="8fc37-146">Remplacez la `<a href="#" class="dropdown-item">Sign Out</a>` ligne par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="8fc37-146">Replace the `<a href="#" class="dropdown-item">Sign Out</a>` line with the following.</span></span>

```html
<a href="{% url 'signout' %}" class="dropdown-item">Sign Out</a>
```

<span data-ttu-id="8fc37-147">Redémarrez le serveur et suivez le processus de connexion.</span><span class="sxs-lookup"><span data-stu-id="8fc37-147">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="8fc37-148">Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.</span><span class="sxs-lookup"><span data-stu-id="8fc37-148">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

<span data-ttu-id="8fc37-150">Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour \*\*\*\* accéder au lien Déconnexion.</span><span class="sxs-lookup"><span data-stu-id="8fc37-150">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="8fc37-151">Cliquez \*\*\*\* sur Déconnexion pour réinitialiser la session et revenir à la page d’accueil.</span><span class="sxs-lookup"><span data-stu-id="8fc37-151">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![Capture d’écran du menu déroulant avec le lien Déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="8fc37-153">Actualisation des jetons</span><span class="sxs-lookup"><span data-stu-id="8fc37-153">Refreshing tokens</span></span>

<span data-ttu-id="8fc37-154">À ce stade, votre application a un jeton d’accès, qui est envoyé `Authorization` dans l’en-tête des appels d’API.</span><span class="sxs-lookup"><span data-stu-id="8fc37-154">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="8fc37-155">Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="8fc37-155">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="8fc37-156">Toutefois, ce jeton est éphémère.</span><span class="sxs-lookup"><span data-stu-id="8fc37-156">However, this token is short-lived.</span></span> <span data-ttu-id="8fc37-157">Le jeton expire une heure après son émission.</span><span class="sxs-lookup"><span data-stu-id="8fc37-157">The token expires an hour after it is issued.</span></span> <span data-ttu-id="8fc37-158">C’est ici que le jeton d’actualisation devient utile.</span><span class="sxs-lookup"><span data-stu-id="8fc37-158">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="8fc37-159">Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans demander à l’utilisateur de se reconnecter.</span><span class="sxs-lookup"><span data-stu-id="8fc37-159">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="8fc37-160">Mettez à jour le code de gestion des jetons pour implémenter l’actualisation des jetons.</span><span class="sxs-lookup"><span data-stu-id="8fc37-160">Update the token management code to implement token refresh.</span></span>

<span data-ttu-id="8fc37-161">Remplacez la méthode `get_token` `./tutorial/auth_helper.py` existante par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="8fc37-161">Replace the existing `get_token` method in `./tutorial/auth_helper.py` with the following.</span></span>

```python
def get_token(request):
  token = request.session['oauth_token']
  if token != None:
    # Check expiration
    now = time.time()
    # Subtract 5 minutes from expiration to account for clock skew
    expire_time = token['expires_at'] - 300
    if now >= expire_time:
      # Refresh the token
      aad_auth = OAuth2Session(settings['app_id'],
        token = token,
        scope=settings['scopes'],
        redirect_uri=settings['redirect'])

      refresh_params = {
        'client_id': settings['app_id'],
        'client_secret': settings['app_secret'],
      }
      new_token = aad_auth.refresh_token(token_url, **refresh_params)

      # Save new token
      store_token(request, new_token)

      # Return new access token
      return new_token

    else:
      # Token still valid, just return it
      return token
```

<span data-ttu-id="8fc37-162">Cette méthode vérifie d’abord si le jeton d’accès a expiré ou s’il arrive à expiration.</span><span class="sxs-lookup"><span data-stu-id="8fc37-162">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="8fc37-163">Si c’est le cas, il utilise le jeton d’actualisation pour obtenir de nouveaux jetons, puis il met à jour le cache et renvoie le nouveau jeton d’accès.</span><span class="sxs-lookup"><span data-stu-id="8fc37-163">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>