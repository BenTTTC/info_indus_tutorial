Documentation du projet
=======================

Cette page explique comment participer au développement de ce site.

Ressources pédagogiques
-----------------------
Pour comprendre la structure de la documentation, référez-vous au support de cours :

* `Tutoriel officiel M. Yguel - Créer et publier une doc <https://yguel.github.io/informatique_industrielle_avec_ROS2/c01_create_and_publish_doc/p01s02_create_and_publish_doc.html>`_

Guide du contributeur (Git)
---------------------------

1. Récupérer le projet
~~~~~~~~~~~~~~~~~~~~~~
Assurez-vous d'abord d'avoir été ajouté comme **Collaborateur** sur le dépôt GitHub et d'avoir accepté l'invitation par email.

Ensuite, ouvrez un terminal et clonez le dépôt :

.. code-block:: bash

   git clone https://github.com/BenTTTC/info_indus_tutorial.git
   cd info_indus_tutorial

2. Identifier sa branche de travail
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Avant de modifier quoi que ce soit, vérifiez sur quelle branche vous êtes (parfois ``main``, ``master`` ou ``rolling``). Tapez :

.. code-block:: bash

   git branch

La branche actuelle est celle avec une étoile ``*`` devant.
*Exemple : si vous voyez* ``* rolling``, *c'est que vous travaillez sur la branche rolling.*

3. Modifier et Sauvegarder (Workflow Git)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Une fois vos modifications effectuées sur les fichiers ``.rst`` :

.. code-block:: bash

   # 1. Ajouter les fichiers modifiés
   git add .

   # 2. Enregistrer les modifications (avec un message clair)
   git commit -m "Description de ma modification"

   # 3. Envoyer sur GitHub (remplacer 'rolling' par votre branche)
   git push origin rolling

4. Astuces Pratiques
~~~~~~~~~~~~~~~~~~~~

**Ne plus taper son mot de passe à chaque fois :**
Pour éviter que Git ne demande votre Pseudo et votre Token à chaque ``push``, lancez cette commande une seule fois :

.. code-block:: bash

   git config --global credential.helper store

*(Au prochain push, entrez vos identifiants une dernière fois, et ils seront mémorisés).*