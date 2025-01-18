---
lab:
  title: "Labo\_: Déployer et gérer une application conteneur à l’aide d’Azure Container Apps"
  type: Answer Key
  module: 'Module 6: Deploy and manage a container app using Azure Container Apps'
---

# Labo : Déployer et gérer une application conteneur à l’aide d’Azure Container Apps
# Labo étudiant - Corrigé

## Instructions

Dans ce labo, vous allez déployer et gérer une application en utilisant Azure Container Apps. Pour implémenter la solution, commencez par configurer un environnement de développement qui utilise un mélange d’outils locaux et de ressources Azure. Une fois que l’environnement est préparé, vous utilisez Azure Container Registry, Azure Container Apps et Azure Pipelines pour déployer et gérer votre application.  

À la fin de ce labo, vous êtes capable de :

1. Configurer une connexion sécurisée entre Azure Container Registry et Azure Container Apps.
1. créer et configurer une application conteneur dans Azure Container Apps.
1. configurer l'intégration continue à l'aide d'Azure Pipelines.
1. mettre à l'échelle une application déployée dans Azure Container Apps.
1. gérer les révisions dans Azure Container Apps.

### Exercice 1 : Configurer des ressources Azure

Dans cet exercice, vous allez configurer des ressources Azure qui prennent en charge votre solution Azure Container Apps.

La réalisation des tâches suivantes prend environ 10 à 15 minutes :

- Examiner les paramètres du groupe de ressources.
- Configurer un réseau virtuel Azure avec des sous-réseaux.
- Configurer une ressource Azure Service Bus.
- Configurer une ressource Azure Container Registry.

#### Tâche 1 : Examiner les paramètres du groupe de ressources

Effectuez les étapes suivantes pour examiner le paramètre de localisation du groupe de ressources utilisé dans ce labo.

1. Dans l’environnement du labo, ouvrez une fenêtre de navigateur et accédez au portail Azure : `https://portal.azure.com/`

    Demandez de l’aide à votre formateur pour vous connecter au portail Azure avec un abonnement/un compte approprié pour ce labo, si besoin.

1. Dans votre page d’accueil du portail Azure, sous **Naviguer**, sélectionnez **Groupes de ressources**.

1. Dans la page Groupes de ressources, sélectionnez **RG1**.

    Si le groupe de ressources RG1 n’a pas été créé, créez-le maintenant.

1. Notez le paramètre de **localisation** affecté au groupe de ressources **RG1**.

    Vous utiliserez la même localisation/région lorsque vous créerez d’autres ressources Azure au cours de ce labo.

1. Fermez la page **RG1**, puis la page **Groupes de ressources**.

#### Tâche 2 : Configurer un réseau virtuel et des sous-réseaux

Suivez les étapes ci-dessous pour configurer un réseau virtuel et des sous-réseaux.

1. Dans la barre de recherche supérieure du portail Azure, dans la zone de texte Rechercher, entrez le **réseau virtuel**

1. Dans les résultats de recherche, sélectionnez **Réseaux virtuels**.

1. Sélectionner **Créer un réseau virtuel**.

1. Sous l'onglet De bases, configurez votre réseau virtuel comme suit :

    - Abonnement : vérifiez que l'abonnement Azure que vous utilisez pour ce projet guidé est sélectionné.
    - Nom du groupe de ressources : Sélectionner **RG1**
    - Nom du réseau virtuel : Entrer **VNET1**
    - Région : Vérifiez que la région spécifiée correspond au paramètre de localisation de votre groupe de ressources.

1. Sélectionnez l'onglet **Adresses IP**.

1. Sous l'onglet Adresses IP, sous **Sous-réseaux**, sélectionnez **par défaut**.

1. À la page Modifier le sous-réseau, configurez le sous-réseau comme suit :

    - Nom : Entrer **`PESubnet`**
    - Adresse de début : Vérifiez que **10.0.0.0** est spécifié.
    - Taille du sous-réseau : Vérifiez que **/24 (256 adresses)** est spécifié.

    Ce premier sous-réseau sera utilisé pour un point de terminaison privé (Azure Container Registry).

1. Sélectionnez **Enregistrer**.

1. Sous l'onglet Adresses IP, sélectionnez **+ Ajouter un sous-réseau**.

1. À la page Ajouter un sous-réseau, configurez le sous-réseau comme suit :

    - Nom : Entrer **`ACASubnet`**
    - Adresse de début : Vérifiez que **10.0.4.0** est spécifié.
    - Taille du sous-réseau : Vérifiez que **/23 (512 adresses)** est spécifiée.

    Le sous-réseau pour Azure Container Apps nécessite un espace d’adressage supérieur à 256.

1. Sélectionnez **Ajouter**.

1. Sélectionnez **Revoir + créer**.

1. Une fois la validation réussie, sélectionnez **Créer**.

1. Attendez que le déploiement se termine, puis fermez la page VNET1.

#### Tâche 3 : Configurer Service Bus

Suivez les étapes ci-dessous pour configurer une instance Service Bus.

1. Dans la barre de recherche supérieure du portail Azure, dans la zone de texte Rechercher, entrer **Service Bus**

1. Dans les résultats de recherche, sélectionnez **Service Bus**.

1. Sélectionnez **Créer un espace de noms Service Bus**.

1. Sous l'onglet De bases, configurez votre espace de noms Service Bus comme suit :

    - Abonnement : vérifiez que l'abonnement Azure que vous utilisez pour ce projet guidé est sélectionné.
    - Nom du groupe de ressources : Sélectionner **RG1**
    - Nom de l’espace de noms : Entrez **sb-az2003-** suivi de vos initiales et de la date. Par exemple : **sb-az2003-cah12oct**.
    - Emplacement : Vérifiez que la localisation spécifiée correspond au paramètre de localisation de votre groupe de ressources.
    - Niveau tarifaire : Sélectionnez **De base**.

1. Sélectionnez **Revoir + créer**.

1. Une fois le message de validation réussi affiché, sélectionnez **Créer**.

1. Attendez que le déploiement se termine, puis fermez la page Espace de noms Service Bus.

#### Tâche 4 : Configurer Azure Container Registry

Suivez les étapes ci-dessous pour configurer une instance Container Registry.

1. Dans la barre de recherche supérieure du portail Azure, dans la zone de texte Rechercher, entrer le **registre de conteneurs**

1. Dans les résultats de recherche, sélectionnez **Registres de conteneurs**.

1. À la page Registres de conteneurs, sélectionnez **Créer un registre de conteneurs** ou **+ Créer**.

