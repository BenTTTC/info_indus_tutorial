Documentation du projet
=======================

Cette page explique comment créer un site qui permettra de documenter votre projet. Ce site a lui même été créé de cette façon !

Créer et publier une documentation
----------------------------------
Pour créer sa documentation, M.Yguel a déjà fait un tutoriel :

* `Tutoriel officiel M. Yguel - Créer et publier une doc <https://yguel.github.io/informatique_industrielle_avec_ROS2/c01_create_and_publish_doc/p01s02_create_and_publish_doc.html>`_

Contribuer à un projet existant
-------------------------------
Si c'est votre projet dans ce cas là vous pouvez directement passer à l'étape 2 ! 
Si vous voulez modifier un projet existant, il faut d'abord récupérer le projet.

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

.. code-block:: bash

   # 1. Ajouter TOUS les fichiers (nouveaux et modifiés)
   git add .

   # 2. Enregistrer les modifications
   git commit -m "Ajout d'une nouvelle page de doc"

   # 3. Envoyer sur GitHub (remplacer 'rolling' par votre branche)
   git push origin rolling

5. Astuces Pratiques
~~~~~~~~~~~~~~~~~~~~

**Ne plus taper son mot de passe à chaque fois :**
Pour éviter que Git ne demande votre Pseudo et votre Token à chaque ``push``, lancez cette commande une seule fois :

.. code-block:: bash

   git config --global credential.helper store

*(Au prochain push, entrez vos identifiants une dernière fois, et ils seront mémorisés).*