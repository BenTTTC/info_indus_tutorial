Documentation du projet
=======================

Cette page explique comment créer un site qui permettra de documenter votre projet. Ce site a lui même été créé de cette façon !

Créer et publier une documentation
----------------------------------
Pour créer sa documentation, M.Yguel a déjà fait un tutoriel :

* `Tutoriel officiel M. Yguel - Créer et publier une doc <https://yguel.github.io/informatique_industrielle_avec_ROS2/c01_create_and_publish_doc/p01s02_create_and_publish_doc.html>`_

Contribuer à un projet existant
-------------------------------
Si vous voulez modifier un projet existant, suivez les étapes ci-dessous.

0. Prérequis : Configuration et Token
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Avant toute chose, configurez votre identité dans le terminal (à faire une seule fois) :

.. code-block:: bash

   git config --global user.name "Votre Pseudo GitHub"
   git config --global user.email "votre-email@exemple.com"

**Important : Le mot de passe est un Token**
GitHub n'accepte plus votre mot de passe de compte habituel pour les commandes dans le terminal. Il faut utiliser un **Personal Access Token (Classic)**.
Si vous ne l'avez pas, générez-le sur le site GitHub (*Settings > Developer settings > Tokens (classic)*) en cochant la case **repo**.

.. warning::
   Au moment d'envoyer vos fichiers (push), quand le terminal affichera :
   ``Password for 'https://github.com':``
   
   C'est ce **Token** qu'il faudra coller, et pas votre mot de passe GitHub.

1. Récupérer le projet
~~~~~~~~~~~~~~~~~~~~~~
Assurez-vous d'abord d'avoir été ajouté comme **Collaborateur** sur le dépôt GitHub et d'avoir accepté l'invitation par email.

Ensuite, ouvrez un terminal et clonez le dépôt :

.. code-block:: bash

   git clone https://github.com/BenTTTC/info_indus_tutorial.git

La partie ``BenTTTC/info_indus_tutorial`` est bien sûr à remplacer par le nom du GitHub que vous voulez modifier !

Placez vous ensuite dans le répertoire du projet que vous venez de cloner :

.. code-block:: bash

   cd info_indus_tutorial

Le dossier ``info_indus_tutorial`` est bien sûr à remplacer par le nom de votre dossier cloné.

2. Identifier sa branche de travail
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Avant de modifier quoi que ce soit, vérifiez sur quelle branche vous êtes (parfois ``main``, ``master`` ou ``rolling``). Tapez :

.. code-block:: bash

   git branch

La branche actuelle est celle avec une étoile ``*`` devant.
*Exemple : si vous voyez* ``* rolling``, *c'est que vous travaillez sur la branche rolling.*

3. Créer une nouvelle page (Sous-partie)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Pour ajouter une nouvelle page (comme ``installations.rst`` par exemple) et mieux organiser le site :

**Étape A : Créer le fichier**
Créez un nouveau fichier dans le dossier ``source`` avec l'extension ``.rst`` :

.. code-block:: bash

   nano ma_nouvelle_page.rst

Ajoutez obligatoirement un titre souligné avec des signes égal ``=`` :

.. code-block:: rst

   Mon Titre de Page
   =================

   Mon contenu ici...

**Étape B : Lier la page au menu (Index)**
Ouvrez le fichier ``index.rst`` et ajoutez le nom de votre fichier (sans le .rst) dans la liste ``toctree`` :

.. code-block:: rst

   .. toctree::
      :maxdepth: 2
      :caption: Contents:

      installations
      ma_nouvelle_page

.. warning::
   Attention à l'alignement ! Le nom de votre fichier doit être aligné avec les autres (généralement 3 espaces).

4. Sauvegarder et Envoyer
~~~~~~~~~~~~~~~~~~~~~~~~~
Une fois vos modifications effectuées (fichiers créés ou modifiés) :

**1. Ajouter TOUS les fichiers (nouveaux et modifiés)**

.. code-block:: bash

   git add .

**2. Enregistrer les modifications (avec un message clair)**

.. code-block:: bash

   git commit -m "Ajout d'une nouvelle page de doc"

**3. Envoyer sur GitHub (remplacer 'rolling' par votre branche)**

.. code-block:: bash

   git push origin rolling

5. Astuces Pratiques
~~~~~~~~~~~~~~~~~~~~

**Ne plus taper son mot de passe à chaque fois :**
Pour éviter que Git ne demande votre Pseudo et votre Token à chaque ``push``, lancez cette commande une seule fois :

.. code-block:: bash

   git config --global credential.helper store

*(Au prochain push, entrez vos identifiants une dernière fois, et ils seront mémorisés).*

6. Vérifier et Voir le résultat
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Une fois le ``git push`` effectué, le travail n'est pas tout à fait fini ! GitHub doit maintenant reconstruire le site.

**Vérifier que tout a fonctionné :**
Allez sur la page du projet GitHub et cliquez sur l'onglet **Actions** (en haut).
Vous verrez votre dernière modification dans la liste :

* 🟡 **Cercle Jaune** : Le site est en cours de construction (patience...).
* ✅ **Coche Verte** : Le site est prêt !
* ❌ **Croix Rouge** : Il y a une erreur dans le code (souvent un problème d'alignement dans le .rst).



**Voir votre magnifique site :**
Le lien du site se trouve généralement dans la section "About" à droite sur la page d'accueil du GitHub, ou dans *Settings > Pages*.

.. tip::
   **Le site ne change pas ?**
   Les navigateurs gardent l'ancienne version en mémoire ("Cache").
   Pour être sûr de voir vos modifications, ouvrez le lien en **Navigation Privée** (``Ctrl + Maj + N``) ou forcez l'actualisation (``Ctrl + F5``).

Bravo ! 🎉
==========
