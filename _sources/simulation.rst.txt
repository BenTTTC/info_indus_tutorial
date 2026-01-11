Pantographe en simulation
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

Modèle Géométrique et Contrôle
==============================

Pour piloter le robot, nous devons traduire une position cartésienne cible $(X, Y)$ en angles moteurs. C'est le rôle du **Modèle Géométrique Inverse (MGI)**.

Schéma Cinématique
------------------

Le robot est une structure parallèle de type "Five-Bar". Voici les paramètres géométriques et les repères définis pour la modélisation :

.. image:: images/schema_cinematique.png
   :width: 80%
   :align: center
   :alt: Schéma cinématique du robot Five Bar

Paramètres du Code
~~~~~~~~~~~~~~~~~~

Le script de contrôle Python intègre les dimensions exactes mesurées sur la CAO  :

* **Bras Gauche (Left Arm -** :math:`A_1 \to A_2 \to P` **)** :
    * Position Moteur (:math:`A_1`) : :math:`x = -0.08m, y = -0.07m`
    * Longueur :math:`L_1` : :math:`0.080m`
    * Longueur :math:`L_2` : :math:`0.157m`

* **Bras Droit (Right Arm -** :math:`A_5 \to A_4 \to P` **)** :
    * Position Moteur (:math:`A_5`) : :math:`x = 0.058m, y = -0.07m`
    * Longueur :math:`L_1` : :math:`0.079m`
    * Longueur :math:`L_2` : :math:`0.147m`

Résolution Mathématique (MGI)
-----------------------------

La fonction ``solve_arm_ik`` du script découpe le problème en deux chaînes articulées indépendantes (RR : Rotoïde-Rotoïde). Pour chaque bras, nous formons un triangle entre le moteur, le coude et la cible.

Nous utilisons le **Théorème d'Al-Kashi** (Loi des cosinus) pour trouver les angles articulaires.

1.  **Calcul de la distance et de l'angle global** :
    On calcule le vecteur entre le moteur et la cible :math:`(dx, dy)` et la distance :math:`D`.

    .. math::

        D = \sqrt{dx^2 + dy^2}

    .. math::

        \alpha = \arctan2(dy, dx)

2.  **Calcul de l'angle moteur** (:math:`\theta_{motor}`) :
    L'angle interne :math:`\beta` du triangle au niveau du moteur est donné par :

    .. math::

        \cos(\beta) = \frac{L_1^2 + D^2 - L_2^2}{2 \cdot L_1 \cdot D}

    L'angle final du moteur dépend de la configuration du coude (signe de :math:`\beta`) :

    .. math::

        \theta_{motor} = \alpha + (\text{config} \times \beta)

3.  **Calcul de l'angle du coude** (:math:`\theta_{elbow}`) :
    Bien que passif en réalité, cet angle est calculé pour la simulation afin de fermer la chaîne cinématique.

    .. math::

        \cos(\gamma) = \frac{L_1^2 + L_2^2 - D^2}{2 \cdot L_1 \cdot L_2}

    .. math::

        \theta_{elbow} = \gamma

Script de Contrôle (Python)
---------------------------

Ce script ROS 2 implémente la logique ci-dessus. Il inclut également :
* Une interpolation linéaire pour lisser les mouvements.
* Des vérifications de sécurité (limites angulaires et portée maximale).
* La publication des commandes pour les 4 joints (2 moteurs + 2 coudes simulés).

