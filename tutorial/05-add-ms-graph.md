<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="01d4a-101">Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application.</span><span class="sxs-lookup"><span data-stu-id="01d4a-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="01d4a-102">Récupérer les événements de calendrier à partir d’Outlook</span><span class="sxs-lookup"><span data-stu-id="01d4a-102">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="01d4a-103">Commencez par ajouter une méthode à **./tutorial/graph_helper. py** pour extraire un affichage du calendrier entre deux dates.</span><span class="sxs-lookup"><span data-stu-id="01d4a-103">Start by adding a method to **./tutorial/graph_helper.py** to fetch a view of the calendar between two dates.</span></span> <span data-ttu-id="01d4a-104">Ajoutez la méthode suivante.</span><span class="sxs-lookup"><span data-stu-id="01d4a-104">Add the following method.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    <span data-ttu-id="01d4a-105">Que fait ce code ?</span><span class="sxs-lookup"><span data-stu-id="01d4a-105">Consider what this code is doing.</span></span>

    - <span data-ttu-id="01d4a-106">L’URL qui sera appelée est `/v1.0/me/calendarview`.</span><span class="sxs-lookup"><span data-stu-id="01d4a-106">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="01d4a-107">L' `Prefer: outlook.timezone` en-tête entraîne l’ajustement des heures de début et de fin dans les résultats sur le fuseau horaire de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="01d4a-107">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="01d4a-108">Les `startDateTime` `endDateTime` paramètres et définissent le début et la fin de l’affichage.</span><span class="sxs-lookup"><span data-stu-id="01d4a-108">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="01d4a-109">Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que l’affichage utilise réellement.</span><span class="sxs-lookup"><span data-stu-id="01d4a-109">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="01d4a-110">Le `$orderby` paramètre trie les résultats par heure de début.</span><span class="sxs-lookup"><span data-stu-id="01d4a-110">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="01d4a-111">Le `$top` paramètre limite les résultats à 50 événements.</span><span class="sxs-lookup"><span data-stu-id="01d4a-111">The `$top` parameter limits the results to 50 events.</span></span>

1. <span data-ttu-id="01d4a-112">Ajoutez le code suivant à **./tutorial/graph_helper. py** pour rechercher un [identificateur de fuseau horaire IANA](https://www.iana.org/time-zones) en fonction du nom du fuseau horaire Windows.</span><span class="sxs-lookup"><span data-stu-id="01d4a-112">Add the following code to **./tutorial/graph_helper.py** to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="01d4a-113">Cette opération est nécessaire, car Microsoft Graph peut renvoyer des fuseaux horaires sous la forme de noms de fuseaux horaires Windows et la bibliothèque de **type python DateTime** requiert des identificateurs de fuseau horaire IANA.</span><span class="sxs-lookup"><span data-stu-id="01d4a-113">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Python **datetime** library requires IANA time zone identifiers.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="ZoneMappingsSnippet":::

1. <span data-ttu-id="01d4a-114">Ajoutez la vue suivante à **./Tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="01d4a-114">Add the following view to **./tutorial/views.py**.</span></span>

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

1. <span data-ttu-id="01d4a-115">Ouvrez **./Tutorial/URLs.py** et remplacez les `path` instructions existantes par, `calendar` comme suit.</span><span class="sxs-lookup"><span data-stu-id="01d4a-115">Open **./tutorial/urls.py** and replace the existing `path` statements for `calendar` with the following.</span></span>

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. <span data-ttu-id="01d4a-116">Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="01d4a-116">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="01d4a-117">Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="01d4a-117">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="01d4a-118">Afficher les résultats</span><span class="sxs-lookup"><span data-stu-id="01d4a-118">Display the results</span></span>

<span data-ttu-id="01d4a-119">À présent, vous pouvez ajouter un modèle pour afficher les résultats de manière plus conviviale.</span><span class="sxs-lookup"><span data-stu-id="01d4a-119">Now you can add a template to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="01d4a-120">Créez un fichier dans le répertoire **./Tutorial/templates/Tutorial** nommé `calendar.html` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="01d4a-120">Create a new file in the **./tutorial/templates/tutorial** directory named `calendar.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    <span data-ttu-id="01d4a-121">Cela permet de parcourir une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.</span><span class="sxs-lookup"><span data-stu-id="01d4a-121">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="01d4a-122">Remplacez la `calendar` vue dans **./Tutorial/views.py** par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="01d4a-122">Replace the `calendar` view in **./tutorial/views.py** with the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. <span data-ttu-id="01d4a-123">Actualisez la page et l’application doit maintenant afficher un tableau d’événements.</span><span class="sxs-lookup"><span data-stu-id="01d4a-123">Refresh the page and the app should now render a table of events.</span></span>

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