1. Sous l'onglet De base de la page Créer un registre de conteneurs, spécifiez les informations suivantes :

    > [!NOTE]
    > le nom du registre doit être unique. En outre, le niveau Premium est requis pour une liaison privée avec des points de terminaison privés.

    - Abonnement : vérifiez que l'abonnement Azure que vous utilisez pour ce projet guidé est sélectionné.
    - Groupe de ressources : Sélectionnez **RG1**.
    - Nom du registre : Entrez **acraz2003** suivi de vos initiales et de la date. Par exemple : **acraz2003cah12oct**
    - Emplacement : Vérifiez que la localisation spécifiée correspond au paramètre de localisation de votre groupe de ressources.
    - Référence (SKU) : Sélectionnez **Premium**.

1. Sélectionnez **Revoir + créer**.

1. Une fois que le message de validation réussie apparaît, sélectionnez **Créer**.

1. Une fois le déploiement terminé, ouvrez votre ressource de registre de conteneurs.

1. Dans le menu de gauche de la page Registre de conteneurs, sous **Paramètres**, sélectionnez **Réseau**.

1. À la page Mise en réseau, sous l'onglet Accès public, vérifiez que **tous les réseaux** sont sélectionnés.

1. Dans le menu de gauche, sous **Paramètres**, sélectionnez **Propriétés**.

1. À la page Propriétés, sélectionnez **Utilisateur administrateur**, puis **Enregistrer**.

1. Fermez la page Registre de conteneurs.

1. Fermez votre fenêtre (de navigateur) du portail Azure.

### Exercice 2 : Configurer les outils de développement dans l’environnement hôte

Dans cet exercice, vous vous assurez que les outils de script et de développement sont configurés correctement sur la machine virtuelle.

La réalisation des tâches suivantes prend environ 20 à 25 minutes :

- Configurer les extensions Azure CLI.
- Installez Docker Desktop.
- Installer le SDK .NET 8.
- Mettre à jour Visual Studio Code et configurer les extensions.

#### Tâche 1 : Désinstaller Visual Studio Code

Suivez ces étapes pour désinstaller Visual Studio Code.

1. Ouvrez le menu Démarrer de Windows.

1. Dans le menu Démarrer, sélectionnez **Paramètres**.

1. Dans le menu de gauche, sélectionnez **Applications**, puis **Applications installées**.

1. Recherchez **Microsoft Visual Studio Code**.

1. À droite de Microsoft Visual Studio Code, sélectionnez les points de suspension (...), puis sélectionnez **Désinstaller**.

1. Dans la fenêtre contextuelle, sélectionnez **Désinstaller**.

1. À l’invite, sélectionnez **Oui**, puis **OK**.

1. Fermer les paramètres.

#### Tâche 2 : Configurer les extensions Azure CLI

Suivez ces étapes pour configurer Azure CLI.

1. Ouvrez une ligne de commande ou une application de terminal, comme l'invite de commandes Windows.

1. Connectez-vous à Azure à l'aide de la commande `az login`.

    Une fenêtre de navigateur s’ouvre pour vous permettre de sélectionner le compte Azure.

1. Suivez les invites pour terminer le processus d'authentification.

1. Fermez la fenêtre du navigateur.

1. Dans l’application de ligne de commande, pour installer l’extension Azure Container Apps, entrez la commande suivante : `az extension add --name containerapp --upgrade`

1. Fermez l’application de ligne de commande.

#### Tâche 3 : Installer Docker Desktop

Suivez les étapes ci-dessous pour installer le bureau Docker.

1. Ouvrez une fenêtre de navigateur, puis accédez à la page d'installation de bureau Docker : `https://docs.docker.com/desktop/install/windows-install/`

1. Sélectionnez **Docker Desktop pour Windows** et attendez que le fichier du programme d’installation soit téléchargé.

1. Ouvrez le fichier du programme d’installation une fois qu’il est téléchargé, puis suivez les instructions en ligne pour installer Docker Desktop.

    Le processus d’installation prend environ 5 minutes.

1. Une fois l’installation terminée, sélectionnez **Fermer et redémarrer**.

1. Lorsque la machine virtuelle mise à jour redémarre, attendez que la fenêtre **Docker Subscription Service Agreement** s’affiche.

1. Dans la page Docker Subscription Service Agreement, sélectionnez **Accept**.

1. Dans la page Finish setting up Docker Desktop, sélectionnez **Finish**.

1. Dans la page User Account Control, sélectionnez **Yes**.

1. Dans la page Welcome to Docker Desktop, sélectionnez **Continue without signing in**.

1. Dans la page Tell us about the work you do, sélectionnez **Skip**.

1. Attendez que le processus de démarrage de Docker Engine se termine, puis réduisez l’application Docker Desktop.

    Ne fermez pas Docker Desktop, réduisez simplement l’application en cours d’exécution.

#### Tâche 4 : Installer le SDK .NET 8

Suivez ces étapes pour installer le SDK .NET 8.

1. Ouvrez une fenêtre de navigateur web, puis accédez à la page de téléchargement du SDK .NET 8 : `https://dotnet.microsoft.com/download`

1. Sélectionnez **.NET SDK x64**

1. Une fois le téléchargement terminé, ouvrez le fichier d’installation et suivez les instructions en ligne pour installer le SDK .NET 8.

1. Fermez la fenêtre du navigateur.

#### Tâche 5 : Configurer Visual Studio Code avec les extensions C#, Docker et Azure App Service

Suivez ces étapes pour configurer Visual Studio Code avec des extensions.

1. Ouvrez une fenêtre de navigateur web, puis accédez à la page de téléchargement de Visual Studio Code : `https://code.visualstudio.com/download`

1. Sélectionnez **Windows**, puis attendez que le fichier d’installation soit téléchargé.

1. Ouvrez le fichier du programme d’installation de Visual Studio Code.

1. Acceptez le contrat de licence, puis continuez à accepter les paramètres par défaut jusqu’à la fin de l’installation.

1. Ouvrez Visual Studio Code.

1. Dans la **barre Activité**, sélectionnez **Extensions**.

    La barre d’activité est le menu vertical à gauche de l’interface utilisateur de Visual Studio Code.

1. Dans la zone **Rechercher des extensions dans la Place de marché**, entrez **C#**.

    La saisie de « C# » filtre la liste des extensions pour afficher uniquement les extensions qui ont quelque chose à voir avec le développement en C#.

1. Dans la liste filtrée des extensions disponibles, sélectionnez l’extension intitulée « **Kit de développement C#** – Extension C# officielle de Microsoft » publiée par Microsoft.

1. Pour installer l’extension, sélectionnez **Installer**.