.. code-block:: python
   :caption: scripts/five_bar_safe.py
   :linenos:

    #!/usr/bin/env python3
    import rclpy
    from rclpy.node import Node
    from std_msgs.msg import Float64MultiArray
    import math
    import threading
    import time

    # === PARAMETRES EXACTS (URDF) ===
    MOTOR_L_POS = (-0.08, -0.07) 
    L1_L = 0.080   
    L2_L = 0.157   

    MOTOR_R_POS = (0.058, -0.07)
    L1_R = 0.079   
    L2_R = 0.147   

    # === CONTRAINTES DE SECURITE ===
    R_MOT_MIN_DEG = 5.0   
    R_MOT_MAX_DEG = 114.0 

    # === PARAMETRES DE MOUVEMENT ===
    VITESSE_M_S = 0.05 
    FREQ_LOOP = 50.0    
    DT = 1.0 / FREQ_LOOP

    class FiveBarSafe(Node):
        def __init__(self):
            super().__init__('five_bar_safe')
            
            # Publication vers le contrôleur défini dans controllers.yaml
            self.publisher_ = self.create_publisher(
                Float64MultiArray, 
                '/forward_position_controller/commands', 
                10)
            
            # Position initiale sure
            self.curr_x = -0.01
            self.curr_y = 0.12 
            self.target_x = self.curr_x
            self.target_y = self.curr_y

            self.last_valid_cmd = [0.0, 0.0, 0.0, 0.0]
            
            self.timer = self.create_timer(DT, self.control_loop)
            
            # Thread séparé pour ne pas bloquer ROS avec l'input utilisateur
            self.input_thread = threading.Thread(target=self.user_input_loop)
            self.input_thread.daemon = True
            self.input_thread.start()
            
            self.get_logger().info(f"Démarrage sûr à X={self.curr_x}, Y={self.curr_y}")

        def solve_arm_ik(self, target_x, target_y, motor_x, motor_y, L1, L2, elbow_config):
            """
            Calcule la cinématique inverse pour un bras (Moteur -> Coude -> Cible)
            Utilise le théorème d'Al-Kashi.
            """
            dx = target_x - motor_x
            dy = target_y - motor_y
            dist = math.sqrt(dx*dx + dy*dy)
            
            # Vérification de portée (Workspace)
            if dist > (L1 + L2) or dist < abs(L1 - L2):
                return None, None 

            # Calcul de l'angle alpha (direction cible)
            alpha = math.atan2(dy, dx)

            # Calcul de l'angle beta (angle interne moteur) via Al-Kashi
            val_cos_beta = (L1**2 + dist**2 - L2**2) / (2 * L1 * dist)
            val_cos_beta = max(-1.0, min(1.0, val_cos_beta)) # Clamp pour éviter erreurs num.
            beta = math.acos(val_cos_beta)
            
            # Angle moteur final (dépend de la config coude haut/bas)
            theta_motor = alpha + (elbow_config * beta)

            # Calcul de l'angle gamma (angle interne coude) via Al-Kashi
            val_cos_gamma = (L1**2 + L2**2 - dist**2) / (2 * L1 * L2)
            val_cos_gamma = max(-1.0, min(1.0, val_cos_gamma))
            gamma = math.acos(val_cos_gamma)
            
            theta_elbow = gamma

            return theta_motor, theta_elbow

        def control_loop(self):
            # 1. Génération de trajectoire (Interpolation linéaire)
            dx = self.target_x - self.curr_x
            dy = self.target_y - self.curr_y
            dist_to_target = math.sqrt(dx*dx + dy*dy)
            step = VITESSE_M_S * DT
            
            next_x, next_y = self.curr_x, self.curr_y

            if dist_to_target > 0.001:
                if dist_to_target < step:
                    next_x, next_y = self.target_x, self.target_y
                else:
                    ratio = step / dist_to_target
                    next_x += dx * ratio
                    next_y += dy * ratio

            # 2. Calcul IK pour les deux bras
            # Note: elbow_config=1 pour gauche, -1 pour droite
            mot_L, elb_L = self.solve_arm_ik(
                next_x, next_y, MOTOR_L_POS[0], MOTOR_L_POS[1], L1_L, L2_L, 1
            )
            mot_R, elb_R = self.solve_arm_ik(
                next_x, next_y, MOTOR_R_POS[0], MOTOR_R_POS[1], L1_R, L2_R, -1
            )

            valid_move = True

            # Vérification validité solution
            if mot_L is None or mot_R is None:
                valid_move = False

            # Vérification limites moteur droit
            if valid_move:
                mot_R_deg = math.degrees(mot_R)
                if not (R_MOT_MIN_DEG <= mot_R_deg <= R_MOT_MAX_DEG):
                    self.get_logger().warn(f"LIMITE ANGLE : {mot_R_deg:.1f}°")
                    valid_move = False

            # 3. Envoi de la commande
            msg = Float64MultiArray()

            if valid_move:
                self.curr_x = next_x
                self.curr_y = next_y
                # Ordre: [Left_Mot, Right_Mot, Left_Elbow, Right_Elbow]
                # Notez le '-elb_R' pour corriger l'orientation du repère
                self.last_valid_cmd = [mot_L, mot_R, elb_L, -elb_R]
                msg.data = self.last_valid_cmd
            else:
                # Si mouvement invalide, maintien de la position (Torque ON)
                msg.data = self.last_valid_cmd
                if dist_to_target > 0.01:
                   self.target_x = self.curr_x # Reset target si bloqué
                   self.target_y = self.curr_y

            self.publisher_.publish(msg)

        def user_input_loop(self):
            time.sleep(1)
            print("\n--- CONTROLEUR SECURISE ---")
            while rclpy.ok():
                try:
                    print("Entrez cible X Y :")
                    in_x = input("  X > ")
                    in_y = input("  Y > ")
                    x_val = float(in_x)
                    y_val = float(in_y)
                    
                    if y_val < 0.0:
                        print("Refusé (Y < 0)")
                        continue
                    
                    self.target_x = x_val
                    self.target_y = y_val
                except ValueError:
                    print("Erreur de saisie.")

    def main(args=None):
        rclpy.init(args=args)
        node = FiveBarSafe()
        try:
            rclpy.spin(node)
        except KeyboardInterrupt:
            pass
        finally:
            node.destroy_node()
            rclpy.shutdown()

    if __name__ == '__main__':
        main()

