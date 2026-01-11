Compilation et Lancement
========================

Une fois le code implémenté et validé, nous pouvons lancer la simulation complète. Cette procédure nécessite l'utilisation de deux terminaux : l'un pour l'environnement de simulation, l'autre pour le script de contrôle.

1. Compilation du paquet
------------------------

Avant toute chose, assurez-vous de compiler les dernières modifications (URDF, scripts, configuration) depuis la racine de votre workspace :

.. code-block:: bash

   colcon build --packages-select five_bar_bot --symlink-install

.. note:: 
   L'option ``--symlink-install`` est très utile en développement : elle permet de modifier les scripts Python (comme ``five_bar.py``) sans avoir à recompiler à chaque fois.

2. Lancement de la Simulation (Terminal 1)
------------------------------------------

Dans ce premier terminal, nous sourçons l'environnement et lançons la visualisation (RViz/Gazebo) ainsi que le `controller_manager`.

.. code-block:: bash

   source install/setup.bash
   ros2 launch five_bar_bot view_robot.launch.py

À ce stade, le robot doit apparaître dans la fenêtre de simulation. Il est immobile et maintenu en position par les contrôleurs.

.. warning:: **Si le robot n'apparaît pas dans RViz**

   Il est fréquent que RViz s'ouvre "vide" lors du premier lancement. Vous devez configurer l'affichage manuellement :

   1. **Changer le repère fixe (Fixed Frame)** : 
      Dans le panneau de gauche "Displays", sous **Global Options**, changez l'option **Fixed Frame** de ``map`` à ``base_link`` (ou ``world`` selon votre URDF).
   
   2. **Ajouter le modèle du robot** :
      Cliquez sur le bouton **Add** (en bas à gauche), cherchez **RobotModel** dans la liste et cliquez sur **OK**.
      
   Le robot devrait maintenant apparaître.

   .. tip:: 
      Pour ne plus avoir à refaire ces étapes, faites **File > Save Config As...** dans RViz une fois tout réglé.

3. Exécution du Contrôle (Terminal 2)
-------------------------------------

Ouvrez un **nouveau terminal**. Il est nécessaire de recharger l'environnement ROS 2 ici aussi.

.. code-block:: bash

   source install/setup.bash

Lancez ensuite le script de pilotage Python :

.. code-block:: bash

   # Si le script a été installé via setup.py :
   ros2 run five_bar_bot five_bar.py
   
   # Ou directement via Python (selon votre arborescence) :
   # python3 src/five_bar_bot/scripts/five_bar.py

Une interface textuelle devrait apparaître vous demandant de saisir les coordonnées cibles :

.. code-block:: text

   --- CONTROLEUR SECURISE ---
   Entrez cible X Y :
     X > 

Vous pouvez maintenant entrer des valeurs (par exemple ``0.0`` et ``0.15``) et observer le robot bouger dans la fenêtre de simulation du premier terminal.