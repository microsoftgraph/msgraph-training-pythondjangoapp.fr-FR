<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="2b540-101">Dans cette section, vous allez ajouter la possibilité de créer des événements dans le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="2b540-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="2b540-102">Ajoutez la méthode suivante à **./tutorial/graph_helper. py** pour créer un événement.</span><span class="sxs-lookup"><span data-stu-id="2b540-102">Add the following method to **./tutorial/graph_helper.py** to create a new event.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/graph_helper.py" id="CreateEventSnippet":::

## <a name="create-a-new-event-form"></a><span data-ttu-id="2b540-103">Créer un formulaire d’événement</span><span class="sxs-lookup"><span data-stu-id="2b540-103">Create a new event form</span></span>

1. <span data-ttu-id="2b540-104">Créez un fichier dans le répertoire **./Tutorial/templates/Tutorial** nommé `newevent.html` et ajoutez le code suivant.</span><span class="sxs-lookup"><span data-stu-id="2b540-104">Create a new file in the **./tutorial/templates/tutorial** directory named `newevent.html` and add the following code.</span></span>

    :::code language="html" source="../demo/graph_tutorial/tutorial/templates/tutorial/newevent.html" id="NewEventSnippet":::

1. <span data-ttu-id="2b540-105">Ajoutez la vue suivante à **./Tutorial/views.py**.</span><span class="sxs-lookup"><span data-stu-id="2b540-105">Add the following view to **./tutorial/views.py**.</span></span>

    :::code language="python" source="../demo/graph_tutorial/tutorial/views.py" id="NewEventViewSnippet":::

1. <span data-ttu-id="2b540-106">Ouvrez **./Tutorial/URLs.py** et ajoutez `path` des instructions pour la `newevent` vue.</span><span class="sxs-lookup"><span data-stu-id="2b540-106">Open **./tutorial/urls.py** and add a `path` statements for the `newevent` view.</span></span>

    ```python
    path('calendar/new', views.newevent, name='newevent'),
    ```

1. <span data-ttu-id="2b540-107">Enregistrez vos modifications et actualisez l’application.</span><span class="sxs-lookup"><span data-stu-id="2b540-107">Save your changes and refresh the app.</span></span> <span data-ttu-id="2b540-108">Sur la page **calendrier** , cliquez sur le bouton **nouvel événement** .</span><span class="sxs-lookup"><span data-stu-id="2b540-108">On the **Calendar** page, select the **New event** button.</span></span> <span data-ttu-id="2b540-109">Remplissez le formulaire et sélectionnez **créer** pour créer l’événement.</span><span class="sxs-lookup"><span data-stu-id="2b540-109">Fill in the form and select **Create** to create the event.</span></span>

    ![Capture d’écran du nouveau formulaire d’événement](images/create-event-01.png)
