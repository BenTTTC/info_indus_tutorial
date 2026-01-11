Création de l'URDF
=========================

Source des fichiers
-------------------
Le modèle 3D de base provient du cours de M. Yguel.
Fichier source : ``maquette-5-barres_asm.stp`` (Format STEP).

Lien : `Cours Informatique Industrielle <https://yguel.github.io/informatique_industrielle_avec_ROS2/p00_pentograph_pencil_holder_with_rpi_and_dynamixel/p50s03_urdf.html>`_

Comment bien exporter les pièces du Créo
----------------------------------------

Pour que le robot s'articule correctement dans Rviz ou Gazebo, l'export des fichiers 3D doit suivre une méthodologie rigoureuse.

1. **Format de fichier** : Bien que la source soit en STEP (.stp), les fichiers doivent être exportés en **.STL** pour être lisibles par l'URDF.
2. **Unité** : La CAO est souvent en millimètres (mm). ROS travaille en mètres (m).
   
   * *Astuce* : On applique une échelle de ``0.001`` dans l'URDF plus tard, pas besoin de redimensionner la pièce dans Créo.

3. **Le Repère (Origine)** : C'est l'étape la plus critique.
   
   Lors de l'export de chaque pièce (base, bielles, bras), il faut définir un repère de sortie.
   
   .. warning::
      Il ne faut pas utiliser l'origine globale de l'assemblage !
      
      Il faut choisir le repère situé **au niveau de la liaison avec la pièce précédente (le parent)**.
      
      * Exemple : Pour la bielle gauche, l'origine du fichier STL doit être placée exactement sur l'axe du moteur gauche.
      * Si cela n'est pas fait, la pièce tournera autour d'un point arbitraire au lieu de tourner autour de son axe.

Comment faire l'URDF
--------------------

L'URDF (Unified Robot Description Format) est un fichier XML qui décrit la structure cinématique du robot.

La contrainte de la boucle fermée (5 barres)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Notre robot pantographe est mécaniquement une **boucle fermée** (mécanisme 5 barres). Cependant, le format URDF impose une structure stricte en **arbre** (arborescence parent-enfant), où chaque pièce ne peut avoir qu'un seul parent.

.. error::
   Il est **impossible** de définir une boucle fermée directement dans un fichier URDF. Si l'on essaie de relier le dernier maillon au premier, ROS ne pourra pas construire l'arbre cinématique.

**La Solution : 2 boucles ouvertes**

Pour contourner ce problème, nous définissons le robot dans l'URDF comme **deux bras indépendants** (deux chaînes ouvertes) qui partent de la même base :

1.  **Branche gauche :** Base → Moteur G → Bielle G → Bras G
2.  **Branche droite :** Base → Moteur D → Bielle D → Bras D

Visuellement, les bras se touchent à la fin, mais informatiquement, ils sont séparés. C'est le simulateur ou le contrôleur qui assurera la cohérence physique.

Import des fichiers 3D dans l'URDF
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

L'URDF ne lit pas directement les fichiers `.stp`. Il faut utiliser les fichiers `.stl` exportés précédemment. On utilise la balise ``<mesh>`` à l'intérieur de ``<geometry>``.

La syntaxe est la suivante : ``package://nom_du_dossier/chemin/fichier.stl``.

Code URDF complet
~~~~~~~~~~~~~~~~~

Voici le code utilisé pour décrire le pantographe. Notez l'utilisation de ``scale="0.001 0.001 0.001"`` pour convertir les meshes (mm) en mètres lors de l'import.

