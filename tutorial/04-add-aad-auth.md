<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph. Dans cette étape, vous allez intégrer la bibliothèque [requêtes-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) dans l’application.

1. Créez un fichier à la racine du projet nommé `oauth_settings.yml`et ajoutez le contenu suivant.

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. Remplacez `YOUR_APP_ID_HERE` par l’ID de l’application dans le portail d’inscription de `YOUR_APP_SECRET_HERE` l’application et remplacez par le mot de passe que vous avez généré.

> [!IMPORTANT]
> Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le fichier **oauth_settings. yml** du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application et de votre mot de passe.

## <a name="implement-sign-in"></a>Implémentation de la connexion

1. Créez un fichier dans le répertoire **./Tutorial** nommé `auth_helper.py` et ajoutez le code suivant.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    Ce fichier contiendra toutes vos méthodes d’authentification. L `get_sign_in_url` 'génère une URL d’autorisation et `get_token_from_code` la méthode échange la réponse d’autorisation pour un jeton d’accès.

1. Ajoutez les instructions `import` suivantes en haut de **./Tutorial/views.py**.

    ```python
    from django.urls import reverse
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code
    ```

1. Ajoutez une vue de connexion dans le fichier **./Tutorial/views.py** .

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. Ajoutez une vue de rappel dans le fichier **./Tutorial/views.py** .

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

    Tenez compte de ce que ces affichages effectuent :

    - L' `signin` action génère l’URL de connexion Azure ad, enregistre la `state` valeur générée par le client OAuth, puis redirige le navigateur vers la page de connexion Azure ad.

    - L' `callback` action est l’endroit où Azure redirige une fois la connexion terminée. Cette action permet de s' `state` assurer que la valeur correspond à la valeur enregistrée, puis utilise le code d’autorisation envoyé par Azure pour demander un jeton d’accès. Il redirige ensuite vers la page d’accueil avec le jeton d’accès dans la valeur d’erreur temporaire. Vous l’utiliserez pour vérifier que notre connexion fonctionne avant de poursuivre.

1. Ouvrez **./Tutorial/URLs.py** et remplacez les instructions `path` existantes par `signin` , comme suit.

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. Ajoutez un nouveau `path` pour l' `callback` affichage.

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. Démarrez le serveur et accédez à `https://localhost:8000`. Cliquez sur le bouton de connexion. vous serez redirigé vers `https://login.microsoftonline.com`. Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées. Le navigateur vous redirige vers l’application, affichant le jeton.

### <a name="get-user-details"></a>Obtenir les détails de l’utilisateur

1. Créez un fichier dans le répertoire **./Tutorial** nommé `graph_helper.py` et ajoutez le code suivant.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    La `get_user` méthode effectue une requête get au point de terminaison `/me` Microsoft Graph pour obtenir le profil de l’utilisateur à l’aide du jeton d’accès que vous avez acquis précédemment.

1. Mettez à `callback` jour la méthode dans **./Tutorial/views.py** pour obtenir le profil de l’utilisateur à partir de Microsoft Graph. Ajoutez l’instruction `import` suivante en haut du fichier.

    ```python
    from tutorial.graph_helper import get_user
    ```

1. Remplacez la méthode `callback` par le code ci-dessous.

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

Maintenant que vous pouvez obtenir des jetons, nous vous conseillons d’implémenter un moyen de les stocker dans l’application. Étant donné qu’il s’agit d’un exemple d’application, pour des raisons de simplicité, vous les stockerez dans la session. Une application réelle utilise une solution de stockage sécurisé plus fiable, comme une base de données.

1. Ajoutez les nouvelles méthodes suivantes à **./tutorial/auth_helper. py**.

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

1. Mettez à `callback` jour la fonction dans **./Tutorial/views.py** pour stocker les jetons dans la session et rediriger vers la page principale. Remplacez la `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` ligne par ce qui suit.

    ```python
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_token, store_user, remove_user_and_token, get_token
    ```

1. Remplacez la `callback` méthode par ce qui suit.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a>Mettre en œuvre la déconnexion

1. Ajoutez un nouvel `sign_out` affichage dans **./Tutorial/views.py**.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. Ouvrez **./Tutorial/URLs.py** et remplacez les instructions `path` existantes par `signout` , comme suit.

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. Redémarrez le serveur et suivez le processus de connexion. Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour accéder au lien **déconnexion** . Le fait de cliquer sur **Se déconnecter** réinitialise la session et vous ramène à la page d’accueil.

    ![Capture d’écran du menu déroulant avec le lien de déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualisation des jetons

À ce stade, votre application a un jeton d’accès, qui est envoyé `Authorization` dans l’en-tête des appels d’API. Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.

Cependant, ce jeton est de courte durée. Le jeton expire une heure après son émission. C’est là que le jeton d’actualisation devient utile. Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans obliger l’utilisateur à se reconnecter. Mettez à jour le code de gestion des jetons pour implémenter l’actualisation des jetons.

1. Remplacez la méthode `get_token` existante dans **./Tutorial/auth_helper. py** par ce qui suit.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="GetTokenSnippet":::

    Cette méthode vérifie d’abord si le jeton d’accès a expiré ou s’il arrive à expiration. Si c’est le cas, il utilise le jeton d’actualisation pour obtenir de nouveaux jetons, puis il met à jour le cache et renvoie le nouveau jeton d’accès.
