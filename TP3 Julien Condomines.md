
# TP 3 Utilisateurs, groupes et permissions Julien CONDOMINES

## Exercice 1. Gestion des utilisateurs et des groupes

#### 1. Utilisez la commande groupadd pour créer deux groupes dev et infra
On créer donc les deux groupes avec les commandes `groupadd dev` et `groupadd infra`.

#### 2. Créez ensuite 4 utilisateurs alice, bob, charlie, dave avec la commande useradd, en demandant la création de leur dossier personnel et avec bash pour shell
Pour créer un utilisateur, avec un dossier personnel et un bash pour le shell il faut utiliser la commande suivante :

> `sudo useradd alice -m --shell /bin/bas`

On va donc la reproduire quatre fois pour chacun des utilisateurs. 

#### 3. Ajoutez les utilisateurs dans les groupes créés : - alice, bob, dave dans dev - bob, charlie, dave dans infra

Pour ajouter un utilisateur existant à un groupe existant il faut utiliser la commande :

> `usermod -a -G dev alice`
> 
#### 4. Donnez deux moyens d’afficher les membres de infra
On va tout d'abord afficher grâce au chemin et a la commande grep qui récupère va récupérer la valeur "groupe"

> `cat etc/group | grep "infra"`

On peut aussi installer le packet _members_ pour afficher les membres du groupe passé en argument :

> `members "infra"`

#### 5. Faites de dev le groupe propriétaire des répertoires /home/alice et /home/bob et de infra le groupe propriétaire de /home/charlie et /home/dave

On va utiliser ici la commande chgrp pour changer le propriétaires des groupes :

> `chgrp dev /home/alice /home/bob`

#### 6. Remplacez le groupe primaire des utilisateurs : - dev pour alice et bob - infra pour charlie et dave.

On va utiliser la commande usermod qui permet de modifier le groupe primaire d’un utilisateur

> `usermod dev -g alice`

#### 7. Créez deux répertoires /home/dev et /home/infra pour le contenu commun aux membres de chaque groupe, et mettez en place les permissions leur permettant d’écrire dans ces dossiers.

On va tout d'abord créer les répertoires à l'aide de la commande `mkdir` :

> ` mkdir /home/dev`

Ensuite, on change les droits du dossier avec la commande `chmod` :

> `chmod 770 /home/dev`

#### 8. Comment faire pour que, dans ces dossiers, seul le propriétaire d’un fichier ait le droit de renommer ou supprimer ce fichier ?

Afin de s'assurer que seulement le propriétaire d'un fichier dans ces dossiers puisse le renommer ou le supprimer, il faut ajouter une notion que l'on appel "sticky bit". C'est un bit particulier que l'on ajoute avec `chmod +t /home/dev` ou alors de façon octale avec la commande `chmod 1770 /home/dev`.

#### 9. Pouvez-vous ouvrir une session en tant que alice ? Pourquoi ?

Dans l'état actuel, il est impossible de se connecter en temp que alice car sa session est désactivé. Si l'on souhaite l'activer, il va falloir lui définir un mot de passe. 

#### 10. Activez le compte de l’utilisateur alice et vérifiez que vous pouvez désormais vous connecter avec son compte.

On va donc tout d'abord définir un mot de passe pour la session de alice avec la commande `passwd` :
> `sudo passwd alice`

Cela a activé la session de alice, on peut donc ensuite si connecter :
```bash
User@localhost:~$ sudo passwd alice
New password:
Retype new password:
passwd: password updated successfully
User@localhost:~$ su alice
Password:
alice@localhost:/home/User$
```
#### 11. Comment obtenir l’uid et le gid de alice ?
Afin d'obtenir l'uid d'un utilisateur, il faut utiliser la commande `id` :
```bash
alice@localhost:/home/User$ id
uid=1002(alice) gid=1002(dev) groups=1002(dev)
```

#### 12. Quelle commande permet de retrouver l’utilisateur dont l’uid est 1003 ?

Pour retrouver l'utilisateur qui a l'uid 1003, il faut utiliser cette commande :
```bash
alice@localhost:/home/User$ cat /etc/passwd | grep 1003
bob:x:1003:1002::/home/bob:/bin/bash
```
#### 13. Quel est l’id du groupe dev ?

Afin de retrouver l'id du groupe dev, on va utiliser cette commande :
```bash
alice@localhost:/home/User$ cat /etc/group | grep dev
plugdev:x:46:User
dev:x:1002:alice,bob,dave
```
On peut donc voir que l'id du groupe dev est 1002.

