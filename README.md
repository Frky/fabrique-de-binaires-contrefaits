## Fabrique de binaires contrefaits (de l’innocence à la malveillance)

Peut-on faire confiance à des programmes binaires ? L’objectif de ce projet est de permettre l’ajout automatisé de code (mailveillant) dans un programme binaire quelconque. Par exemple, l’exécution de la commande grep “patchée” pourra engendrer une communication avec un serveur distant (à l’insu de l’utilisateur). 

Les différentes étapes de ce projet sont :
1. La compréhension et la récupération (parsing) d’un binaire au format ELF x86-64
1. Écriture d’une preuve de concept fonctionnelle : ajout de code (malveillant) dans la section NOTE du binaire & modification du point d'entrée du programme
1. Écriture de code (malveillant) en assembleur à injecter dans le binaire initial
1. Bonus #1: ajout dans la section CODE (avec mise à jour des offsets)
1. Bonus #2: implémentation d’un packer complet de type UPX (avec 2/ ou 3/)

Compétences mises en oeuvre :

* Langage C
* Langage assembleur (Intel x86-64)
* Pour le bonus #2 : compression et cryptographie

Équipe de 3 ou 4 personnes.

Références : 
* http://www.skyfree.org/linux/references/ELF_Format.pdf
* http://virus.enemy.org/virus-writing-HOWTO/_html/segment.padding.html
* http://virus.enemy.org/virus-writing-HOWTO/_html/additional.cs.html
* ...