1. Attendez que l’installation se termine.

    L’installation du C# Dev Kit prend environ 1 minute.

1. Dans la vue **EXTENSIONS**, remplacez **C#** par **Docker**.

1. Dans la liste filtrée des extensions disponibles, sélectionnez l’extension intitulée **Docker** qui est publiée par Microsoft.

1. Pour installer l’extension, sélectionnez **Installer**.

1. Dans la vue **EXTENSIONS**, remplacez **docker** par **Azure App Service**.

1. Dans la liste filtrée des extensions disponibles, sélectionnez l'extension intitulée **Azure App Service** qui est publiée par Microsoft.

1. Pour installer l’extension, sélectionnez **Installer**.

1. Fermez Visual Studio Code.

### Exercice 3 : Créer et configurer des ressources de déploiement d’applications

Dans cet exercice, vous configurez un projet Azure DevOps et Azure Pipeline, créez et envoyez (push) une image Docker à votre registre de conteneurs, puis déployez un agent Windows autohébergé.

La réalisation des tâches suivantes prend environ 40 minutes :

- Configurer un projet Azure DevOps et initialiser votre dépôt de code.
- Créer une application .NET et la synchroniser avec votre dépôt Azure DevOps.
- Créer une image Docker et envoyer (push) l’image à votre instance Azure Container Registry.
- Créer un pipeline Azure appelé Pipeline1.
- Déployez un agent Windows auto-hébergé.

#### Tâche 1 : Configurer un projet Azure DevOps et initialiser le dépôt de code

Suivez ces étapes pour configurer le projet Azure DevOps.

1. Ouvrez une fenêtre de navigateur, puis accédez au portail Azure : `https://portal.azure.com/`

1. dans la barre de recherche supérieure, dans la zone de texte Rechercher, entrer **devops**

1. Dans les résultats de recherche, sélectionnez **Organisations Azure DevOps**.

1. Sélectionnez **Mes organisations Azure DevOps**.

1. Dans la page Nous avons besoin de quelques informations supplémentaires, sélectionnez **Continuer**.

1. Dans la page Bien démarrer avec Azure DevOps Services, sélectionnez **Créer l’organisation**, puis sélectionnez **Continuer**.

1. Dans la page C’est presque terminé, entrez les caractères affichés, puis sélectionnez **Continuer**.

1. Dans la page de votre organisation Azure DevOps, sélectionnez **Paramètres de l’organisation**.

1. Dans le menu de gauche sous **Sécurité**, sélectionnez **Stratégies**.

1. Définissez **Autoriser les projets publics** sur **Activé**, puis sélectionnez **Enregistrer**.

1. Revenez à la page de votre organisation DevOps.

1. Sous « Pour commencer, créer un projet », entrez les informations suivantes :

    - Nom du projet : **AZ2003Project**
    - Description : **Projet de code AZ2003**
    - Visibilité : **Public**

1. Sélectionnez **Créer un projet**.

1. Dans le menu de gauche de votre page AZ2003Project, sélectionnez **Dépôt**.

1. Sous Initialiser la branche principale avec un fichier README ou gitignore, sélectionnez **Initialiser**.

1. Sélectionnez **Cloner**, puis **Cloner dans VS Code**.

1. Dans la boîte de dialogue Ce site essaie d’ouvrir Visual Studio Code, sélectionnez **Ouvrir**.

1. Dans la boîte de dialogue Autoriser une extension pour ouvrir cet URI, sélectionnez **Ouvrir**.

1. Dans la fenêtre Choisir un dossier à cloner, sélectionnez **Bureau**, sélectionnez **Nouveau dossier**, tapez **AZ2003**, puis appuyez sur Entrée.

1. Sélectionnez **Sélectionner comme destination du dépôt**.

1. Dans la boîte de dialogue Voulez-vous ouvrir le dépôt cloné, sélectionnez **Ouvrir**, puis **Oui, je fais confiance aux auteurs**.

#### Tâche 2 : Créer une application .NET et la synchroniser avec votre dépôt Azure DevOps

Suivez ces étapes pour créer une application .NET et la synchroniser avec votre dépôt Azure DevOps.

1. Dans le menu Terminal de Visual Studio Code, sélectionnez **Nouveau terminal**.

1. À l’invite de commandes du terminal, pour vérifier que le SDK .NET a été correctement installé, entrez la commande suivante :

    ```dotnetcli
    dotnet --version
    ```

    Si vous obtenez une erreur indiquant que le terme « dotnet » n’est pas reconnu, suivez ces étapes :

    - Dans le menu Démarrer de Windows, ouvrez les **Paramètres** Windows.
    - Dans Paramètres, ouvrez l’onglet **Applications**, puis sélectionnez **Applications installées**.
    - Recherchez **Microsoft .NET SDK 8.0.100 (x64)** dans la liste des applications installées.
    - À droite de Microsoft .NET SDK 8.0.100 (x64), sélectionnez les points de suspension (...), puis **Modifier**.
    - Pour autoriser l’application à apporter des modifications, sélectionnez **Oui**.
    - Dans la fenêtre Microsoft .NET SDK 8.0.100, sélectionnez **Réparer**.
    - Attendez que l’opération de réparation soit terminée, puis sélectionnez **Fermer**.
    - Fermez la fenêtre Paramètres.
    - Revenez à la fenêtre Visual Studio Code, puis fermez Visual Studio Code.
    - Rouvrez Visual Studio Code.
    - Dans le Terminal de Visual Studio Code, à l’invite de commandes, entrez `dotnet --version`
    - Vous devriez voir un numéro de version s’afficher. Par exemple : **8.0.100**.

1. À l’invite de commandes du terminal, pour configurer le paramètre de messagerie Git, utilisez la commande suivante :

    Entrez **git config --global user.email** suivi des informations de messagerie du compte fournies dans votre environnement de labo

    Par exemple : git config --global user.email LabUser-12345678@labhoster.onmicrosoft.com

1. À l’invite de commandes du terminal, pour configurer le nom d’utilisateur Git, utilisez la commande suivante :

    Entrez **git config --global user.email** suivi des informations de nom d’utilisateur du compte fournies dans votre environnement de labo

    Par exemple : git config --global user.name LabUser-12345678

1. Dans le menu Affichage, sélectionnez **Palette de commandes**.

1. À l’invite de commandes, sélectionnez **.NET : Nouveau projet**, puis sélectionnez **ASP.NET Core vide**.

1. Attendez que les ressources se chargent, puis entrez les informations suivantes :

    - Dans la zone de texte Nommer le nouveau projet, entrez **AZ2003App**
    - Acceptez le répertoire par défaut.

