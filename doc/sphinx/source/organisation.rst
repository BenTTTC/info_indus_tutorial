Organisation des fichiers
-------------------------

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
    :caption: CMakeLists.txt

    cmake_minimum_required(VERSION 3.8)
    project(five_bar_bot)

    # Options de compilation standard
    if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wall -Wextra -Wpedantic)
    endif()

    # 1. On trouve les dépendances
    find_package(ament_cmake REQUIRED)
    find_package(rclcpp REQUIRED)
    find_package(hardware_interface REQUIRED)
    find_package(pluginlib REQUIRED)
    find_package(controller_manager REQUIRED)


    install(
    DIRECTORY config description launch
    DESTINATION share/${PROJECT_NAME}
    )
    install(
    PROGRAMS scripts/five_bar_safe.py
    DESTINATION lib/${PROJECT_NAME}
    )

    # 3. Finalisation
    ament_package()

.. warning:: **Attention aux noms des dossiers !**
   
   Dans la commande ``install(DIRECTORY ...)``, les noms doivent correspondre exactement à vos dossiers.

   * Si votre dossier s'appelle ``description``, écrivez ``description``.
   * Si vous n'avez pas de dossier description mais directement ``urdf`` à la racine, adaptez la commande.
   
   De même pour le script Python : vérifiez que le nom du fichier dans ``PROGRAMS`` (ex: ``scripts/five_bar_safe.py``) est bien le bon.