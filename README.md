# LAB-17-CrackerOWASP-Uncrackable-Android-Level-3
C'est quoi l'objectif de ce lab ?
Ce lab, c'est le niveau 3 du challenge OWASP Uncrackable App pour Android. L'application demande un mot de passe secret, et notre mission c'est de le retrouver sans l'avoir jamais eu — uniquement par analyse statique du code et de la bibliothèque native.
La difficulté par rapport aux niveaux précédents : la logique de vérification n'est plus en Java, elle est enfouie dans du code natif C/C++, avec de l'obfuscation en prime. On va utiliser JADX pour le Java, Ghidra pour le binaire natif, et CyberChef pour déchiffrer le flag final.

Étape 1 — Analyser la couche Java avec JADX
On commence par ouvrir l'APK dans JADX pour lire le code Java décompilé. Première observation : l'application délègue la vérification du mot de passe à une bibliothèque native C/C++. Elle ne fait pas le travail elle-même.
Ce qui est intéressant côté Java, c'est la classe MainActivity. Elle définit une chaîne statique appelée xorkey avec la valeur :
pizzapizzapizzapizzapizz
Cette clé est ensuite passée à une méthode native init(). On va s'en souvenir, elle servira plus tard pour le déchiffrement.
<img width="1488" height="747" alt="Analyse JADX - xorkey" src="https://github.com/user-attachments/assets/eb1234c7-ae95-466f-9eb6-18f9752ab663" />

Étape 2 — Retrouver la logique de vérification dans le code natif
La vérification réelle se passe dans la bibliothèque libfoo.so. On l'extrait de l'APK et on l'ouvre dans Ghidra pour l'analyser.
La fonction qui nous intéresse s'appelle :
Java_sg_vantagepoint_uncrackable3_CodeCheck_bar
C'est la fonction JNI exportée qui contient la boucle de validation du mot de passe. C'est là que le code compare ce que l'utilisateur tape avec le secret attendu.
<img width="1489" height="748" alt="Bibliothèque libfoo.so dans Ghidra" src="https://github.com/user-attachments/assets/b25b531a-af42-4ac5-ac1d-d6caa37486e8" />
<img width="1249" height="617" alt="Fonction JNI bar dans Ghidra" src="https://github.com/user-attachments/assets/e9733c5f-d993-4673-83b7-7c4018e1f24a" />

Étape 3 — Contourner l'obfuscation du flux de contrôle
La fonction FUN_001012c0 est celle qui construit le buffer local_48 contenant le payload chiffré. Elle est volontairement obfusquée : on y trouve des allocations mémoire factices et des bifurcations du flux de contrôle qui ne servent qu'à brouiller la lecture.
Malgré tout ça, en allant jusqu'à la fin de la fonction, le vrai payload finit par apparaître en clair dans Ghidra.
<img width="1171" height="311" alt="Payload visible en fin de fonction" src="https://github.com/user-attachments/assets/56f1932f-2f88-48fd-bb92-2975f16697b8" />

Étape 4 — Corriger l'endianness (le sens des octets)
C'est là qu'il faut faire attention à un détail important. L'émulateur Android tourne sur une architecture ARM en Little Endian : ça veut dire que les entiers 64 bits sont stockés en mémoire avec les octets dans l'ordre inverse.
Les valeurs brutes extraites de Ghidra sont donc à lire à l'envers, par paires d'octets :
Valeur Ghidra (Big Endian)Valeur corrigée (Little Endian)0x1549170f1311081d1d0811130f1749150x15131d5a1903000d0d0003195a1d13150x14130817005a0e08080e5a0017081314
Ce qui donne le payload chiffré final :
1d0811130f1749150d0003195a1d1315080e5a0017081314

Étape 5 — Déchiffrer le flag avec CyberChef
On a maintenant tout ce qu'il faut : le payload chiffré et la clé XOR. Direction CyberChef pour le déchiffrement statique, sans même avoir à lancer l'application.
Les opérations à enchaîner dans CyberChef :

From Hex — convertir le payload hexadécimal en bytes bruts
XOR — appliquer un XOR avec la clé UTF-8 pizzapizzapizzapizzapizz

Et le flag tombe :
making owasp great again
<img width="1600" height="742" alt="Déchiffrement dans CyberChef" src="https://github.com/user-attachments/assets/d1407fd7-40b2-40d8-b0e0-60205f580b1e" />

Étape 6 — Valider dans l'application
On saisit le flag trouvé directement dans l'application pour confirmer que l'analyse était correcte.
<img width="420" height="848" alt="Validation du flag dans l'app" src="https://github.com/user-attachments/assets/0c044da5-be0c-4560-b58b-d6b3444ec075" />