1. Ouvrez l’invite de commandes du terminal, puis exécutez la commande CLI dotnet suivante :

    ```dotnetcli
    dotnet build
    ```

1. Dans le dossier de projet racine, créez un fichier .gitignore qui contient les informations suivantes :

    ```gitignore
    [Bb]in/
    [Oo]bj/
    ```

1. Dans le menu Fichier, sélectionnez **Enregistrer tout**.

1. Ouvrez la vue Contrôle de code source.

1. Dans la zone de texte du message de commit, entrez **commit initial**.

1. Sélectionnez **Commiter**, puis **Oui** pour indexer et commiter les modifications.

1. Sélectionnez **Synchroniser les modifications**, puis **OK** pour synchroniser vos fichiers avec le dépôt DevOps.

1. Dans la boîte de dialogue Gestionnaire d’informations d’identification Git, entrez les informations d’identification de votre environnement de labo (nom d’utilisateur et mot de passe).

#### Tâche 3 : Créer une image Docker et envoyer (push) l’image à votre instance Azure Container Registry

Suivez les étapes ci-dessous pour créer une image Docker et envoyer (push) l'image à votre azure Container Registry.

1. Vérifiez que votre projet de code AZ2003 est ouvert dans Visual Studio Code.

1. Pour créer un Dockerfile, exécutez la commande suivante de la palette de commandes : **Docker : Ajoutez des fichiers Docker à l’espace de travail**.

1. Lorsque vous y êtes invité, spécifiez les informations suivantes :

    - Plateforme d'application : **.NET ASP.NET Core**.
    - Système d’exploitation : **Linux**.
    - Ports : **5000**.
    - Inclure les fichiers Docker Compose : **Non**

1. Pour créer une image Docker, exécutez la commande suivante dans la palette de commandes : **Images Docker : Créez une image**.

1. Attendez que le processus de création d’image se termine, puis fermez le terminal.

1. Dans le menu de gauche, pour ouvrir la vue Docker, sélectionnez Docker.

1. Dans la vue DOCKER, sous Registres, sélectionnez **Connecter le registre**, puis **Azure Container Registry**.

1. Dans la vue DOCKER, développez **Azure**, puis sélectionnez **Autoriser**.

1. Dans la fenêtre du navigateur, sélectionnez le compte Azure que vous utilisez pour ce labo.

1. Revenez à Visual Studio Code.

1. Dans la vue DOCKER, développez l’abonnement Azure et vérifiez que l’instance Azure Container Registry que vous avez créée est listée.

1. Pour envoyer (push) l’image Docker à l’instance Azure Container Registry, exécutez la commande suivante dans la palette de commandes : **Images Docker : Envoyer (push)**.

1. Lorsque la commande s’exécute, suivez ces étapes :

    - Sélectionner un groupe d’images : sélectionner **az2003project**
    - Sélectionner une image (étiquette) : sélectionner **Plus récent**
    - Sélectionner le fournisseur de registre : sélectionner **Azure**
    - Sélectionnez votre abonnement.
    - Sélectionner une instance Azure Container Registry comme destination de l’envoi : sélectionner le registre de conteneurs que vous avez créé. Par exemple : acraz2003cah12oct.
    - Pour déployer l’image, appuyez sur Entrée.

1. Attendez que le processus d’envoi d’image se termine, puis fermez le terminal.

1. Ouvrez la vue Contrôle de code source, entrez un message de commit, puis sélectionnez **Commiter** et **Synchroniser les modifications**.

#### Tâche 4 : Créer un pipeline Azure appelé Pipeline1

Suivez ces étapes pour créer un pipeline Azure appelé Pipeline1.

1. Ouvrez le projet Azure DevOps.

1. Dans le menu de gauche, sélectionnez **Pipelines**.

1. Sélectionnez **Créer un pipeline**.

1. Sélectionnez **Azure Repos Git**.

1. Dans la page Sélectionner un dépôt, sélectionnez **AZ2003Project**.

1. Sélectionnez **Pipeline de démarrage**.

1. Sous Enregistrer et exécuter, sélectionnez **Enregistrer**, puis **Enregistrer**.

1. Pour remplacer le nom de votre pipeline par « Pipeline1 », effectuez les étapes suivantes :

    1. Dans le menu de gauche, sélectionnez **Pipelines**.

    1. À droite du pipeline AZ2003Project, sélectionnez **Plus d’options**, puis **Renommer/déplacer**.

    1. Dans la boîte de dialogue Renommer/déplacer le pipeline, sous Nom, entrez **Pipeline1** et sélectionnez **Enregistrer**.

#### Tâche 5 : Déployer un agent Windows autohébergé

Pour qu'un pipeline Azure génère et déploie Windows, Azure et d'autres solutions Visual Studio, vous avez besoin d'au moins un agent Windows dans l'environnement hôte.

Suivez ces étapes pour déployer un agent Windows autohébergé.

1. Accédez à la page d’accueil de votre organisation DevOps.

1. En haut à droite, sélectionnez **Paramètres utilisateur**.

1. Dans la boîte de dialogue Paramètres utilisateur, sélectionnez **Jetons d’accès personnels**.

1. Pour créer un jeton d’accès personnel, sélectionnez **+ Nouveau jeton**.

1. Sous Nom, entrez **AZ2003**.

1. Dans la partie inférieure de la fenêtre Créer un jeton d’accès personnel, pour voir la liste complète des étendues, sélectionnez **Afficher toutes les étendues**.

1. Pour l'étendue, sélectionnez **Pools d'agents (lire, gérer)** et **Groupe de déploiement (lire, gérer)**.

    Vérifiez que toutes les autres cases sont décochées.

1. Sélectionnez **Créer**.

1. Dans la page Opération réussie, pour copier le jeton, sélectionnez **Copier dans le Presse-papiers**, puis **Fermer**.

1. Ouvrez le Bloc-notes et enregistrez-y une copie du jeton.

    Vous allez utiliser ce jeton quand vous configurerez l’agent.

1. Accédez à votre organisation DevOps, puis sélectionnez **Paramètres de l’organisation**.

1. Dans le menu de gauche sous Pipelines, sélectionnez **Pools d'agents**.

1. Si la boîte de dialogue **Obtenir l’agent** s’ouvre, passez à l’étape suivante, sinon effectuez les étapes suivantes :

    1. Pour sélectionner le pool par défaut, sélectionnez **Par défaut**.

        Si le **pool par défaut** n'existe pas, sélectionnez **Ajouter un pool**, puis entrez les informations suivantes :

        1. Sous Type de pool, sélectionnez **Auto-hébergé**.

        1. Sous Nom, entrez **Par défaut**.

        1. Sélectionnez **Créer**.

        1. Pour ouvrir le pool que vous venez de créer, sélectionnez **Par défaut**.

    1. Sous Par défaut, sélectionnez l’onglet **Agents**, puis **Nouvel agent**.

