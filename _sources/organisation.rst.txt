Organisation et Installation (ament_cmake)
------------------------------------------

Ce projet utilise le système de compilation **ament_cmake**. Contrairement aux projets purement Python, nous devons déclarer explicitement dans le fichier ``CMakeLists.txt`` quels dossiers doivent être installés pour être visibles par ROS 2.

Structure des dossiers
~~~~~~~~~~~~~~~~~~~~~~

Assurez-vous que votre projet respecte cette arborescence à la racine du paquet :

.. code-block:: text

    five_bar_bot/
    ├── CMakeLists.txt     # <--- Gère l'installation
    ├── package.xml
    ├── config/            # Vos fichiers .yaml
    ├── launch/            # Vos fichiers .launch.py
    ├── description/       # Contient tout le robot
    │    ├── urdf/         # Votre fichier .urdf ou .xacro
    │    └── meshes/       # Vos fichiers .stl
    └── scripts/           # Vos scripts Python (ex: five_bar.py)

Installation des fichiers
~~~~~~~~~~~~~~~~~~~~~~~~~

Pour que ROS trouve votre robot et vos configurations, le fichier ``CMakeLists.txt`` doit contenir les règles d'installation suivantes.

Ouvrez ``CMakeLists.txt`` et vérifiez la section ``install`` :

.. code-block:: cmake

   # 1. Installation des dossiers de configuration et de description
   # Notez que l'on installe le dossier parent "description"
   install(
     DIRECTORY config launch description
     DESTINATION share/${PROJECT_NAME}
   )

   # 2. Installation des scripts Python (exécutables)
   install(
     PROGRAMS scripts/five_bar.py
     DESTINATION lib/${PROJECT_NAME}
   )

.. warning:: **Attention aux noms des dossiers !**
   
   Dans la commande ``install(DIRECTORY ...)``, les noms doivent correspondre exactement à vos dossiers.

   * Si votre dossier s'appelle ``description``, écrivez ``description``.
   * Si vous n'avez pas de dossier description mais directement ``urdf`` à la racine, adaptez la commande.
   
   De même pour le script Python : vérifiez que le nom du fichier dans ``PROGRAMS`` (ex: ``scripts/five_bar.py``) est bien le bon.