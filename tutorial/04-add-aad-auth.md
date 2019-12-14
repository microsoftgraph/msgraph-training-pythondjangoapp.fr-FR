<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph. Dans cette étape, vous allez intégrer la bibliothèque [requêtes-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) dans l’application.

Créez un fichier à la racine du projet nommé `oauth_settings.yml`et ajoutez le contenu suivant.

```text
app_id: "YOUR_APP_ID_HERE"
app_secret: "YOUR_APP_PASSWORD_HERE"
redirect: "http://localhost:8000/tutorial/callback"
scopes: "openid profile offline_access user.read calendars.read"
authority: "https://login.microsoftonline.com/common"
authorize_endpoint: "/oauth2/v2.0/authorize"
token_endpoint: "/oauth2/v2.0/token"
```

Remplacez `YOUR_APP_ID_HERE` par l’ID de l’application dans le portail d’inscription de `YOUR_APP_SECRET_HERE` l’application et remplacez par le mot de passe que vous avez généré.

> [!IMPORTANT]
> Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le `oauth_settings.yml` fichier du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application et de votre mot de passe.

## <a name="implement-sign-in"></a>Mettre en œuvre la connexion

Créez un fichier dans le `./tutorial` répertoire nommé `auth_helper.py` et ajoutez le code suivant.

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
settings = yaml.load(stream, yaml.SafeLoader)
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

Ce fichier contiendra toutes vos méthodes d’authentification. L `get_sign_in_url` 'génère une URL d’autorisation et `get_token_from_code` la méthode échange la réponse d’autorisation pour un jeton d’accès.

Ajoutez les instructions `import` suivantes en haut de `./tutorial/views.py`.

```python
from django.urls import reverse
from tutorial.auth_helper import get_sign_in_url, get_token_from_code
```

À présent, ajoutez quelques nouvelles vues dans le `./tutorial/views.py` fichier.

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

Cette définition définit deux nouvelles vues `signin` : `callback`et.

L' `signin` action génère l’URL de connexion Azure ad, enregistre la `state` valeur générée par le client OAuth, puis redirige le navigateur vers la page de connexion Azure ad.

L' `callback` action est l’endroit où Azure redirige une fois la connexion terminée. Cette action permet de s' `state` assurer que la valeur correspond à la valeur enregistrée, puis utilise le code d’autorisation envoyé par Azure pour demander un jeton d’accès. Il redirige ensuite vers la page d’accueil avec le jeton d’accès dans la valeur d’erreur temporaire. Vous l’utiliserez pour vérifier que notre connexion fonctionne avant de poursuivre. Avant de procéder au test, vous devez ajouter les vues `./tutorial/urls.py`à.

```python
path('signin', views.sign_in, name='signin'),
path('callback', views.callback, name='callback'),
```

Remplacez la `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` ligne `./tutorial/templates/tutorial/home.html` par ce qui suit.

```html
<a href="{% url 'signin' %}" class="btn btn-primary btn-large">Click here to sign in</a>
```

Remplacez la `<a href="#" class="nav-link">Sign In</a>` ligne `./tutorial/templates/tutorial/layout.html` par ce qui suit.

```html
<a href="{% url 'signin' %}" class="nav-link">Sign In</a>
```

Démarrez le serveur et accédez à `https://localhost:8000/tutorial`. Cliquez sur le bouton de connexion et vous devez être redirigé vers `https://login.microsoftonline.com`. Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées. Le navigateur redirige vers l’application en affichant le jeton.

### <a name="get-user-details"></a>Obtenir les détails de l’utilisateur

Créez un fichier dans le `./tutorial` répertoire nommé `graph_helper.py` et ajoutez le code suivant.

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

La `get_user` méthode effectue une requête get au point de terminaison `/me` Microsoft Graph pour obtenir le profil de l’utilisateur à l’aide du jeton d’accès que vous avez acquis précédemment.

Mettez à `callback` jour la `./tutorial/views.py` méthode dans pour obtenir le profil de l’utilisateur à partir de Microsoft Graph.

Tout d’abord, ajoutez `import` l’instruction suivante en haut du fichier.

```python
from tutorial.graph_helper import get_user
```

Remplacez la `callback` méthode par le code suivant.

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

Le nouveau code appelle la `get_user` méthode pour demander le profil de l’utilisateur. Il ajoute l’objet User à la sortie temporaire à des fins de test.

## <a name="storing-the-tokens"></a>Stockage des jetons

Maintenant que vous pouvez obtenir des jetons, il est temps de mettre en œuvre un moyen de les stocker dans l’application. Étant donné qu’il s’agit d’un exemple d’application, pour des raisons de simplicité, vous les stockerez dans la session. Une application réelle utiliserait une solution de stockage sécurisé plus fiable, comme une base de données.

Ajoutez les nouvelles méthodes suivantes à `./tutorial/auth_helper.py`.

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

Ensuite, mettez à `callback` jour la `./tutorial/views.py` fonction dans pour stocker les jetons dans la session et rediriger vers la page principale. Remplacez la `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` ligne par ce qui suit.

```python
from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
```

Remplacez la `callback` méthode par ce qui suit.

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

## <a name="implement-sign-out"></a>Mettre en œuvre la déconnexion

Avant de tester cette nouvelle fonctionnalité, ajoutez une méthode pour vous déconnecter. Ajoutez un nouvel `sign_out` affichage dans `./tutorial/views.py`.

```python
def sign_out(request):
  # Clear out the user and token
  remove_user_and_token(request)

  return HttpResponseRedirect(reverse('home'))
```

À présent, ajoutez cette `./tutorial/urls.py`vue à.

```python
path('signout', views.sign_out, name='signout'),
```

Mettez à jour le lien **déconnexion** dans `./tutorial/templates/tutorial/layout.html` pour utiliser cette nouvelle vue. Remplacez la `<a href="#" class="dropdown-item">Sign Out</a>` ligne par ce qui suit.

```html
<a href="{% url 'signout' %}" class="dropdown-item">Sign Out</a>
```

Redémarrez le serveur et suivez le processus de connexion. Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.

![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour accéder au lien **déconnexion** . Cliquez sur **déconnexion** pour réinitialiser la session et revenir à la page d’accueil.

![Capture d’écran du menu déroulant avec le lien Déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualisation des jetons

À ce stade, votre application a un jeton d’accès, qui est envoyé `Authorization` dans l’en-tête des appels d’API. Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.

Toutefois, ce jeton est éphémère. Le jeton expire une heure après son émission. C’est ici que le jeton d’actualisation devient utile. Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans demander à l’utilisateur de se reconnecter. Mettez à jour le code de gestion des jetons pour implémenter l’actualisation des jetons.

Remplacez la méthode `get_token` `./tutorial/auth_helper.py` existante par ce qui suit.

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

Cette méthode vérifie d’abord si le jeton d’accès a expiré ou s’il arrive à expiration. Si c’est le cas, il utilise le jeton d’actualisation pour obtenir de nouveaux jetons, puis il met à jour le cache et renvoie le nouveau jeton d’accès.
