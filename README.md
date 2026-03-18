Projet PAA 

--------------------------------------------------------------------------------------------------
||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
--------------------------------------------------------------------------------------------------
Présentation

Ce projet consiste à modéliser un réseau électrique composé de générateurs et de maisons.
Le programme permet d’ajouter, modifier et connecter des maisons à des générateurs, de charger et sauvegarder un réseau depuis un fichier, de calculer un coût et d’optimiser le réseau.

--------------------------------------------------------------------------------------------------
||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
--------------------------------------------------------------------------------------------------
Version 

Langage : Java
Version Java : Java 25
Tests unitaires : JUnit 4

--------------------------------------------------------------------------------------------------
||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
--------------------------------------------------------------------------------------------------
Arborescence du projet 

ProjetPAA/
│
├── src/ (code principal)
│   └── java/
│       ├── ReseauMain.java
│       └── ReseauPAA/
│           ├── Reseau.java
│           ├── Maison.java
│           ├── Generateur.java
│           ├── Consommation.java
│           └── TestAlgo.java
│
├── java/ (test unitaires)
│   └── test/
│       ├── ReseauTest.java
│       ├── MaisonTest.java
│       └── GenerateurTest.java
│
├── out/ (code compilé)
└── ProjetPAA.iml

--------------------------------------------------------------------------------------------------
||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
--------------------------------------------------------------------------------------------------
Compilation et exécution

Depuis la racine du projet vous devez exécuter la commande suivante : 

java -cp out/production/ProjetPAA ReseauMain

