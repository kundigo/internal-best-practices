# internal-best-practices
==================================

## Git et outils associés
==================================

Ce document n'a pas pour but d'expliquer le principe de fonctinnement de git. Si vous avez des lacunes sur le sujet, je vous invite à aller voir un des nombreux tuto en ligne ([par exemple celui d'open classroom](https://openclassrooms.com/courses/gerez-vos-codes-source-avec-git)).

### Installation de Git
----------------------------------

L'installation est très simple, vous trouverez toutes les informations nécessaires ici: [documentation officielle](https://git-scm.com/book/fr/v1/D%C3%A9marrage-rapide-Installation-de-Git).

### Installation des outils associés
------------------------------------

Il n'y a qu'un seul outil à installer en plus de git pour vous simplifier ENORMEMENT la vie: un programme de comparaison de fichiers.
Nous utiliserons ici [Meld](http://meldmerge.org/), que je vous invite à installer. Son utilisation sera expliqué dans le chapitre de gestion des conflits.


Afin d'associer Meld à git, il vous faudra ajouter la ligne:
```
[merge]
    tool = meld
```
au fichier .git/gitconfig qui se trouve à la racine de votre répertoire home.

### Utilisation de Git
------------------------------------

Dans ce petit guide, nous supposons que vous savez ce que fait Git, ce qu'est une branche, ce qu'est un commit, ce que sont des pull et des pushs, mais que parfois les choses sont flou quant à l'ordre d'utilisation des différentes commandes (mais qu'est ce que ça veut dire que ma branche diverge de master ? au secours je peux pas push !!!).

#### Workflow

Avant tout, il y a deux choses qu'il faut toujours vérifier avant de commencer à changer quoi que ce soit dans votre projet(pour vous éviter des migraines):
- s'assurer qu'on est sur la bonne branche, c'est bête mais ça peut sauver des vies.
- faire des 'git pull' réguliers pour récupérer le travail de vos collègues. Cela vous évitera de nombreux conflits.

Ensuite, il est très important de ne JAMAIS push son travail sur la branche master directement. La branche master doit en permanence avoir un code déployable, c'est à dire qu'elle doit être suffisamment clean pour permettre de lancer l'application en production. C'est le chef de projet qui sera en charge de merge votre code sur master si il estime que ce dernier est conforme à ce qu'on attend.

Classiquement, lorsque vous allez commencez à travailler sur une issue qui vous aura été assignée, la première chose à faire - si cela n'a pas déjà été fait - est de créer une branche spécifique pour travailler sans risque de démolir ce qui existe déjà et fonctionne:
```git branch nom_de_la_branche```

sans oublier de vous déplacer ensuite sur la branche en question:
```git checkout nom_de_la_branche```

Notez qu'il est possible de créer une branche et de se déplacer directement dessus avec la commande:
```git checkout -b nom_de_la_branche```

Si il existe déjà une branche pour l'issue en cours, il faut la récupérer sur le dépot distant:
```git fetch```
suivi d'un
```git checkout nom_de_la_branche```

Si à un moment dans votre travail vous avez besoin de savoir quelles sont les branches présentent, vous pouvez faire un
```git branch``` pour voir les branches locales.
```git branch --all``` pour voir les branches distantes.

Vous avez désormais en local la branche sur laquelle vous allez pouvoir travailler, et tout se passe bien, vous avancer et il est temps de faire une pause. Pour sauvegarder votre travail, voici les étapes à suivre:
Faire un ```git add nom_du_fichier_a_sauvegarder``` (ou plus simplement un ```git add .``` ou un ```git add -A```, cela va sauvegarder l'état du code de tous les fichiers modifiées). Pour plus d'infos sur cette commande, c'est [ici](https://git-scm.com/docs/git-add). 

Il est important de noter que la commande ```git add``` et ses variantes permet simplement de signifier à git que des fichiers ont été modifiés, et que les fichiers ciblés doivent être inclus dans le prochain commit qui sera réalisé sur la branche.
Si vous avez ajouté des fichiers par erreur, il existe la commande ```git reset nom_du_fichier``` ou ```git reset``` pour enlever un ou plusieurs fichiers des fichiers taggués.
Cette commande n'écrit rien dans l'historique de git, et il vous sera très simple d'annuler les changements fait jusqu'ici avec la commande ```git stash```.
Cette commande va faire revenir le code à son état initial tout en gardant en mémoire les modifications, qui pourront être réappliqués par la commande ```git stash apply```. Attention cependant, il n'y a qu'un seul stash actif à la fois. Pour plus d'information je vous invite à aller voir la [doc](https://git-scm.com/book/fr/v1/Utilitaires-Git-Le-remisage).

Pour voir toutes les modifs prise en compte par git: ```git status```

L'étape suivante est donc de faire:
```git commit -m 'ce que je veux raconter dans mon commit'``` pour mettre à jour l'historique des commits de git.
Il est possible de créer un commit 'vide', c'est à dire d'ajouter un commit à l'historique sans qu'aucun fichier n'ait été modifié avec la commande ```git commit --allow-empty -m 'ceci est un commit vide'```, vous ne devriez pas en avoir besoin souvent, mais vous savez désormais que ça existe.

Vous n'êtes absolument pas obliger de partager votre code à chaque commit, mais selon qui sera votre responsable de projet, il est possible qu'on vous demande de ne partager vos changements qu'avec un seul commit enregistré dans l'historique pour l'intégralité de l'issue en cours. Nous allons introduire une commande qui va vous simplifier la vie, mais qui est à utiliser avec parcimonie:
```git rebase -i branche_sur_laquelle_on_veut_rebase```
Pour plus d'informations, vous pouvez vous référer au chapitre [Le cas du rebase](#Le-cas-di-rebase).




#### Le cas du rebase

Le but premier du rebase est de fusionner des branches avec un historique divergent. Ce qui va nous intéresser ici est simplement la possibilité qu'il offre de réécrire l'historique des commits.
Si vous voulez devenir des pros du rebase je vous invite à vous rendre [ici](https://git-scm.com/docs/git-rebase).

Important: Evitez de faire un rebase sur une branche si vous n'êtes pas la seule personne à la modifier si vous ne voulez pas vous faire sauvagement assasiner par vos collègues.

ATTENTION: il est PRIMORDIAL de récupérer la branche d'origine sur laquelle on souhaite rebase AVANT de rebase, sinon vous êtes parti pour des heures de pleurs de et souffrances si quelqu'un à effectué des modifications de la dite branche entre temps.

Je vais prendre un exemple concret pour expliquer les étapes à suivre dans le cas d'utilisation le plus courant (réécrire l'historique).
Vous avez une branche mon_code qui a été créée à partir de la branche master, et vous avez enregistré 5 commits sur cette branche que nous appelerons 'commit 1-5'. Nous sommes actuellement sur la branche mon_dev.
Vous voulez garder 1 seul des 5 commits, et ce commit doit contenir l'intégralité de vos changements (cf le commit 1 doit contenir l'état du commit 5).

1) git checkout master
2) git pull
3) git checkout mon_dev
4) git rebase -i master