.. code-block:: xml
    :caption: description/urdf/five_bar.urdf.xacro

    <?xml version="1.0"?>
    <robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="five_bar_bot">

      <link name="base_link">
        <visual>
          <origin xyz="0 0 0" rpy="1.57 0 0"/>
          <geometry>
            <mesh filename="package://five_bar_bot/description/meshes/base.stl" scale="0.001 0.001 0.001"/>
          </geometry>
          <material name="grey">
            <color rgba="0.5 0.5 0.5 1"/>
          </material>
        </visual>
      </link>

      <joint name="left_motor_joint" type="revolute">
        <parent link="base_link"/>
        <child link="left_bielle_link"/>
        <origin xyz="-0.08 -0.07 0.032" rpy="0 0 0" />
        <axis xyz="0 0 1"/>
        <limit lower="-6.28" upper="6.28" effort="10.0" velocity="5.0"/>
      </joint>

      <link name="left_bielle_link">
        <visual>
          <origin xyz="0 0 0" rpy="0 0 0"/>
          <geometry>
            <mesh filename="package://five_bar_bot/description/meshes/link1.stl" scale="0.001 0.001 0.001"/>
          </geometry>
          <material name="blue">
            <color rgba="0 0 0.8 1"/>
          </material>
        </visual>
      </link>

      <joint name="left_elbow_joint" type="revolute">
        <parent link="left_bielle_link"/>
        <child link="left_arm_link"/>
        <origin xyz="0.08 0 0.054" rpy="0 0 3.14"/> 
        <axis xyz="0 0 1"/>
        <limit lower="-6.28" upper="6.28" effort="10.0" velocity="5.0"/>
      </joint>

      <link name="left_arm_link">
        <visual>
          <origin xyz="0 0 0" rpy="1.57 0 3.92699081699"/>
          <geometry>
             <mesh filename="package://five_bar_bot/description/meshes/link2.stl" scale="0.001 0.001 0.001"/>
          </geometry>
          <material name="white">
             <color rgba="1 1 1 1"/>
          </material>
        </visual>
      </link>

      <joint name="right_motor_joint" type="revolute">
        <parent link="base_link"/>
        <child link="right_bielle_link"/>
        <origin xyz="0.058 -0.07 0.032" rpy="0 0 0"/> 
        <axis xyz="0 0 1"/>
        <limit lower="-6.28" upper="6.28" effort="10.0" velocity="5.0"/>
      </joint>

      <link name="right_bielle_link">
        <visual>
          <origin xyz="0 0 0" rpy="0 0 0"/>
          <geometry>
            <mesh filename="package://five_bar_bot/description/meshes/link4.stl" scale="0.001 0.001 0.001"/> 
          </geometry>
          <material name="blue"/>
        </visual>
      </link>

      <joint name="right_elbow_joint" type="revolute">
        <parent link="right_bielle_link"/>
        <child link="right_arm_link"/>
        <origin xyz="0.079 0 0.053" rpy="0 0 3.14"/> 
        <axis xyz="0 0 1"/>
        <limit lower="-6.28" upper="6.28" effort="10.0" velocity="5.0"/>
      </joint>

      <link name="right_arm_link">
        <visual>
          <origin xyz="0 0 0" rpy="-1.57 0 -3.92699081699"/>
          <geometry>
            <mesh filename="package://five_bar_bot/description/meshes/link3.stl" scale="0.001 0.001 0.001"/>
          </geometry>
          <material name="white"/>
        </visual>
      </link>

      <ros2_control name="FiveBarBotSystem" type="system">
        <hardware>
          <plugin>mock_components/GenericSystem</plugin>
        </hardware>
        
        <joint name="left_motor_joint">
          <command_interface name="position"/>
          <state_interface name="position"/>
        </joint>
        <joint name="right_motor_joint">
          <command_interface name="position"/>
          <state_interface name="position"/>
        </joint>

        <joint name="left_elbow_joint">
          <command_interface name="position"/> <state_interface name="position"/>
        </joint>

        <joint name="right_elbow_joint">
          <command_interface name="position"/> <state_interface name="position"/>
        </joint>
      </ros2_control>

    </robot>

Configuration des contrôleurs
-----------------------------

Pour piloter le robot en simulation, nous devons définir les paramètres du gestionnaire de contrôle (`controller_manager`). Créez un fichier nommé ``controllers.yaml`` (généralement situé dans le dossier ``config/`` de votre paquet) et ajoutez-y le contenu suivant :

.. code-block:: yaml
   :caption: config/controllers.yaml

   controller_manager:
     ros__parameters:
       update_rate: 100  # Hz

       joint_state_broadcaster:
         type: joint_state_broadcaster/JointStateBroadcaster

       forward_position_controller:
         type: forward_command_controller/ForwardCommandController

   forward_position_controller:
     ros__parameters:
       interface_name: position
       joints:
         - left_motor_joint
         - right_motor_joint
         - left_elbow_joint   # Nécessaire pour la simulation
         - right_elbow_joint  # Nécessaire pour la simulation

.. note:: **Pourquoi contrôler les coudes ("elbows") ?**

   Vous remarquerez l'ajout de ``left_elbow_joint`` et ``right_elbow_joint`` dans la liste des articulations contrôlées.

   Bien que ces articulations soient des **liaisons passives** dans la réalité (elles n'ont pas de moteurs et suivent simplement le mouvement mécanique), elles doivent être déclarées comme **contrôlées** dans la simulation.
   
   Cette configuration est indispensable pour **fermer cinématiquement la boucle**. Sans cela, le simulateur physique pourrait traiter la structure comme une chaîne ouverte, entraînant un comportement physique incorrect ou instable. En leur attribuant une interface de position, nous forçons le simulateur à respecter la contrainte de fermeture géométrique du mécanisme.