Si vous souhaitez lancer le programme avec un fichier en entrée (Attention : le fichier doit toujours être accompagné d'une valeur lambda). Pour pouvoir utiliser notre commande sur tous vos fichiers veuillez les insérez à la racine du projet. 

java -cp out/production/ProjetPAA ReseauMain reseau.txt 10
--------------------------------------------------------------------------------------------------
||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
--------------------------------------------------------------------------------------------------
Fonctionnement général 

- ReseauMain est le point d'entrée du programme 
- La classe Réseau est notre classe dites principal, elle gère tous le réseau d'électricité
- Les classes Maison, Générateurs et Consommation servent à modéliser les entités du réseau
- La classe TestAlgo sert à tester la rapidité de notre algorithme optimisé. 

--------------------------------------------------------------------------------------------------
||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
--------------------------------------------------------------------------------------------------
Partie algorithmique 

### 1. Problème à résoudre

L'on souhaite pouvoir faire un algorithme qui nous permettrait d'avoir le coût d'un réseau optimal.
## Partie algorithmique

### 2. Algorithme naïf

L’algorithme naïf proposé repose sur une approche de recherche locale aléatoire.L’objectif est de minimiser le coût global du réseau, calculé à l’aide d’une fonction de coût dépendant du paramètre λ (lambda), qui pénalise notamment les surcharges ou déséquilibres entre générateurs et maisons.

À chaque itération (jusqu’à un nombre maximal k d’itérations), l’algorithme :

sélectionne aléatoirement une maison,sélectionne aléatoirement un générateur,tente de déplacer la maison vers ce nouveau générateur, si celui-ci est différent de son générateur actuel.Une fois le déplacement effectué, le coût du nouvel état du réseau est recalculé et on conserve le coût si celui ci est meilleur que l'ancien.

Limites de l’algorithme naïf

Bien que simple à implémenter et fonctionnel, cet algorithme présente plusieurs limites. Les choix fait aléatoirement ne sont pas optimal. De plus la complexité augmente avec le nombre de maisons et de générateurs, ce qui réduit son efficacité sur de grandes instances.

### 3. Algorithme optimisé 

Pour commencer, nous avons fais les test avec l'algorithme naïf basé sur des déplacements aléatoires. C’est une approche qui permet d'obtenir très vite une solution valide, mais le problème c'est qu'elle a rapidement montrer ses limites en se bloquant dans des minimums locaux et la qualité dépend trop du facteur aléatoire de random. On a tenté d'améliorer ça en acceptant parfois des solutions moins bonnes pour "débloquer" l'algorithme, mais ça restait trop instable.

C'est pourquoi on a changé de stratégie avec une recherche locale déterministe. Ici, on a abandonné complètement le hasard : l'algorithme explore systématiquement tout le voisinage (il teste chaque maison vers chaque générateur) et choisit uniquement le déplacement qui réduit le plus le coût. L'avantage, c'est que la convergence est propre et rapide, mais le gros défaut, c'est que l'algorithme ce retrouvait encore trop souvent dans des minimum locaux.

Pour corriger ce problème, on est passé à l'Iterated Local Search (ILS). Le principe est de garder cette recherche locale pour l'efficacité, mais d'ajouter des perturbations quand on est bloqué, de manière à sortir du minimum local. Enfin, pour rendre l'algorithme vraiment robuste, on a adopté une stratégie Multi-départ qui consiste à lancer plusieurs ILS depuis des points différents, ce qui permet de bien couvrir l'espace de recherche et de ne plus dépendre de la qualité de la solution initiale.


L’algorithme multi-départ repose sur le principe de diversification des points de départ initiales afin d’éviter le blocage dans un minimum local. Plusieurs exécutions indépendantes de la recherche locale sont lancées à partir de points de départ différents. Pour certains départs, une solution est générée à l’aide de l’algorithme naïf, tandis que pour d’autres, une initialisation volontairement perturbée est appliquée afin de modifier fortement la structure de la solution. Chaque départ est ensuite amélioré par une recherche locale déterministe, qui converge vers un minimum local. Le meilleur coût obtenu parmi l’ensemble des départs est conservé comme solution finale. Cette approche permet d’explorer différentes régions de l’espace des solutions et augmente la probabilité d’atteindre une solution proche du minimum global.

Avantages : 

- Limiter le blocage dans des optima locaux grâce aux multi-départs
- Explorer plus efficacement l’espace des solutions
- Obtenir des solutions significativement meilleures que l’algorithme naïf, tout en restant dans un temps de calcul raisonnable


Algorithme ILS_MultiDepart(λ, k, nbDeparts)

    meilleurCoutGlobal <- Max

    Pour start de 0 à nbDeparts − 1 faire


        Si start est pair alors
            Appliquer AlgoNaif(λ, k)
        Sinon
            Pour chaque maison m du réseau faire
                Si m possède un générateurs
                    Connecter m à un générateur différent
                FinSi
            FinPour
        FinSi

        coutCourant ← calculCout(λ)


        Répéter
            amelioration <- faux
            meilleurVoisin <- coutCourant
            meilleureMaison <- null
            meilleurGenerateur <- null
            ancienGenerateur <- null

            Pour chaque maison m du réseau faire
                Si m possede generateur 
                    g <- le generateur de m

                Pour chaque générateur g du réseau faire
                    Si g != générateur actuel alors

                        Déplacer m de son générateur actuel vers g
                        coutTest <- calculCout(λ)

                        Si coutTest < meilleurVoisin alors
                            meilleurVoisin <- coutTest
                            meilleureMaison <- m
                            meilleurGenerateur <- g
                            ancienGenerateur <- générateur actuel
                        FinSi

                        Annuler le déplacement de m
                    FinSi
                FinPour
            FinPour

            Si une amélioration a été trouvée alors
                Appliquer définitivement le meilleur déplacement
                coutCourant <- meilleurVoisin
                amelioration <- vrai
            FinSi

        Jusqu’à ce qu’aucune amélioration ne soit possible


        Si coutCourant < meilleurCoutGlobal alors
            meilleurCoutGlobal <- coutCourant
        FinSi

    FinPour

    Retourner meilleurCoutGlobal

FinAlgorithme


--------------------------------------------------------------------------------------------------
||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
--------------------------------------------------------------------------------------------------
Précision

Nous avons essayé d'implémenter le projet et de respecter les consignes du mieux que nous pouvions.
Plusieurs fonctionnalités et beug ont été corrigé comme le fait qu'on gère toutes les erreurs (à notre 
connaissance) concernant la première partie du projet. C'est à dire par exemple
l'ajout d'espace lors de la création ou connexion de maison 
et générateurs. 

Cependant nous n'avons pas terminé tous nos tests unitaires et n'avons pas eu le temps 
de concevoir une interface graphique, c'est une éventuelle amélioration de notre projet. 