Quand vous allez exécuter la commande ```git rebase -i master``` depuis la branche mon_code, vous demandez à git de coller tous vos commit à la suite du dernier commit de master. A ce niveau, on a l'équivalent d'un simple merge, à la différence prêt que le rebase va ouvrir une fenêtre dans votre éditeur de texte par défaut (il est possible de le modifier dans le fichier de config de git, je vous laisse chercher la solution spécifique à votre éditeur).
Cette fenêtre va se présenter sous cette forme:
```
pick fb2f677 commit 1
pick c5069d5 commit 2
pick c9e138f commit 3
pick 733e2ff commit 4
pick 536fsx2 commit 5

```

Ce qu'on veut, c'est remplacer les 'pick' des commits à effacer par des 'fixup'. Si on souhaite réécrire le message de commit du premier commit on peut changer 'pick' par 'reword':
```
reword fb2f677 mon beau message de commit
fixup c5069d5 commit 2
fixup c9e138f commit 3
fixup 733e2ff commit 4
fixup 536fsx2 commit 5

```

Sauvegarder le fichier et ça devrait être bon.

Si ce n'est pas bon, c'est que vous avez rencontré des conflits. Pour la gestion des conflits à proprement parler, allez au chapitre [Les difficultés probables](#Les-difficultés-probables).

Pourquoi un conflit ? si une modification effectué sur master n'est pas compatible avec une de vos modification dans un commit, vous allez avoir des conflits. A chaque étape du rebase (une étape == fusion du master avec un de vos commits), il va falloir gérer les conflits associés. Si vous rencontrer plusieurs fois le même conflit c'est parfaitement normal: supposons que vous ayez modifié le fichier pilou.rb dans votre premier commit. Si ce fichier à été modifié dans le code du master avec des modifications incompatibles, le rebase va, pour chacun de VOS commit, vous demander comment il doit gérer ce conflit (soit gérer 5 fois le même conflit).

Une fois que tous les conflits ont été traité, vous pouvez continuer le rebase avec la commande ```git rebase --continue```.

Si à un moment donné vous êtes perdu et souhaitez annuler le rebase: ```git rebase --abort```.

#### Les difficultés probables

conflits (parler de meld)

branches divergentes

#### Recapitulatifs des commandes communes

Créer une nouvelle branche
```git branch nom_de_la_branche```
```git checkout -b nom_de_la_branche```

Changer de branche:
```git checkout nom_de_la_branche```

Créer + changer de branche:
```git checkout -b nom_de_la_branche```

Récupérer les branches distantes:
```git fetch```

Voir toutes les branches locales:
```git branch```

Voir toutes les branches locales et distantes:
```git branch --all```

Récupérer les commits d'une branche distante:
```git pull```

Sauvegarder son code:
```git add nom_du_fichier_a_sauvegarder```
```git add .```
```git add -A```
```git status```
```git reset fichier```
```git reset```
```git stash```
```git apply```
```git commit -m 'message du commit'```
```git commit --allow-empty -m "message du commit"```

Réécrire l'historique:
```git rebase -i nom_de_la_branche_sur_laquelle_je_veux_rebase```
```git rebase --continue```
```git rebase --abort```
```git commit --amend```

Voir l'historique:
```git log```