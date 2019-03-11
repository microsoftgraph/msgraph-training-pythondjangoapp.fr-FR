<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="8c604-101">Dans cet exercice, vous allez incorporer Microsoft Graph dans l'application.</span><span class="sxs-lookup"><span data-stu-id="8c604-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="8c604-102">Pour cette application, vous allez utiliser la bibliothèque [requêtes-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) pour passer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="8c604-102">For this application, you will use the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="8c604-103">Obtenir des événements de calendrier à partir d'Outlook</span><span class="sxs-lookup"><span data-stu-id="8c604-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="8c604-104">Commencez par ajouter une méthode pour `./tutorial/graph_helper.py` extraire les événements de calendrier.</span><span class="sxs-lookup"><span data-stu-id="8c604-104">Start by adding a method to `./tutorial/graph_helper.py` to fetch the calendar events.</span></span> <span data-ttu-id="8c604-105">Ajoutez la méthode suivante.</span><span class="sxs-lookup"><span data-stu-id="8c604-105">Add the following method.</span></span>

```python
def get_calendar_events(token):
  graph_client = OAuth2Session(token=token)

  # Configure query parameters to
  # modify the results
  query_params = {
    '$select': 'subject,organizer,start,end',
    '$orderby': 'createdDateTime DESC'
  }

  # Send GET to /me/events
  events = graph_client.get('{0}/me/events'.format(graph_url), params=query_params)
  # Return the JSON result
  return events.json()
```

<span data-ttu-id="8c604-106">Examinez ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="8c604-106">Consider what this code is doing.</span></span>

- <span data-ttu-id="8c604-107">L'URL qui sera appelée est `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="8c604-107">The URL that will be called is `/v1.0/me/events`.</span></span>
- <span data-ttu-id="8c604-108">Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que l'affichage utilise réellement.</span><span class="sxs-lookup"><span data-stu-id="8c604-108">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
- <span data-ttu-id="8c604-109">Le `$orderby` paramètre trie les résultats en fonction de la date et de l'heure de leur création, avec l'élément le plus récent en premier.</span><span class="sxs-lookup"><span data-stu-id="8c604-109">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

<span data-ttu-id="8c604-110">Créez maintenant un affichage Calendrier.</span><span class="sxs-lookup"><span data-stu-id="8c604-110">Now create a calendar view.</span></span> <span data-ttu-id="8c604-111">Commencez par modifier `from tutorial.graph_helper import get_user` la ligne comme suit.</span><span class="sxs-lookup"><span data-stu-id="8c604-111">First change the `from tutorial.graph_helper import get_user` line to the following.</span></span>

```python
from tutorial.graph_helper import get_user, get_calendar_events
```

<span data-ttu-id="8c604-112">Ensuite, ajoutez la vue suivante à `./tutorial/views.py`.</span><span class="sxs-lookup"><span data-stu-id="8c604-112">Then, add the following view to `./tutorial/views.py`.</span></span>

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

<span data-ttu-id="8c604-113">Mettre `./tutorial/urls.py` à jour pour ajouter cette nouvelle vue.</span><span class="sxs-lookup"><span data-stu-id="8c604-113">Update `./tutorial/urls.py` to add this new view.</span></span>

```python
path('calendar', views.calendar, name='calendar'),
```

<span data-ttu-id="8c604-114">Enfin, mettez à \*\*\*\* jour le lien `./tutorial/templates/tutorial/layout.html` de calendrier dans pour établir un lien vers cet affichage.</span><span class="sxs-lookup"><span data-stu-id="8c604-114">Finally, update  the **Calendar** link in `./tutorial/templates/tutorial/layout.html` to link to this view.</span></span> <span data-ttu-id="8c604-115">Remplacez la `<a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="#">Calendar</a>` ligne par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="8c604-115">Replace the `<a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="#">Calendar</a>` line with the following.</span></span>

```html
<a class="nav-link{% if request.resolver_match.view_name == 'calendar' %} active{% endif %}" href="{% url 'calendar' %}">Calendar</a>
```

<span data-ttu-id="8c604-116">À présent, vous pouvez le tester.</span><span class="sxs-lookup"><span data-stu-id="8c604-116">Now you can test this.</span></span> <span data-ttu-id="8c604-117">Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="8c604-117">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="8c604-118">Si tout fonctionne, vous devez voir un vidage JSON des événements sur le calendrier de l'utilisateur.</span><span class="sxs-lookup"><span data-stu-id="8c604-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="8c604-119">Afficher les résultats</span><span class="sxs-lookup"><span data-stu-id="8c604-119">Display the results</span></span>

<span data-ttu-id="8c604-120">À présent, vous pouvez ajouter un modèle pour afficher les résultats de manière plus conviviale.</span><span class="sxs-lookup"><span data-stu-id="8c604-120">Now you can add a template to display the results in a more user-friendly manner.</span></span> <span data-ttu-id="8c604-121">Créez un fichier dans le `./tutorial/templates/tutorial` répertoire nommé `calendar.html` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="8c604-121">Create a new file in the `./tutorial/templates/tutorial` directory named `calendar.html` and add the following code.</span></span>

```html
{% extends "tutorial/layout.html" %}
{% block content %}
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    {% if events %}
      {% for event in events %}
        <tr>
          <td>{{ event.organizer.emailAddress.name }}</td>
          <td>{{ event.subject }}</td>
          <td>{{ event.start.dateTime|date:'SHORT_DATETIME_FORMAT' }}</td>
          <td>{{ event.end.dateTime|date:'SHORT_DATETIME_FORMAT' }}</td>
        </tr>
      {% endfor %}
    {% endif %}
  </tbody>
</table>
{% endblock %}
```

<span data-ttu-id="8c604-122">Cela permet d'exécuter une boucle dans une collection d'événements et d'ajouter une ligne de tableau pour chacun d'eux.</span><span class="sxs-lookup"><span data-stu-id="8c604-122">That will loop through a collection of events and add a table row for each one.</span></span> <span data-ttu-id="8c604-123">Ajoutez l'instruction `import` suivante en haut du `./tutorials/views.py` fichier.</span><span class="sxs-lookup"><span data-stu-id="8c604-123">Add the following `import` statement to the top of the `./tutorials/views.py` file.</span></span>

```python
import dateutil.parser
```

<span data-ttu-id="8c604-124">Remplacez l' `calendar` affichage `./tutorial/views.py` par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="8c604-124">Replace the `calendar` view in `./tutorial/views.py` with the following code.</span></span>

```python
def calendar(request):
  context = initialize_context(request)

  token = get_token(request)

  events = get_calendar_events(token)

  if events:
    # Convert the ISO 8601 date times to a datetime object
    # This allows the Django template to format the value nicely
    for event in events['value']:
      event['start']['dateTime'] = dateutil.parser.parse(event['start']['dateTime'])
      event['end']['dateTime'] = dateutil.parser.parse(event['end']['dateTime'])

    context['events'] = events['value']

  return render(request, 'tutorial/calendar.html', context)
```

<span data-ttu-id="8c604-125">Actualisez la page et l'application doit maintenant afficher un tableau d'événements.</span><span class="sxs-lookup"><span data-stu-id="8c604-125">Refresh the page and the app should now render a table of events.</span></span>

![Capture d'écran du tableau des événements](./images/add-msgraph-01.png)