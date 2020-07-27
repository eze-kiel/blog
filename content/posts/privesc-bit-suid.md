---
title: "PrivEsc | Abuser du bit SUID"
date: 2020-07-07T13:47:51+02:00
draft: false
tags:
  - privesc
  - linux
---

Même si ce n'est pas très fréquent dans la vie courante, le bit SUID est un vecteur d'élévation de privilèges assez fréquent dans les CTF.

## Bit SUID

Lorsque des fichiers exécutables ont le bit SUID de configuré, ils sont lancés en tant que l'user qui possède le fichier, et non en tant que celui qui lance le programme. Dans l'idée, c'est assez pratique. Sauf que ça présente un défaut majeur : la possibilité de pouvoir utiliser des programmes en tant que root si celui-ci est le propriétaire de ceux-ci.

Afin de savoir quels fichiers ont le bit SUID de configuré, il y a la commande `find / -perm /4000 2> /dev/null`.

Cette commande cherche des fichiers à la racine ayant pour permission n'importe lequel des bits 4000.

Enfin (et c'est question de goût), je redirige la sortie standard vers `/dev/null`, afin de ne voir que ce qui est intéresant pour moi.

## Élévation de privilèges

Énormément de binaires permettent, si leur bit SUID est configuré, d'exécuter des commandes en tant que root ou de lire un flag (dans le cas d'un CTF). Il y a une liste assez importante avec des examples sur [GTFOBins](https://gtfobins.github.io/), mais ce n'est pas parce qu'un programme n'y est pas qu'il n'est pas exploitable !

Le cas typique que l'on retourve en CTF, c'est pour lire le dernier flag, traditionnellement à `/root/root.txt`.

Prenons l'exemple ou `/bin/sed` a le bit SUID de positionné.

Pour lire le flag, il suffirait donc d'exécuter les commandes suivantes :
```
$ LFILE=/root/root.txt
$ /bin/sed -e '' "$LFILE"
```
Ici, nous plaçons le fichier à lire (`/root/root.txt`) dans une variable d'environement `LFILE`.

Ensuite on appelle `sed` avec le flag `-e` qui permet d'exécuter un script. Sauf que nous ne lui donnons aucun script à exécuter, il retournera donc le fichier tel quel. Et voilà !

Comme dit plus haut, une liste non-exhaustive de binaires vulnérables est disponible sur [GTFOBins](https://gtfobins.github.io/). Ce site est une perle pour l'élévation de privilèges. À garder sous le coude donc !