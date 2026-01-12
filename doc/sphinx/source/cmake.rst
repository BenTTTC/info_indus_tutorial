Création du fichier CMakeLists.txt
=========================================

Pour que ROS trouve votre robot et vos configurations, le fichier ``CMakeLists.txt`` doit contenir les règles d'installation suivantes.

Créez le fichier ``CMakeLists.txt`` avec la commande suivante et copiez-y le code.

.. code-block:: bash
    
    nano CMakeLists.txt


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