1. Dans la boîte de dialogue Obtenir l'agent, procédez comme suit :

    1. Sélectionnez l'onglet **Windows**.

    1. Dans le volet gauche, sélectionnez **x64**.

    1. Dans le volet droit, sélectionnez **Télécharger**.

1. Attendez la fin du téléchargement.

1. Fermez la boîte de dialogue Obtenir l’agent.

    La prochaine série d’étapes d’instruction vous guide tout au long du processus « Créer l’agent ».

1. Utilisez l’Explorateur de fichiers Windows pour créer l’emplacement de dossier suivant pour l'agent :

    ```dos
    C:\agents
    ```

1. Utilisez l’Explorateur de fichiers Windows pour décompresser le fichier zip de l’agent téléchargé dans le répertoire des agents que vous venez de créer.

1. Attendez que le processus d’extraction du fichier se termine, puis fermez l’Explorateur de fichiers.

1. Ouvrez Windows PowerShell en tant qu’administrateur, accédez au répertoire des agents, puis entrez la commande PowerShell suivante :

    ```powershell
    .\config
    ```

1. Répondez aux invites de configuration comme suit :

    - Enter server URL > : entrez l'URL de votre organisation DevOps. Par exemple : `https://dev.azure.com/<your organization>`
    - Enter authentication type (press enter for PAT) > : appuyez sur Entrée.
    - Enter personal access token > : Collez le jeton d’accès personnel que vous avez copié dans le Bloc-notes.
    - Enter agent pool (press enter for default) > : appuyez sur Entrée.
    - Enter agent name (press enter for YOUR-PC-NAME) > : entrez **az2003-agent**
    - Enter work folder (press enter for _work) > : appuyez sur Entrée.
    - Souhaitez-vous saisir l’agent d’exécution comme service ? (Y/N) (press enter for N) > : entrez **Y**
    - Enter enable SERVICE_SID_TYPE_UNRESTRICTED for agent service (Y/N) (press enter for N) > : entrez **Y**
    - Enter User account to use for the service (press enter for NT AUTHORITY\NETWORK SERVICE) > : appuyez sur Entrée.
    - Indiquez si vous voulez empêcher le démarrage du service immédiatement après la fin de la configuration. (Y/N) (press enter for N) > : appuyez sur Entrée.

    Un message vous informant que l'agent a démarré correctement s'affiche.

    Pour obtenir de l'aide supplémentaire, consultez la documentation suivante : `https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent`

1. Fermez Windows PowerShell.

### Exercice 4 : Configurer Azure Container Registry pour une connexion sécurisée avec Azure Container Apps

Vous êtes invité à configurer les ressources Azure qui répondent aux exigences suivantes :

- Votre groupe de ressources doit inclure une identité managée affectée par l'utilisateur.
- Votre registre de conteneurs doit être en mesure d'utiliser l'identité managée pour extraire des artefacts.
- L'accès pour l'identité managée doit être limité à l'aide du principe du privilège minimum.
- Votre registre de conteneurs doit être accessible à partir d'un point de terminaison privé sur VNET1/PESubnet.

Dans cet exercice, vous allez configurer une instance de registre de conteneurs pour une connexion sécurisée avec une application de conteneur.

La réalisation des tâches suivantes prend environ 10 minutes :

- Configurer une identité managée affectée par l’utilisateur.
- Configurez votre registre de conteneurs avec des autorisations AcrPull pour l'identité managée.
- Configurez votre registre de conteneurs avec une connexion de point de terminaison privé.

#### Tâche 1 : Configurer une identité managée affectée par l’utilisateur

Effectuez les étapes suivantes pour configurer une identité managée affectée par l'utilisateur.

1. Ouvrez votre portail Azure.

1. Dans la barre de recherche supérieure du portail Azure, entrez **identité managée**

1. Dans la liste filtrée des ressources, sélectionnez **Identité managée affectée par l'utilisateur**.

1. Dans la page Créer une identité managée affectée par l'utilisateur, spécifiez les informations suivantes :

    - Abonnement : spécifiez l'abonnement Azure que vous utilisez pour ce projet guidé.
    - Groupe de ressources : **RG1**
    - Région : Entrez la région qui correspond au paramètre de région de votre groupe de ressources.
    - Nom : **uai-az2003**

1. Sélectionnez **Revoir + créer**.

1. Attendez que les paramètres soient validés, puis sélectionnez **Créer**.

1. Fermez la page de l’identité managée.

#### Tâche 2 : Configurer votre registre de conteneurs avec des autorisations AcrPull pour l’identité managée

Effectuez les étapes suivantes pour configurer Container Registry avec des autorisations AcrPull pour l'identité managée.

1. Dans le portail Azure, ouvrez votre ressource Container Registry.

1. Dans le menu de gauche, sélectionnez **Contrôle d'accès (IAM)**.

1. Dans la page Contrôle d'accès (IAM), sélectionnez **Ajouter une attribution de rôle**.

1. Recherchez le rôle AcrPull, puis sélectionnez **AcrPull**.

1. Cliquez sur **Suivant**.

1. Sous l'onglet Membres, à droite de Attribuer l'accès à, sélectionnez **Identité managée**.

1. Sélectionnez **+ Sélectionner des membres**.

1. Dans la page Sélectionner des identités managées, sous Identité managée, sélectionnez **Identité**managée affectée par l'utilisateur, puis sélectionnez l'identité managée affectée par l'utilisateur créée pour ce projet.

    Par exemple : uai-az2003.

1. Dans la page Sélectionner des identités managées, sélectionnez **Sélectionner**.

1. Sous l'onglet Membres de la page Ajouter une attribution de rôle, sélectionnez **Réviser + attribuer**.

1. Dans l’onglet Vérifier + attribuer, sélectionnez **Vérifier + attribuer**.

1. Attendez que l'attribution de rôle soit ajoutée.

    Une notification s’affiche, mais si vous la manquez, vous pouvez accéder à l’onglet Attributions de rôle pour vérifier que le rôle AcrPull a été attribué à uai-az2003.

#### Tâche 3 : Configurer votre registre de conteneurs avec une connexion de point de terminaison privé

Suivez ces étapes pour configurer votre registre de conteneurs avec une connexion de point de terminaison privé.

