<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application. Pour cette application, vous allez utiliser la bibliothèque [requêtes-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) pour passer des appels à Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Récupérer les événements de calendrier à partir d’Outlook

1. Commencez par ajouter une méthode à **./tutorial/graph_helper. py** pour extraire les événements de calendrier. Ajoutez la méthode suivante.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    Que fait ce code ?

    - L’URL qui sera appelée est `/v1.0/me/events`.
    - Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que l’affichage utilise réellement.
    - Le `$orderby` paramètre trie les résultats en fonction de la date et de l’heure de leur création, avec l’élément le plus récent en premier.

1. Dans **./Tutorial/views.py**, modifiez la `from tutorial.graph_helper import get_user` ligne comme suit.

    ```python
    from tutorial.graph_helper import get_user, get_calendar_events
    ```

1. Ajoutez la vue suivante à **./Tutorial/views.py**.

    ```python
    def calendar(request):
      context = initialize_context(request)

      token = get_token(request)

      events = get_calendar_events(token)

      context['errors'] = [
        { 'message': 'Events', 'debug': format(events)}
      ]

      return render(request, 'tutorial/home.html', context)
    ```

1. Ouvrez **./Tutorial/URLs.py** et remplacez les instructions `path` existantes par `calendar` , comme suit.

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation. Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.

## <a name="display-the-results"></a>Afficher les résultats

À présent, vous pouvez ajouter un modèle pour afficher les résultats de manière plus conviviale.

1. Créez un fichier dans le répertoire **./Tutorial/templates/Tutorial** nommé `calendar.html` et ajoutez le code suivant.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    Cela permet de parcourir une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.

1. Ajoutez l’instruction `import` suivante en haut du fichier **./Tutorials/views.py** .

    ```python
    import dateutil.parser
    ```

1. Remplacez la `calendar` vue dans **./Tutorial/views.py** par le code suivant.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. Actualisez la page et l’application doit maintenant afficher un tableau d’événements.

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
