---
title: "Tips | Ressources utiles"
date: 2020-07-07T14:53:01+02:00
draft: false
tags:
  - tips
  - linux
  - pentest
  - infosec
---

Je vais faire dans ce post une liste des ressources que j'utilise souvent lors des CTF ou dans des projets de dev, et qui font gagner un temps précieux. Il sera mis à jour assez régulièrement.

## Crypto & co
[CrackStation](https://crackstation.net/) - Casser des hash en ligne. Bien évidemment il ne trouve pas tout, mais c'est un bon début pour trouver du 'password' hashé en md5.

[CyberChef](https://gchq.github.io/CyberChef/) - Plateforme en ligne qui permet de coder/décoder des hash ou des strings chiffrées. Le gros + est que les fonctions de chiffrement/déchiffrement sont sous la forme de blocs, que l'on peut mettre à la suite. C'est donc extrêment facile de retrouver un message qui a été transformé avec du ROT13, puis du base32, puis traduit en hexadécimal par exemple.

[dCode](https://www.dcode.fr/) - Une grande liste de solveurs, notamment César et chiffre de Vigenère.

[hashCat's hash modes list](https://hashcat.net/wiki/doku.php?id=example_hashes) - Celui-là fait gagner du temps quand il est question d'utiliser hashcat.

## Dev
[For The Badge](https://forthebadge.com) - Pour faire joli sur GitHub.

[GitHunt](https://kamranahmed.info/githunt/) - Découvrir les repo GitHub qui sont en tendance.

[Go Report Card](https://goreportcard.com/) - Pour les gophers, un site d'analyse de qualité de code.

[Go Web Examples](https://gowebexamples.com/) - Pour se mettre au web en Golang.

[Markdown Tables Generator](https://www.tablesgenerator.com/markdown_tables) - Tout est dans le titre.

[shields.io](https://shields.io/) - Dans le même esprit que For The Badge.

## Linux
[crontab guru](https://crontab.guru/) - Éditeur routines cron. Plus d'erreurs possibles !

[explainshell.com](https://explainshell.com/) - Site web qui décompose des commandes afin de les expliquer.

## Priv Esc
[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md) - Une grande liste de vercteurs d'élévation de privilèges. Le repo de base contient bien plus de documents sur d'autres techniques ou outils (AD...).

## Reverse shell
[HighOn.Coffee](https://highon.coffee/blog/reverse-shell-cheat-sheet/) - Cheat sheet de nombreux reverse shells en plusieurs langages.

[PentestMonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) - Une autre cheat sheet de revshell.