Validation du Modèle : Tests Unitaires
======================================

Avant d'intégrer le modèle géométrique inverse (MGI) dans la simulation ROS 2, nous devons valider sa robustesse mathématique. Pour cela, nous utilisons un script de **test unitaire** isolé qui ne nécessite ni Gazebo ni ROS.

Ce script simule des cas d'utilisation critiques pour vérifier que les équations réagissent correctement.

.. code-block:: python
   :caption: tests/test_kinematics.py
   :linenos:

    import unittest
    import math

    # Paramètres simplifiés pour le test (basés sur l'URDF)
    L1 = 0.080
    L2 = 0.157

    def solve_arm_ik_test(target_x, target_y, motor_x, motor_y, config):
        """ 
        Version isolée de la fonction MGI pour le test.
        Permet de tester la logique mathématique sans lancer ROS.
        """
        dx = target_x - motor_x
        dy = target_y - motor_y
        dist = math.sqrt(dx*dx + dy*dy)

        # 1. Vérification de portée (Sécurité)
        if dist > (L1 + L2) or dist < abs(L1 - L2):
            return None # Cible inatteignable

        # 2. Calcul Al-Kashi
        alpha = math.atan2(dy, dx)
        
        # 'Clamp' pour éviter les erreurs numériques (ex: 1.000000002)
        # Si la valeur dépasse 1, math.acos() ferait planter le programme.
        cos_beta = (L1**2 + dist**2 - L2**2) / (2 * L1 * dist)
        cos_beta = max(-1.0, min(1.0, cos_beta)) 
        
        beta = math.acos(cos_beta)
        return alpha + (config * beta)

    class TestInverseKinematics(unittest.TestCase):

        def test_bras_tendu(self):
            """ Cas nominal : Vérifie un résultat connu (bras tendu). """
            # Si on vise exactement la longueur totale du bras (L1+L2) sur l'axe X,
            # l'angle du moteur doit être mathématiquement 0°.
            theta = solve_arm_ik_test(L1 + L2, 0, 0, 0, 1)
            self.assertAlmostEqual(theta, 0.0, places=3)

        def test_hors_portee(self):
            """ Cas d'erreur : Vérifie la sécurité de portée. """
            # On demande une cible impossible (10 mètres).
            # La fonction DOIT renvoyer None et ne pas planter.
            theta = solve_arm_ik_test(10.0, 10.0, 0, 0, 1)
            self.assertIsNone(theta)

        def test_config_signe(self):
            """ Cas d'ambiguïté : Vérifie la gestion du coude (Haut/Bas). """
            # Pour un même point, config=1 et config=-1 doivent donner 
            # deux angles moteurs différents (symétrie du coude).
            target = (0.1, 0.1)
            theta_pos = solve_arm_ik_test(target[0], target[1], 0, 0, 1)
            theta_neg = solve_arm_ik_test(target[0], target[1], 0, 0, -1)
            
            self.assertNotEqual(theta_pos, theta_neg)

    if __name__ == '__main__':
        print("Lancement des tests cinématiques...")
        unittest.main()

Explication des Tests
---------------------

Ce fichier valide trois aspects fondamentaux du contrôle robotique :

1.  **Validité Géométrique (`test_bras_tendu`)** :
    Vérifie que pour une position évidente (bras totalement déplié à l'horizontale), l'algorithme retourne exactement :math:`0` radian. Cela valide que les formules d'Al-Kashi sont correctement implémentées.

2.  **Sécurité de l'Espace de Travail (`test_hors_portee`)** :
    Si l'utilisateur ou le planificateur de trajectoire demande une position impossible (trop loin), le code ne doit pas planter avec une erreur mathématique. Il doit détecter le problème proprement. Ce test confirme que la protection de distance fonctionne.

3.  **Gestion des Singularités (`test_config_signe`)** :
    Vérifie que l'algorithme est capable de distinguer les deux solutions possibles pour atteindre un point (coude vers la gauche ou vers la droite) grâce au paramètre ``config``. C'est crucial pour le robot Five Bar qui utilise les deux configurations simultanément (bras gauche vs bras droit).

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