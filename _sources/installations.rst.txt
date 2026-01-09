Installations
=============

Cette section couvre l'installation des outils nécessaires au projet.

Comment installer ROS2
----------------------

   Pour installer ROS2 (version Humble), nous recommandons de suivre les ressources suivantes :

   * `Documentation officielle ROS2 Humble <https://docs.ros.org/en/humble/Installation.html>`_
   * `Tutoriel vidéo d'installation (YouTube) <https://www.youtube.com/watch?v=flT3LIIR5qo>`_
   La chaine youtube fait aussi plein d'autres tutoriels sur ROS2 qui pourraient vous interesser.
Comment installer Rviz
----------------------
   .. note::
      Rviz est normalement **déjà installé** si vous avez choisi la version "Desktop" de ROS2 (recommandée).

   Pour vérifier, ouvrez un terminal et tapez :
   ``rviz2``

   Si la commande est introuvable (erreur "command not found"), installez-le manuellement avec :

   .. code-block:: bash

      sudo apt install ros-humble-rviz2