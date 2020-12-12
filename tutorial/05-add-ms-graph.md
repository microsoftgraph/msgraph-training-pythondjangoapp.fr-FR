<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application.

## <a name="get-calendar-events-from-outlook"></a>Récupérer les événements de calendrier à partir d’Outlook

1. Commencez par ajouter une méthode à **./tutorial/graph_helper. py** pour extraire un affichage du calendrier entre deux dates. Ajoutez la méthode suivante.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    Que fait ce code ?

    - L’URL qui sera appelée est `/v1.0/me/calendarview`.
        - L' `Prefer: outlook.timezone` en-tête entraîne l’ajustement des heures de début et de fin dans les résultats sur le fuseau horaire de l’utilisateur.
        - Les `startDateTime` `endDateTime` paramètres et définissent le début et la fin de l’affichage.
        - Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que l’affichage utilise réellement.
        - Le `$orderby` paramètre trie les résultats par heure de début.
        - Le `$top` paramètre limite les résultats à 50 événements.

1. Ajoutez le code suivant à **./tutorial/graph_helper. py** pour rechercher un [identificateur de fuseau horaire IANA](https://www.iana.org/time-zones) en fonction du nom du fuseau horaire Windows. Cette opération est nécessaire, car Microsoft Graph peut renvoyer des fuseaux horaires sous la forme de noms de fuseaux horaires Windows et la bibliothèque de **type python DateTime** requiert des identificateurs de fuseau horaire IANA.

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="ZoneMappingsSnippet":::

1. Ajoutez la vue suivante à **./Tutorial/views.py**.

    ```python
    def calendar(request):
      context = initialize_context(request)
      user = context['user']

      # Load the user's time zone
      # Microsoft Graph can return the user's time zone as either
      # a Windows time zone name or an IANA time zone identifier
      # Python datetime requires IANA, so convert Windows to IANA
      time_zone = get_iana_from_windows(user['timeZone'])
      tz_info = tz.gettz(time_zone)

      # Get midnight today in user's time zone
      today = datetime.now(tz_info).replace(
        hour=0,
        minute=0,
        second=0,
        microsecond=0)

      # Based on today, get the start of the week (Sunday)
      if (today.weekday() != 6):
        start = today - timedelta(days=today.isoweekday())
      else:
        start = today

      end = start + timedelta(days=7)

      token = get_token(request)

      events = get_calendar_events(
        token,
        start.isoformat(timespec='seconds'),
        end.isoformat(timespec='seconds'),
        user['timeZone'])

      context['errors'] = [
        { 'message': 'Events', 'debug': format(events)}
      ]

      return render(request, 'tutorial/home.html', context)
    ```

1. Ouvrez **./Tutorial/URLs.py** et remplacez les `path` instructions existantes par, `calendar` comme suit.

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation. Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.

## <a name="display-the-results"></a>Afficher les résultats

À présent, vous pouvez ajouter un modèle pour afficher les résultats de manière plus conviviale.

1. Créez un fichier dans le répertoire **./Tutorial/templates/Tutorial** nommé `calendar.html` et ajoutez le code suivant.

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    Cela permet de parcourir une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.

1. Remplacez la `calendar` vue dans **./Tutorial/views.py** par le code suivant.

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. Actualisez la page et l’application doit maintenant afficher un tableau d’événements.

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