#### 14. Quel groupe a pour gid 1002 ? (Rien n’empêche d’avoir un groupe dont le nom serait 1002...)

Afin de rechercher quel groupe a le gid 1002, il va falloir utiliser la commande 
```bash 
alice@localhost:/home/User$ grep 1002 /etc/group | awk -F '1002' '{print $1}'
dev:x:
```
La commande `awk` a permis de "scanner" le dossier group afin de trouver lequel comportait le gid '1002'.

#### 15. Retirez l’utilisateur charlie du groupe infra. Que se passe-t-il ? Expliquez.
Pour retirer charlie du groupe infra, il nous faut utiliser la commande `gpasswd` :
```bash
User@localhost:~$ sudo gpasswd --delete charlie infra
Removing user charlie from group infra
```
Ainsi, maintenant que charlie ne possède plus de droit dans le groupe infra, il lui est impossible de s'y rendre. 
```bash
charlie@localhost:/home/User$ cd infra
bash: cd: infra: Permission denied
```
#### 16. Modifiez le compte de dave de sorte que : — il expire au 1 er juin 2021 — il faut changer de mot de passe avant 90 jours — il faut attendre 5 jours pour modifier un mot de passe — l’utilisateur est averti 14 jours avant l’expiration de son mot de passe — le compte sera bloqué 30 jours après expiration du mot de passe.

Pour modifier les paramètres d'un compte, on peut tout d'abord les changer en utilisant la commande `usermod` suivi de l'attribut de l'on souhaite changer ainsi que de la nouvelle valeur souhaité :
```bash
User@localhost:~$ sudo usermod --expiredate 2021-06-01 dave
User@localhost:~$ sudo chage -l dave
Last password change                                    : Sep 19, 2022
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Jun 01, 2021
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```
Cependant, on peut aussi utiliser la commande `chage` qui va nous permettre de changer un par un chaque paramètres à l'aide d'un interface graphique:
```bash
User@localhost:~$ sudo chage dave
Changing the aging information for dave
Enter the new value, or press ENTER for the default

        Minimum Password Age [0]:
        Maximum Password Age [99999]: 90
        Last Password Change (YYYY-MM-DD) [2022-09-19]:
        Password Expiration Warning [7]: 14
        Password Inactive [-1]: 5
        Account Expiration Date (YYYY-MM-DD) [2021-06-01]:
User@localhost:~$ sudo chage -l dave
Last password change                                    : Sep 19, 2022
Password expires                                        : Dec 18, 2022
Password inactive                                       : Dec 23, 2022
Account expires                                         : Jun 01, 2021
Minimum number of days between password change          : 0
Maximum number of days between password change          : 90
Number of days of warning before password expires       : 14
```
#### 17. Quel est l’interpréteur de commandes (Shell) de l’utilisateur root ?
On va donc vérifier le shell utiliser par l'utilisateur root à l'aide de la commande `grep` :
```bash
User@localhost:~$ cat /etc/passwd | grep root
root:x:0:0:root:/root:/bin/bash
```
On peut donc voir que l'interpréteur utilisé est le bash.

#### 18. Si vous regardez la liste des comptes présents sur la machine, vous verrez qu’il en existe un nommé nobody. A quoi correspond-il ?

Le compte nobody est un compte d'utilisateur à qui aucun fichier n'appartient, qui n'est dans aucun groupe qui a des privilèges et dont les seules possibilités sont celles que tous les "autres utilisateurs" ont.

#### 19. Par défaut, combien de temps la commande sudo conserve-t-elle votre mot de passe en mémoire ? Quelle commande permet de forcer sudo à oublier votre mot de passe.

La commande sudo conserve le mot de passe 15 minutes en mémoire.
Afin de forcer sudo à oublier notre mot de passe, il faut utiliser la commande suivante  `sudo -K`.

## Exercice 2. Gestion des permissions

#### 1. Dans votre $HOME, créez un dossier test, et dans ce dossier un fichier fichier contenant quelques lignes de texte. Quels sont les droits sur test et fichier ?

Le dossier test possèdes les droits suivants: rwxrwxrw-
Le fichier fichier possède lui les droits suivants :  rw-rw-r--

#### 2. Retirez tous les droits sur ce fichier (même pour vous), puis essayez de le modifier et de l’afficher en tant que root. Conclusion ?

