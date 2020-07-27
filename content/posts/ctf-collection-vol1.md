---
title: "Write-up | CTF Collection Vol.1"
date: 2020-07-08T01:43:30+02:00
draft: false
tags:
  - writeup
  - tryhackme
---

Ce write-up sera consacré à la room CTF collection Vol.1, qui est sur [Try Hack Me](https://tryhackme.org).
Cette box contient un certain nombre d'exercices dédiés aux débutants. Let's go !

Synopsis :
> Just another random CTF room created by me. Well, the main objective of the room is to test your CTF skills. For your information, vol.1 consists of 20 tasks and all the challenges are extremely easy. Stay calm and Capture the flag. :)

## What does the base said?
Dans cet exo, il faut simplement décoder une chaine de caractère qui semble être encodée en base64.

```
$ echo "VEhNe2p1NTdfZDNjMGQzXzdoM19iNDUzfQ==" | base64 -d
THM{[edited]}
```

## Meta meta
Ici, il faut juste regarder les métadonnées de l'image à télécharger, je ne vais pas plus détailler :)

## Mon, are we going to be okay?
Petit challenge de stégano, on trouve le contenu de l'image avec `steghide`:
```
$ steghide extract -sf Extinction.jpg 
Entrez la passphrase: 
Écriture des donnÉes extraites dans "Final_message.txt".
$ cat Final_message.txt 
It going to be over soon. Sleep my child.

THM{[edited]}
```
(C'est le chall le plus déprimant que j'ai fait de ma vie... ).

## Erm......Magick
Cette tâche se résoud directement en regardant son code source sur le navigateur, pas besoin d'en dire plus ;)

## QRrrrr
Ici, un simple QR code à télécharger et à scanner avec son téléphone.

## Reverse it or read it?
Dans cet exercice, on récupère un fichier binaire appelé `hello.hello`. Il existe sous Linux une commande permettant d'extraire les chaines de caractères lisibles d'un binaire. Cette commande est `strings`. Ainsi, on peut faire :
```
$ strings hello.hello | grep THM
THM{[edited]}
```

## Another decoding stuff
Ici, nous avons seulement une chaine de caractère codée.
Je la passe dans `hash-identifier`, mais aucun format de hash n'est reconnu. C'est probablement un codage sur une base différente du base64. Un petit tour sur [CyberChef](https://gchq.github.io/CyberChef/) et on découvre que c'est du base58.

## Left or right
L'énoncé nous donne `MAF{atbe_max_vtxltk}`. Ce n'est probablement pas du ROT13.
Je vais sur [dcode.fr](https://www.dcode.fr) afin de trouver le décalage du code César. On trouve ainsi le flag.

## Make a comment
Aucun document ou string n'est donné. Le chall s'appellant Make a comment, je regarde le code source et... Oh bah tiens :)

## Can you fix it?
On voit en regardant le header du fichier .png à télécharger que les 8 premiers octets ne sont pas les bons. Avec `hexeditor`, on peut les modifier pour qu'ils correspondent à `89 50 4e 47 0d 0a 1a 0a`. L'image est maintenant lisible est contient le flag.

## Read it
>  Some hidden flag inside Tryhackme social account.

Un exo d'OSINT ? Chouette ! Il va falloir aller stalker les réseaux sociaux de TryHackMe !
Le flag se trouve sur reddit assez facilement !

## Spin my head
Le format de la chaine de caractères donnée est du brainfuck. [dcode.fr](https://www.dcode.fr) a tous les outils nécessaires pour la décoder et obtenir le flag.

## An exclusive!
Pour résoudre celui-là, il faut faire un XOR entre `44585d6b2368737c65252166234f20626d` et  `1010101010101010101010101010101010`. Je suis passé par [xor.pw](https://wor.pw) car il permet d'afficher la sortie en ASCII.

## Binary Walk
Après avoir récupéré l'image, on utilise `binwalk` pour voir si il y a du contenu caché à l'intérieur :
```
$ binwalk hell.jpg 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.02
30            0x1E            TIFF image data, big-endian, offset of first image directory: 8
265845        0x40E75         Zip archive data, at least v2.0 to extract, uncompressed size: 69, name: hello_there.txt
266099        0x40F73         End of Zip archive, footer length: 22
```
On voit que c'est le cas, on peut alors utiliser le flag `--extract` pour extraire `hello_there.txt`, puis le lire :
```
$ cat _hell.jpg.extracted/hello_there.txt 
Thank you for extracting me, you are the best!

THM{[edited]}
```

## Darkness
L'aide du challenge conseille d'utiliser `stegsolve`, outil que je n'ai pas.
Il est possible de le récupérer comme ça :
```
wget http://www.caesum.com/handbook/Stegsolve.jar -O stegsolve.jar
chmod +x stegsolve.jar
```
puis de l'utiliser comme ceci : `java -jar stegsolve.jar`.
En jouant un peu avec les options, on voit le flag apparaitre.

## A sounding QR
Un QR à scanner donne un lien vers soundcloud. En écoutant le fichier, on comprends le flag. Il est possible de l'enregistrer et de l'écouter plus lentement ;)

## Dig up the past
Celui-là est sympa, il faut utiliser la Wayback Machine.

## Uncrackable !
On dirait du chiffrement par décalage; ayant déjà eu du code César, je pencherai cette fois-ci pour du chiffre de Vigenère.
Un petit tour sur [guballa.de](https://www.guballa.de/vigenere-solver) et le tour est joué.

## Small bases
Pour trouver le flag de cet exo, il faut convertir la string de la base décimale à l'hexa, puis de l'hexa à l'ASCII

## Read the packet
Ici j'ai ouvert la capture avec `Wireshark`, et j'ai cherché la string contant 'THM'.

### Le mot de la fin

Voilà cette room terminée ! Elle aura été assez longue. Même si elle n'a pas été aussi sympa qu'un CTF plus classique, elle m'a permit de revoir quelques bases et quelques outils que je n'ai pas l'habitude d'utiliser.