1. Vérifiez que votre ressource Container Registry est ouverte dans le portail.

1. Sous Paramètres, sélectionnez **Mise en réseau**.

1. Sous l'onglet Accès privé, sélectionnez **+ Créer une connexion de point de terminaison privé**.

1. Sous l'onglet Informations de base, sous Détails du projet, spécifiez les informations suivantes :

    - Abonnement : spécifiez l'abonnement Azure que vous utilisez pour ce projet guidé.
    - Groupe de ressources : **RG1**
    - Nom : **pe-acr-az2003**
    - Région : Vérifiez que la région spécifiée correspond au paramètre de région de votre groupe de ressources.

1. Sélectionnez **Suivant : Ressource**.

1. Sous l'onglet Ressource, vérifiez que les informations suivantes s'affichent :

    - Abonnement : vérifiez que l'abonnement Azure que vous utilisez pour ce projet guidé est sélectionné.
    - Type de ressource : Vérifiez que **Microsoft.ContainerRegistry/registrys** est sélectionné.
    - Resource : Vérifiez que le nom de votre registre est sélectionné.
    - Sous-ressource cible : Vérifiez que le **registre** est sélectionné.

1. Sélectionnez **Suivant : réseau virtuel**.

1. Sous l'onglet Réseau virtuel, sous Mise en réseau, vérifiez que les informations suivantes s'affichent :

    - Réseau virtuel : Vérifiez que VNET1 est sélectionné.
    - Sous-réseau : Vérifiez que PESubnet est sélectionné.

1. Sélectionnez **Suivant : DNS**.

1. Dans l'onglet DNS, sous Intégration DNS privée, assurez-vous que les informations suivantes sont affichées :

    - intégrer avec la zone DNS privée : Vérifiez que l’option **Oui** est sélectionnée.
    - Zone DNS privée : Notez que **(nouveau) privatelink.azurecr.io** est spécifié.

1. Sélectionnez **Suivant : Balises**.

1. Sélectionnez **Suivant : Vérifier + créer**.

1. Sur l'onglet Réviser + créer, lorsque vous lisez le message Validation réussie, sélectionnez **Créer**.

1. Attendez la fin du déploiement.

1. Lorsque vous voyez s’afficher **Votre déploiement a été effectué**, fermez la page de déploiement du point de terminaison privé.

### Exercice 5 : Créer et configurer une application conteneur dans Azure Container Apps

Vous êtes invité à configurer une application conteneur qui répond aux exigences suivantes :

- est déployé sur VNET1/ACASubnet.
- Extrait une image d'un registre de conteneurs.
- S’authentifie avec une identité managée affectée par l’utilisateur (uai-az2003).
- Utilise Container App pour se connecter à une instance Service Bus à l'aide du type de client .NET.
- L’application peut exécuter jusqu’à deux réplicas qui sont ajoutés chaque fois qu’il y a 1 000 requêtes HTTP simultanées.

Dans cet exercice, vous allez déployer une application conteneur à partir d’une image Azure Container Registry sur la plateforme Azure Container Apps.

La réalisation des tâches suivantes prend environ 20 à 25 minutes :

- Créer une application conteneur qui utilise une image Azure Container Registry.
- Configurer l’application conteneur pour qu’elle s’authentifie avec l’identité affectée par l’utilisateur.
- Configurer une connexion entre l’application conteneur et Service Bus.
- Configurer des règles de mise à l’échelle HTTP.

#### Tâche 1 :  Créer une application conteneur qui utilise une image Azure Container Registry

Suivez ces étapes pour créer une application conteneur qui utilise une image Azure Container Registry.

1. Dans la barre de recherche supérieure du portail Azure, entrez **application conteneur**

1. Dans la liste filtrée des ressources, sélectionnez **Container Apps**.

1. Dans la page Container Apps, sélectionnez **Créer une application conteneur**.

1. Dans l'onglet Informations de base, indiquez ce qui suit :

    - Abonnement : spécifiez l'abonnement Azure que vous utilisez pour ce projet guidé.
    - Groupe de ressources : **RG1**
    - Nom de l’application conteneur : **aca-az2003**
    - Région : Vérifiez que la région spécifiée correspond au paramètre de région de VNET1 (qui devrait correspondre à la région de votre groupe de ressources).

        L'application conteneur doit se trouver dans la même région/emplacement que le réseau virtuel afin de pouvoir choisir VNET1 pour l'environnement managé. Pour ce projet guidé, conservez toutes vos ressources dans la région/l'emplacement spécifié pour votre groupe de ressources.

    - Environnement Container Apps : Sélectionnez **Créer nouveau**.

1. Dans la page Créer un environnement Container Apps, sélectionnez l'onglet **Mise en réseau**, puis spécifiez les éléments suivants :

    - Utiliser votre propre réseau virtuel : Sélectionnez **Oui**.
    - Réseau virtuel : sélectionnez **VNET1**.
    - Sous-réseau d'infrastructure : **ACASubnet**.

    > [!NOTE]
    > Si le sous-réseau ACASubnet n'est pas répertorié, annulez ce processus de création, ouvrez votre ressource de réseau virtuel, définissez la plage d’adresses d’ACASubnet sur **10.0.2.0/23**, puis redémarrez les étapes de création de la ressource Container App.

1. Dans la page Créer un environnement Container Apps, sélectionnez **Créer**.

1. Dans la page Créer une application conteneur, sélectionnez l'onglet Conteneur, puis spécifiez les éléments suivants :

    - utiliser l'image de démarrage rapide : **décochez** ce paramètre.
    - Nom : Vérifiez qu’**aca-az2003** est spécifié.
    - Source d'image : vérifiez qu'**Azure Container Registry** est sélectionné.
    - Registre : sélectionnez votre registre de conteneurs. Par exemple : **acraz2003cah12oct.azurecr.io**
    - Image : Sélectionnez **az2003project**
    - Étiquette d'image : Sélectionnez **dernière**.

1. Sélectionnez **Revoir + créer**.

1. Une fois la vérification réussie, sélectionnez **Créer**.

1. Attendez la fin du déploiement.

    > [!NOTE]
    > Ce déploiement peut prendre 5 à 10 minutes.

#### Tâche 2 : Configurer l’application conteneur pour qu’elle s’authentifie avec l’identité affectée par l’utilisateur

Effectuez les étapes suivantes pour configurer l'application conteneur afin qu'elle se connecte à l'aide de l'identité attribuée à l'utilisateur.

1. Sur le portail Azure, ouvrez l'application conteneur que vous avez créée.

1. Sous Paramètres, sélectionnez **Identité**.

