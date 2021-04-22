<!-- markdownlint-disable MD002 MD041 -->

Ce didacticiel vous apprend à créer une application web Python Django qui utilise l'API Microsoft Graph pour récupérer les informations de calendrier d'un utilisateur.

> [!TIP]
> Si vous préférez simplement télécharger le didacticiel terminé, vous pouvez le télécharger de deux manières.
>
> - Téléchargez le [démarrage rapide de Python](https://developer.microsoft.com/graph/quick-start?platform=option-Python) pour obtenir du code de travail en quelques minutes.
> - Téléchargez ou clonez [le référentiel GitHub.](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp)

## <a name="prerequisites"></a>Configuration requise

Avant de commencer ce didacticiel, [python](https://www.python.org/) (avec [pip)](https://pypi.org/project/pip/)doit être installé sur votre ordinateur de développement. Si vous n'avez pas Python, consultez le lien précédent pour obtenir les options de téléchargement.

Vous devez également avoir un compte Microsoft personnel avec une boîte aux lettres sur Outlook.com ou un compte scolaire ou scolaire Microsoft. Si vous n'avez pas de compte Microsoft, deux options s'offrent à vous pour obtenir un compte gratuit :

- Vous pouvez [vous inscrire à un nouveau compte Microsoft personnel.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Vous pouvez vous inscrire au programme pour les développeurs [Office 365](https://developer.microsoft.com/office/dev-program) pour obtenir un abonnement Office 365 gratuit.

> [!NOTE]
> Ce didacticiel a été écrit avec Python version 3.9.2 et Django version 3.2. Les étapes de ce guide peuvent fonctionner avec d'autres versions, mais elles n'ont pas été testées.

## <a name="feedback"></a>Commentaires

Veuillez nous faire part de vos commentaires sur ce didacticiel dans [le référentiel GitHub.](https://github.com/microsoftgraph/msgraph-training-pythondjangoapp)
