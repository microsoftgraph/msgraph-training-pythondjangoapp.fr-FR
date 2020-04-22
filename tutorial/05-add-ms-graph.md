<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5e4dc-101">Dans cet exercice, vous allez incorporer Microsoft Graph dans l’application.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="5e4dc-102">Pour cette application, vous allez utiliser la bibliothèque [requêtes-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) pour passer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-102">For this application, you will use the [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="5e4dc-103">Récupérer les événements de calendrier à partir d’Outlook</span><span class="sxs-lookup"><span data-stu-id="5e4dc-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="5e4dc-104">Commencez par ajouter une méthode à **./tutorial/graph_helper. py** pour extraire les événements de calendrier.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-104">Start by adding a method to **./tutorial/graph_helper.py** to fetch the calendar events.</span></span> <span data-ttu-id="5e4dc-105">Ajoutez la méthode suivante.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-105">Add the following method.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="GetCalendarSnippet":::

    <span data-ttu-id="5e4dc-106">Que fait ce code ?</span><span class="sxs-lookup"><span data-stu-id="5e4dc-106">Consider what this code is doing.</span></span>

    - <span data-ttu-id="5e4dc-107">L’URL qui sera appelée est `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-107">The URL that will be called is `/v1.0/me/events`.</span></span>
    - <span data-ttu-id="5e4dc-108">Le `$select` paramètre limite les champs renvoyés pour chaque événement à ceux que l’affichage utilise réellement.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-108">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="5e4dc-109">Le `$orderby` paramètre trie les résultats en fonction de la date et de l’heure de leur création, avec l’élément le plus récent en premier.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-109">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="5e4dc-110">Dans **./Tutorial/views.py**, modifiez la `from tutorial.graph_helper import get_user` ligne comme suit.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-110">In **./tutorial/views.py**, change the `from tutorial.graph_helper import get_user` line to the following.</span></span>

    ```python
    from tutorial.graph_helper import get_user, get_calendar_events
    ```

1. <span data-ttu-id="5e4dc-111">Ajoutez la vue suivante à **./Tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-111">Add the following view to **./tutorial/views.py**.</span></span>

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

1. <span data-ttu-id="5e4dc-112">Ouvrez **./Tutorial/URLs.py** et remplacez les instructions `path` existantes par `calendar` , comme suit.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-112">Open **./tutorial/urls.py** and replace the existing `path` statements for `calendar` with the following.</span></span>

    ```python
    path('calendar', views.calendar, name='calendar'),
    ```

1. <span data-ttu-id="5e4dc-113">Connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-113">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="5e4dc-114">Si tout fonctionne, vous devriez voir une image mémoire JSON des événements dans le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-114">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="5e4dc-115">Afficher les résultats</span><span class="sxs-lookup"><span data-stu-id="5e4dc-115">Display the results</span></span>

<span data-ttu-id="5e4dc-116">À présent, vous pouvez ajouter un modèle pour afficher les résultats de manière plus conviviale.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-116">Now you can add a template to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="5e4dc-117">Créez un fichier dans le répertoire **./Tutorial/templates/Tutorial** nommé `calendar.html` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-117">Create a new file in the **./tutorial/templates/tutorial** directory named `calendar.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/calendar.html" id="CalendarSnippet":::

    <span data-ttu-id="5e4dc-118">Cela permet de parcourir une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-118">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="5e4dc-119">Ajoutez l’instruction `import` suivante en haut du fichier **./Tutorials/views.py** .</span><span class="sxs-lookup"><span data-stu-id="5e4dc-119">Add the following `import` statement to the top of the **./tutorials/views.py** file.</span></span>

    ```python
    import dateutil.parser
    ```

1. <span data-ttu-id="5e4dc-120">Remplacez la `calendar` vue dans **./Tutorial/views.py** par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-120">Replace the `calendar` view in **./tutorial/views.py** with the following code.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="CalendarViewSnippet":::

1. <span data-ttu-id="5e4dc-121">Actualisez la page et l’application doit maintenant afficher un tableau d’événements.</span><span class="sxs-lookup"><span data-stu-id="5e4dc-121">Refresh the page and the app should now render a table of events.</span></span>

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