1. Sélectionnez l'onglet pour **Utilisateur attribué**.

1. Sélectionnez **Ajouter une identité managée attribuée à l'utilisateur**.

1. Dans la page Ajouter une identité managée affectée par l’utilisateur, sélectionnez **uai-az2003**, puis **Ajouter**.

#### Tâche 3 : Configurer une connexion entre l’application conteneur et Service Bus

Effectuez les étapes suivantes pour configurer une connexion entre l'application conteneur et Service Bus.

1. Sur le portail Azure, vérifiez que votre application conteneur est ouverte.

1. Sous Paramètres, sélectionnez **Connecteur de services (préversion)**.

1. Sélectionnez **Se connecter à vos services**.

1. Dans la page Créer une connexion, spécifiez les éléments suivants :

    - type de service : sélectionnez **Service Bus**.
    - Type de client : Sélectionnez **.NET**.

1. Sélectionnez **Suivant : authentification**.

1. Sous l'onglet Authentification, sélectionnez **Identité managée attribuée à l'utilisateur**.

1. Pour modifier les onglets, sélectionnez **Examiner + Créer**.

1. Une fois que le message de validation réussie apparaît, sélectionnez **Créer**.

1. Attendez la création de la connexion.

     La connexion Service Bus est listée dans la page Connecteur de service (préversion).

#### Tâche 4 : Configurer des règles de mise à l'échelle HTTP

Suivez ces étapes afin de configurer des règles de mise à l’échelle HTTP pour votre application conteneur.

1. Vérifiez que votre application conteneur est ouverte dans le portail.

1. Dans le menu de gauche sous Application, sélectionnez **Révisions et réplicas**.

1. Notez le nom affecté à votre révision active.

1. Dans le menu de gauche sous Application, sélectionnez **Mettre à l’échelle**.

1. Notez le **paramètre Règle de mise à l’échelle** actuel. Configurez les réplicas min/max comme suit :

    - définissez les réplicas min : 0
    - Définissez les réplicas max : 2

1. Sous Règle de mise à l'échelle, sélectionnez **+ Ajouter**.

1. Dans la page Ajouter une règle de mise à l'échelle, spécifiez les éléments suivants :

    - Nom de la règle : saisissez **scalerule-http**
    - Tapez : sélectionnez la **mise à l'échelle HTTP**.
    - Requêtes simultanées : Définissez la valeur sur **1000**.

1. Sur la page Ajouter une règle de mise à l'échelle, sélectionnez **Ajouter**.

1. Dans la page Mise à l’échelle, sélectionnez **Enregistrer en tant que nouvelle révision**.

1. Assurez-vous que votre nouvelle règle de mise à l'échelle est affichée.

    Si la règle de mise à l’échelle ne s’affiche pas après l’actualisation, accédez à la révision sélectionnée pour voir la révision active actuelle, puis ajustez la révision sélectionnée dans la page Mise à l’échelle et réplicas si nécessaire.

### Exercice 6 : Configurer l’intégration continue à l’aide d’Azure Pipelines

Vous avez été invité à configurer un environnement d’intégration continue pour Container Apps qui répond aux exigences suivantes :

- Vous avez besoin d’une tâche de déploiement Azure Container Apps dans votre environnement ADO.
- Pipeline1 doit déployer une image conteneur à partir de votre registre de conteneurs sur votre application conteneur en utilisant un pool d’agents autohébergés.
- Vous devez vous assurer que le pipeline déploie correctement l’image au moins une fois.

Dans cet exercice, vous déployez une application conteneur à partir d'une image dans Azure Container Registry sur la plateforme Azure Container Apps.

La réalisation des tâches suivantes prend environ 10 minutes :

- Configurer l’utilisation du pool d’agents autohébergés pour Pipeline1.
- Configurer Pipeline1 avec une tâche de déploiement Azure Container Apps.
- Exécuter la tâche de déploiement de Pipeline1.

#### Tâche 1 : Configurer l’utilisation du pool d’agents autohébergés pour Pipeline1

Suivez ces étapes pour configurer vos pipelines de sorte qu’ils utilisent le pool d’agents autohébergés.

1. Assurez-vous que votre organisation Azure DevOps est ouverte sous son propre onglet de navigateur.

    Au besoin, ouvrez un nouvel onglet de navigateur, accédez à `https://dev.azure.com` et ouvrez votre organisation Azure DevOps.

1. Dans votre page Azure DevOps, pour ouvrir votre projet DevOps, sélectionnez **AZ2003Project**.

1. Dans le menu de gauche, sélectionnez **Pipelines**.

1. Sélectionnez **Pipeline1**, puis **Modifier**.

1. Pour utiliser le pool d’agents autohébergés, mettez à jour le fichier azure-pipelines.yml, comme illustré dans l’exemple suivant :

    ```yml
    trigger:
    - main

    pool:
      name: default

    steps:

    ```

1. Sélectionnez **Enregistrer**.

1. Entrez un message de validation, puis sélectionnez **Enregistrer**.

#### Tâche 2 : Configurer Pipeline1 avec une tâche de déploiement Azure Container Apps

Suivez ces étapes pour configurer Pipeline1 avec une tâche de déploiement Azure Container Apps.

1. Vérifiez que Pipeline1 est ouvert pour être modifié.

1. Sur le côté droit, sous Tâches, dans le champ Tâches de recherche, entrez **conteneur azure**

1. Dans la liste filtrée des tâches, sélectionnez **Déployer Azure Container Apps**

1. Sous la connexion Azure Resource Manager, sélectionnez l’abonnement que vous utilisez, puis **Autoriser**.

1. Sous l’onglet Portail Azure, ouvrez votre ressource Container App, puis la page Conteneurs.

1. Copiez les informations suivantes dans le Bloc-notes.

    - Nom
    - Registre
    - Image
    - Balise d'image

1. Utilisez les informations que vous avez copiées depuis la page Conteneurs pour configurer les champs suivants relatifs aux informations de la tâche :

    - Image Docker à déployer : Registre/Image:Étiquette d’image (à remplacer par vos informations du Bloc-notes)
    - Nom de l’application conteneur Azure : Nom (à remplacer par vos informations du Bloc-notes)

    Par exemple :

    - Image Docker à déployer : acraz2003cah12oct.azurecr.io/az2003project:latest
    - Nom de l’application conteneur Azure : aca-az2003

1. Dans le champ Nom du groupe de ressources Azure, entrez **RG1**

    > [!NOTE]
    > Si vous devez vérifier le nom du groupe de ressources, vous pouvez le trouver dans la page Vue d’ensemble de votre ressource Container App.