```
User@localhost:~/test$ sudo chmod 000 fichier
User@localhost:~/test$ sudo cat fichier
un fichier standard
User@localhost:~/test$ sudo nano fichier
User@localhost:~/test$ sudo cat fichier
un fichier standard modifié
```
On peut donc en conclure que l'utilisateur n'y aura plus accès alors que root lui n'est pas soumis à cette restriction. 

#### 3. Redonnez vous les droits en écriture et exécution sur fichier puis exécutez la commande echo "echo Hello" > fichier. On a vu lors des TP précédents que cette commande remplace le contenu d’un fichier s’il existe déjà. Que peut-on dire au sujet des droits ?

```
User@localhost:~/test$ sudo chmod 300 fichier
User@localhost:~/test$ echo "echo Hello" > fichier
User@localhost:~/test$ sudo cat fichier
echo Hello
```
On peut donc en conclure qu'il faut être très vigilants lorsque l'on définis les droits pour un fichier, si l'on en confère trop à un utilisateur en particulier cela va potentiellement ce transformer en menace. 

#### 4.  Essayez d’exécuter le fichier. Est-ce que cela fonctionne ? Et avec sudo ? Expliquez.

```
User@localhost:~/test$ ./fichier
bash: ./fichier: Permission denied
User@localhost:~/test$ sudo ./fichier
Hello
```
Vus que nous n'avons pas les droits d'exécution il est impossible pour l'utilisateur d'exécuter le fichier ce qui n'est pas le cas de sudo qui lui a tout les droits.

#### 5. Placez vous dans le répertoire test, et retirez vous le droit en lecture pour ce répertoire. Listez le contenu du répertoire, puis exécutez ou affichez le contenu du fichier fichier. Qu’en déduisez vous ? Rétablissez le droit en lecture sur test

```
User@localhost:~/test$ sudo chmod u-r ../test
User@localhost:~/test$ ls
ls: cannot open directory '.': Permission denied
User@localhost:~/test$ ./fichier
bash: ./fichier: Permission denied  
```
On peut en déduire que vus que nous n'avons plus les droits pour ce répertoire il nous est impossible de lire ou bien d'exécuter notre fichier fichier.

#### 6. Créez dans test un fichier nouveau ainsi qu’un répertoire sstest. Retirez au fichier nouveau et au répertoire test le droit en écriture. Tentez de modifier le fichier nouveau. Rétablissez ensuite le droit en écriture au répertoire test. Tentez de modifier le fichier nouveau, puis de le supprimer. Que pouvez vous déduire de toutes ces manipulations ?

```bash
User@localhost:~/test$ nano nouveau
User@localhost:~/test$ mkdir sstest
User@localhost:~/test$ chmod u-w nouveau
User@localhost:~/test$ chmod u-w ../test
User@localhost:~/test$ nano nouveau
[ Error writing nouveau: Permission denied ]
User@localhost:~/test$ chmod u+w ../test
User@localhost:~/test$ nano nouveau
[ Error writing nouveau: Permission denied ] 
User@localhost:~/test$ rm nouveau
rm: remove write-protected regular file 'nouveau'? yes
User@localhost:~/test$ ls
fichier  sstest
```
On peut donc en déduire que les fichiers ont leurs propres droits à eux, ils ne dépendent pas des droits des répertoires auxquelles ils appartiennent.

#### 7. Positionnez vous dans votre répertoire personnel, puis retirez le droit en exécution du répertoire test. Tentez de créer, supprimer, ou modifier un fichier dans le répertoire test, de vous y déplacer, d’en lister le contenu, etc…Qu’en déduisez vous quant au sens du droit en exécution pour les répertoires ?

```bash
User@localhost:~/test$ chmod u-x test
User@localhost:~/test$ nano test/nouveau
[ Path 'test' is not accessible ]
User@localhost:~/test$ cd test/
-bash: cd: test/: Permission denied
User@localhost:~/test$ ls test
ls: cannot access 'test/sstest': Permission denied
ls: cannot access 'test/fichier': Permission denied
```
Si nous ne possédons pas le droit d'exécution sur un dossier, nous ne pouvons rien faire avec notre fichier, le directory agit comme parent de tous les fichier. Quand bien même nos droits de fichier n'ont pas été modifiés, le faite que nous ayons modifier nos droits de dossiers restreints nos actions. 

