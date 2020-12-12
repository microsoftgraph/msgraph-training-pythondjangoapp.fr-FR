<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler Microsoft Graph. Dans cette étape, vous allez intégrer la bibliothèque [MSAL pour Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) dans l’application.

1. Créez un fichier à la racine du projet nommé `oauth_settings.yml` et ajoutez le contenu suivant.

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. Remplacez `YOUR_APP_ID_HERE` par l’ID de l’application dans le portail d’inscription de l’application et remplacez `YOUR_APP_SECRET_HERE` par le mot de passe que vous avez généré.

> [!IMPORTANT]
> Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le fichier **oauth_settings. yml** du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application et de votre mot de passe.

## <a name="implement-sign-in"></a>Implémentation de la connexion

1. Créez un fichier dans le répertoire **./Tutorial** nommé `auth_helper.py` et ajoutez le code suivant.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    Ce fichier contiendra toutes vos méthodes d’authentification. L' `get_sign_in_flow` génère une URL d’autorisation et la `get_token_from_code` méthode échange la réponse d’autorisation pour un jeton d’accès.

1. Ajoutez l' `import` instruction suivante en haut de **./Tutorial/views.py**.

    ```python
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code
    ```

1. Ajoutez une vue de connexion dans le fichier **./Tutorial/views.py** .

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. Ajoutez une vue de rappel dans le fichier **./Tutorial/views.py** .

    ```python
    def callback(request):
      # Make the token request
      result = get_token_from_code(request)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(result) }
      return HttpResponseRedirect(reverse('home'))
    ```

    Tenez compte de ce que ces affichages effectuent :

    - L' `signin` action génère l’URL de connexion Azure ad, enregistre le flux généré par le client OAuth, puis redirige le navigateur vers la page de connexion Azure ad.

    - L' `callback` action est l’endroit où Azure redirige une fois la connexion terminée. Cette action utilise le flux enregistré et la chaîne de requête envoyées par Azure pour demander un jeton d’accès. Il redirige ensuite vers la page d’accueil avec la réponse dans la valeur d’erreur temporaire. Vous l’utiliserez pour vérifier que notre connexion fonctionne avant de poursuivre.

1. Ouvrez **./Tutorial/URLs.py** et remplacez les `path` instructions existantes par, `signin` comme suit.

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. Ajoutez un nouveau `path` pour l' `callback` affichage.

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. Démarrez le serveur et accédez à `https://localhost:8000` . Cliquez sur le bouton de connexion. vous serez redirigé vers `https://login.microsoftonline.com`. Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées. Le navigateur redirige vers l’application en affichant la réponse, y compris le jeton d’accès.

### <a name="get-user-details"></a>Obtenir les détails de l’utilisateur

1. Créez un fichier dans le répertoire **./Tutorial** nommé `graph_helper.py` et ajoutez le code suivant.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    La `get_user` méthode effectue une requête get au point de terminaison Microsoft Graph `/me` pour obtenir le profil de l’utilisateur à l’aide du jeton d’accès que vous avez acquis précédemment.

1. Mettez à jour la `callback` méthode dans **./Tutorial/views.py** pour obtenir le profil de l’utilisateur à partir de Microsoft Graph. Ajoutez l’instruction `import` suivante en haut du fichier.

    ```python
    from tutorial.graph_helper import *
    ```

1. Remplacez la méthode `callback` par le code ci-dessous.

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

    Le nouveau code appelle la `get_user` méthode pour demander le profil de l’utilisateur. Il ajoute l’objet User à la sortie temporaire à des fins de test.

1. Ajoutez les nouvelles méthodes suivantes à **./tutorial/auth_helper. py**.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="SecondCodeSnippet":::

1. Mettez à jour la `callback` fonction dans **./Tutorial/views.py** pour stocker l’utilisateur dans la session et rediriger vers la page principale. Remplacez la `from tutorial.auth_helper import get_sign_in_url, get_token_from_code` ligne par ce qui suit.

    ```python
    from tutorial.auth_helper import get_sign_in_url, get_token_from_code, store_user, remove_user_and_token, get_token
    ```

1. Remplacez la `callback` méthode par ce qui suit.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a>Mettre en œuvre la déconnexion

1. Ajoutez un nouvel `sign_out` affichage dans **./Tutorial/views.py**.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. Ouvrez **./Tutorial/URLs.py** et remplacez les `path` instructions existantes par, `signout` comme suit.

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. Redémarrez le serveur et suivez le processus de connexion. Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour accéder au lien **déconnexion** . Le fait de cliquer sur **Se déconnecter** réinitialise la session et vous ramène à la page d’accueil.

    ![Capture d’écran du menu déroulant avec le lien de déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualisation des jetons

À ce stade, votre application a un jeton d’accès, qui est envoyé dans l' `Authorization` en-tête des appels d’API. Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.

Cependant, ce jeton est de courte durée. Le jeton expire une heure après son émission. C’est là que le jeton d’actualisation devient utile. Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans obliger l’utilisateur à se reconnecter.

Étant donné que cet exemple utilise MSAL, vous n’avez pas besoin d’écrire un code spécifique pour actualiser le jeton. `acquire_token_silent`La méthode de MSAL gère l’actualisation du jeton si nécessaire.