1. Dans la page Déployer Azure Container Apps, sélectionnez **Ajouter**.

    Le fichier Yaml de votre pipeline doit maintenant inclure les tâches AzureContainerApps comme suit :

    ```yml
    trigger:
    - main
    pool:
      name: default
    steps:
    - task: AzureContainerApps@1
      inputs:
        azureSubscription: '<Subscription>(<Subscription ID>)'
        imageToDeploy: '<Registry>/<Image>:<Image tag>' from Container App resource
        containerAppName: '<Name>' from Container App resource 
        resourceGroup: '<resource group name>'
    
    ```

    Voici un exemple d’extrait de code de configuration YAML :

    ```yml
    trigger:
    - main
    pool:
      name: default
    steps:
    - task: AzureContainerApps@1
      inputs:
        azureSubscription: 'Visual Studio Enterprise(1111aaaa-22bb-33cc-44dd-555555eeeeee)'
        imageToDeploy: 'acraz2003cah12oct.azurecr.io/aspnetcorecontainer:latest'
        containerAppName: 'aca-az2003'
        resourceGroup: 'RG1'
    ```

1. Dans votre page Pipeline1, sélectionnez **Enregistrer**, entrez un message de commit, puis sélectionnez **Enregistrer** à nouveau pour commiter.

#### Tâche 3 : Exécuter la tâche de déploiement de Pipeline1

Suivez ces étapes pour exécuter la tâche de déploiement de Pipeline1.

1. Vérifiez que Pipeline1 est ouvert dans Azure DevOps.

1. Pour exécuter la tâche AzureContainerApps, sélectionnez **Exécuter**.

1. Sur la page Exécuter le pipeline, sélectionnez **Exécuter**.

    Une page de pipeline s’ouvre pour afficher le travail associé. Sélectionner le travail affiche l’état du travail, qui passe de Mis en file d’attente à En attente.

1. Vérifiez que « Autorisation nécessaire » apparaît sous Travail.

    Si le travail nécessite l’autorisation de continuer, sélectionnez **Affichage**, puis **Autoriser** pour fournir les autorisations requises.

1. Surveillez l’état de l’opération d’exécution et vérifiez que l’exécution réussit.

    L’exécution du travail en file d’attente peut prendre quelques minutes avant de commencer. Après une minute environ, l’état du travail devrait passer de « En cours d’exécution » à « Réussite ».

### Exercice 7 : Gérer les révisions dans Azure Container Apps

Vous avez été invité à configurer la fraction du trafic pour vos Container Apps afin de répondre aux exigences suivantes :

- vous devez créer une nouvelle révision de l'application conteneur qui utilise le suffixe v2.
- vous devez vous assurer que 25 % des requêtes adressées à votre application sont dirigées vers la révision v2.
- Vous devez étiqueter les révisions « actuelle » et « mise à jour » et vous assurer que les requêtes de révision « ----mise à jour » sont dirigées vers la révision étiquetée v2.

Dans cet exercice, vous déployez une nouvelle révision de votre application conteneur et configurez la fraction du trafic entre deux révisions étiquetées.

La réalisation des tâches suivantes prend environ 5 à 10 minutes :

- Définissez la gestion des révisions sur plusieurs.
- Créez une révision avec un suffixe v2.
- Configurez les étiquettes sur les révisions.
- Configurez un pourcentage de trafic sur les révisions.

#### Tâche 1 : Définir la gestion des révisions sur Multiple

Suivez ces étapes pour définir la gestion des révisions sur Multiple.

1. Dans le portail Azure, ouvrez votre ressource d'application conteneur.

1. Dans le menu de gauche, sous Application, sélectionnez **Révisions et réplicas**.

1. Dans la partie supérieure de la page Révisions, sélectionnez **Choisir le mode Révision**.

1. Pour passer du mode Révision unique au mode Plusieurs révisions, sélectionnez **Confirmer**.

1. À la page Révisions, patientez que le paramètre du **mode Révision** soit mis à jour.

    Le mode Révision est défini sur **Multiple** une fois que la mise à jour est terminée.

#### Tâche 2 : Créer une révision avec un suffixe v2

Suivez ces étapes pour créer une révision avec un suffixe v2.

1. Dans le portail Azure, assurez-vous que la page Révisions de votre ressource d'application conteneur est ouverte.

1. Dans la partie supérieure de la page, sélectionnez **+ Créer une révision**.

1. À la page Créer et déployer une révision, procédez comme suit :

    - Nom/suffixe : Entrer **v2**
    - Sous Image conteneur, sélectionnez votre image conteneur. Par exemple, aca-az2003.

1. Sélectionnez **Créer**.

1. Patientez jusqu'à la fin du déploiement.

#### Tâche 3 : Configurer des étiquettes sur les révisions

L’entrée doit être activée avant de pouvoir configurer les étiquettes de révision ou le fractionnement du trafic.

Suivez ces étapes pour configurer des étiquettes sur les révisions.

1. Dans le menu de gauche, sous Paramètres, sélectionnez **Entrée**.

1. Si l'entrée n'est pas activée, sélectionnez **Activée**.

1. À la page Entrée, spécifiez les informations suivantes :

    - Trafic d'entrée : sélectionnez **Accepter le trafic n'importe où**.

    - Type d'entrée : sélectionnez **HTTP**.

    - Mode de certificat client : vérifiez que **Ignorer** est sélectionné.

    - Transport : vérifiez que **Auto** est sélectionné.

    - Connexions non sécurisées : vérifiez que Autorisées n'est **PAS** coché.

    - Port cible : Entrez **5000**

    - Mode Restrictions de sécurité IP : vérifiez que **Autoriser tout le trafic** est sélectionné.

1. Au bas de la page Entrée, sélectionnez **Enregistrer**, puis patientez jusqu'à la fin de la mise à jour.

1. Dans le menu de gauche, sous Révisions, sélectionnez **Révisions et réplicas**.

1. Pour la révision v2, sous Étiquette, entrez **mise à jour**

1. Pour l'autre révision, entrez **en cours**

1. En haut de la page, sélectionnez **Enregistrer**.

1. Attendez que le paramètre Entrée soit mis à jour.

#### Tâche 4 : Configurer un pourcentage de trafic sur les révisions

Suivez ces étapes pour configurer un pourcentage de trafic sur les révisions.

1. Vérifiez que la page Révisions est ouverte.

1. Pour la révision v2, sous Trafic, entrez **25** comme pourcentage.

1. Pour l'autre révision, sous Trafic, entrez **75** comme pourcentage.

1. En haut de la page, sélectionnez **Enregistrer**.
