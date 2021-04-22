<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l'application de l'exercice précédent pour prendre en charge l'authentification avec Azure AD. Cette étape est nécessaire pour obtenir le jeton d'accès OAuth nécessaire pour appeler Microsoft Graph. Dans cette étape, vous allez intégrer la [bibliothèque MSAL pour Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) dans l'application.

1. Créez un fichier à la racine du projet nommé `oauth_settings.yml` et ajoutez le contenu suivant.

    :::code language="ini" source="../demo/graph_tutorial/oauth_settings.yml.example":::

1. Remplacez `YOUR_APP_ID_HERE` par l'ID d'application à partir du portail d'inscription des applications et remplacez-le par le `YOUR_APP_SECRET_HERE` mot de passe que vous avez généré.

> [!IMPORTANT]
> Si vous utilisez un contrôle source tel que Git, il est temps d'exclure le fichier **oauth_settings.yml** du contrôle source afin d'éviter toute fuite accidentelle de votre ID d'application et de votre mot de passe.

## <a name="implement-sign-in"></a>Implémentation de la connexion

1. Créez un fichier dans le **répertoire ./tutorial** nommé `auth_helper.py` et ajoutez le code suivant.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="FirstCodeSnippet":::

    Ce fichier conservera toutes vos méthodes d'authentification. La méthode génère une URL d'autorisation et la méthode échange la réponse d'autorisation `get_sign_in_flow` `get_token_from_code` contre un jeton d'accès.

1. Ajoutez les `import` instructions suivantes en haut de **./tutorial/views.py**.

    ```python
    from dateutil import tz, parser
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code
    ```

1. Ajoutez un affichage de sign-in dans **le fichier ./tutorial/views.py.**

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignInViewSnippet":::

1. Ajoutez un affichage de rappel dans **le fichier ./tutorial/views.py.**

    ```python
    def callback(request):
      # Make the token request
      result = get_token_from_code(request)
      # Temporary! Save the response in an error so it's displayed
      request.session['flash_error'] = { 'message': 'Token retrieved', 'debug': format(result) }
      return HttpResponseRedirect(reverse('home'))
    ```

    Prenez en compte les possibilités de ces affichages :

    - L'action génère l'URL de la signature Azure AD, enregistre le flux généré par le client OAuth, puis redirige le navigateur vers la page de signature `signin` Azure AD.

    - `callback`L'action est l'endroit où Azure redirige une fois la signature terminée. Cette action utilise le flux enregistré et la chaîne de requête envoyée par Azure pour demander un jeton d'accès. Il redirige ensuite vers la page d'accueil avec la réponse dans la valeur d'erreur temporaire. Vous l'utiliserez pour vérifier que notre connectez-vous fonctionne avant de passer à autre chose.

1. Ouvrez **./tutorial/urls.py** et remplacez les `path` instructions existantes `signin` par ce qui suit.

    ```python
    path('signin', views.sign_in, name='signin'),
    ```

1. Ajoutez une nouvelle `path` valeur pour `callback` l'affichage.

    ```python
    path('callback', views.callback, name='callback'),
    ```

1. Démarrez le serveur et accédez à `https://localhost:8000` . Cliquez sur le bouton de connexion. vous serez redirigé vers `https://login.microsoftonline.com`. Connectez-vous avec votre compte Microsoft et consentez aux autorisations demandées. Le navigateur redirige vers l'application, affichant la réponse, y compris le jeton d'accès.

### <a name="get-user-details"></a>Obtenir les détails de l’utilisateur

1. Créez un fichier dans le **répertoire ./tutorial** nommé `graph_helper.py` et ajoutez le code suivant.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="FirstCodeSnippet":::

    La méthode effectue une requête GET au point de terminaison Microsoft Graph pour obtenir le profil de l'utilisateur, à l'aide du jeton d'accès que vous `get_user` `/me` avez acquis précédemment.

1. Mettez à `callback` jour la méthode dans **./tutorial/views.py** pour obtenir le profil de l'utilisateur à partir de Microsoft Graph. Ajoutez l’instruction `import` suivante en haut du fichier.

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

    Le nouveau code appelle la `get_user` méthode pour demander le profil de l'utilisateur. Il ajoute l'objet utilisateur à la sortie temporaire pour le test.

1. Ajoutez les nouvelles méthodes suivantes **à ./tutorial/auth_helper.py**.

    :::code language="python" source="../demo/graph_tutorial/tutorial/auth_helper.py" id="SecondCodeSnippet":::

1. Mettez à `callback` jour la fonction dans **./tutorial/views.py** pour stocker l'utilisateur dans la session et rediriger vers la page principale. Remplacez `from tutorial.auth_helper import get_sign_in_flow, get_token_from_code` la ligne par ce qui suit.

    ```python
    from tutorial.auth_helper import get_sign_in_flow, get_token_from_code, store_user, remove_user_and_token, get_token
    ```

1. Remplacez `callback` la méthode par ce qui suit.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CallbackViewSnippet":::

## <a name="implement-sign-out"></a>Implémenter la signature

1. Ajoutez un nouvel `sign_out` affichage **dans ./tutorial/views.py**.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="SignOutViewSnippet":::

1. Ouvrez **./tutorial/urls.py** et remplacez les `path` instructions existantes `signout` par ce qui suit.

    ```python
    path('signout', views.sign_out, name='signout'),
    ```

1. Redémarrez le serveur et traversez le processus de sign-in. Vous devez revenir sur la page d'accueil, mais l'interface utilisateur doit changer pour indiquer que vous êtes bien inscrit.

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. Cliquez sur l'avatar de l'utilisateur dans le coin supérieur droit pour accéder au lien **de** connexion. Le fait de cliquer sur **Se déconnecter** réinitialise la session et vous ramène à la page d’accueil.

    ![Capture d’écran du menu déroulant avec le lien de déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualisation des jetons

À ce stade, votre application dispose d'un jeton d'accès, qui est envoyé dans l'en-tête des `Authorization` appels d'API. Il s'agit du jeton qui permet à l'application d'accéder à Microsoft Graph au nom de l'utilisateur.

Cependant, ce jeton est de courte durée. Le jeton expire une heure après son émission. C’est là que le jeton d’actualisation devient utile. Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans obliger l’utilisateur à se reconnecter.

Étant donné que cet exemple utilise MSAL, vous n'avez pas besoin d'écrire de code spécifique pour actualiser le jeton. La méthode MSAL gère `acquire_token_silent` l'actualisation du jeton si nécessaire.
