Organisation des fichiers
===============================

Ce projet utilise le système de compilation **ament_cmake**. Contrairement aux projets purement Python, nous devons déclarer explicitement dans le fichier ``CMakeLists.txt`` quels dossiers doivent être installés pour être visibles par ROS 2.

Structure des dossiers
--------------------------

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

