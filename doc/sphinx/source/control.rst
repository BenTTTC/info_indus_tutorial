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
