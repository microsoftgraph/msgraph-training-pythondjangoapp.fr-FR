<!-- markdownlint-disable MD002 MD041 -->

Dans cette section, vous allez ajouter la possibilité de créer des événements dans le calendrier de l’utilisateur.

1. Ajoutez la méthode suivante à **./tutorial/graph_helper. py** pour créer un événement.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="CreateEventSnippet":::

## <a name="create-a-new-event-form"></a>Créer un formulaire d’événement

1. Créez un fichier dans le répertoire **./Tutorial/templates/Tutorial** nommé `newevent.html` et ajoutez le code suivant.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/newevent.html" id="NewEventSnippet":::

1. Ajoutez la vue suivante à **./Tutorial/views.py**.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="NewEventViewSnippet":::

1. Ouvrez **./Tutorial/URLs.py** et ajoutez `path` des instructions pour la `newevent` vue.

    ```python
    path('calendar/new', views.newevent, name='newevent'),
    ```

1. Enregistrez vos modifications et actualisez l’application. Sur la page **calendrier** , cliquez sur le bouton **nouvel événement** . Remplissez le formulaire et sélectionnez **créer** pour créer l’événement.

    ![Capture d’écran du nouveau formulaire d’événement](images/create-event-01.png)