#### 8. Rétablissez le droit en exécution du répertoire test. Positionnez vous dans ce répertoire et retirez lui à nouveau le droit d’exécution. Essayez de créer, supprimer et modifier un fichier dans le répertoire test, de vous déplacer dans ssrep, de lister son contenu. Qu’en concluez vous quant à l’influence des droits que l’on possède sur le répertoire courant ? Peut-on retourner dans le répertoire parent avec ”cd ..” ? Pouvez-vous donner une explication ?
```bash
User@localhost:~/test$ chmod u-x ../test
User@localhost:~/test$ nano nouveau
[ Path '.': Permission denied ]
User@localhost:~/test$ cd sstest
-bash: cd: sstest: Permission denied
User@localhost:~/test$ ls
ls: cannot open directory '.': Permission denied
User@localhost:~/test$ cd ..
User@localhost:~$ 
```
Les droits du répertoire courant on la même valeur que ceux du répertoire parent, ceux ci n'ont pas d'influence particulière. 

```bash
User@localhost:~/test$ cd ..
User@localhost:~$ 
```

Il est possible de faire un cd ce qui est logique puisqu'il faut forcément que nous puissions sortir de notre dossier. 

#### 9. Rétablissez le droit en exécution du répertoire test. Attribuez au fichier fichier les droits suffisants pour qu’une autre personne de votre groupe puisse y accéder en lecture, mais pas en écriture.

```bash
User@localhost:~/test$ chmod 650 fichier
User@localhost:~/test$ ll
total 16
drwxr-xr-x  3 User localhost 4096 oct.   1 12:30 ./
drwxr-xr-x 10 User localhost 4096 oct.   1 11:56 ../
-rw-r-x---  1 User localhost 11 oct.   1 12:12 fichier*
drwxrwxr-x  2 User localhost 4096 oct.   1 12:21 sstest/
```
#### 10. Définissez un umask très restrictif qui interdit à quiconque à part vous l’accès en lecture ou en écriture, ainsi que la traversée de vos répertoires. Testez sur un nouveau fichier et un nouveau répertoire.

```bash
User@localhost:~$ touch fichier
User@localhost:~$ mkdir folder
User@localhost:~$ umask 177
```
#### 11. Définissez un umask très permissif qui autorise tout le monde à lire vos fichiers et traverser vos répertoires, mais n’autorise que vous à écrire. Testez sur un nouveau fichier et un nouveau répertoire.

```bash
User@localhost:~$ umask 011
```

#### 12. Définissez un umask équilibré qui vous autorise un accès complet et autorise un accès en lecture aux membres de votre groupe. Testez sur un nouveau fichier et un nouveau répertoire.

```bash
serveur@serveur:~$ umask 037
```

#### 13. Transcrivez les commandes suivantes de la notation classique à la notation octale ou vice-versa (vous pourrez vous aider de la commande stat pour valider vos réponses) : - chmod u=rx,g=wx,o=r fic - chmod uo+w,g-rx fic en sachant que les droits initiaux de fic sont r--r-x--- - chmod 653 fic en sachant que les droits initiaux de fic sont 711 - chmod u+x,g=w,o-r fic en sachant que les droits initiaux de fic sont r--r-x---

-   chmod u=rx,g=wx,o=r fic  
    

Quand on fait = , les droits se changent  
Quand on fait +, les droits s'ajoutent  
Quand on fait -, les droits se soustraient  

Réponse :  `532`  

-   chmod uo+w,g-rx fic en sachant que les droits initiaux de fic sont r--r-x---  
    
Réponse :  `404`

-   chmod 653 fic en sachant que les droits initiaux de fic sont 711  
 
Réponse :  `653`  car ne prend pas en compte les droits initiaux
    
-   chmod u+x,g=w,o-r fic en sachant que les droits initiaux de fic sont r--r-x---  
    
Réponse :  `520`

#### 14. Affichez les droits sur le programme passwd. Que remarquez-vous ? En affichant les droits du fichier /etc/passwd, pouvez-vous justifier les permissions sur le programme passwd ?

`-rw-r--r-- 1 root root 1840 sept. 21 18:07 /etc/passwd`
Nous remarquons que tous le monde peut lire le fichier mais uniquement root a les droits pour écrire dans le fichier.
Seul l'utilisateur root peut ajouter des utilisateurs, ce qui parait cohérent avec les droits présent sur le fichier.  
Tout le monde peut connaître les différents utilisateurs et groupes du système.


