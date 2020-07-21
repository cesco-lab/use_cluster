# use_cluster

a repo to collect information about using cluster

Prérequis pour utiliser un cluster

On y accède en ssh, sous linux ou mac cela se fait simplement en tapant la ligne suivant dans la console :

```{bash}
ssh identifiant@nomdedomaineducluster.fr 
```

Le nom de domaine du cluster est propre à chaque cluster, normalement il vous
est communiqué lorsque vous demandez un accès au compte (pour le cluster de
l’UPMC par exemple c’est mesu.dsi.upmc.fr).Si vous êtes sous windows, un client
ssh est nécessaire, Duch utilise mobaxterm (https://mobaxterm.mobatek.net/),
yves utilise Smartty qui a une interface “advanced” sympa pour naviguer dans les
dossier, uploader et downloader des fichiers ou des dossiers et vous épargne
d’utiliser trop de commandes. Une fois téléchargé et installé, vous l’ouvrez et
l’utilisez comme une console linux classique.

Ca vaut aussi le coup d’avoir quelques bases en commandes Linux. Exemples
“cp machin truc” pour copier machin à l’emplacement truc
“mkdir dossier” pour créer un dossier “ls” pour avoir la liste des fichiers (on
peut ajouter des arguments pour en savoir plus par exemple “ls -lisart” pour
avoir le détail des fichiers ordonnées du plus ancien au plus récent)
“find” pour chercher quelque chose ...

Pour le reste Google n’est pas votre ami mais il peut quand même pas mal aider
(et il y a un kit de survie dispo ici
https://www.commentcamarche.net/faq/8386-kit-de-survie-linux pour savoir comment
se déplacer dans une arborescence en ligne de commande linux/unix)


# Le cluster de l’UPMC


Pour y avoir accès, il faut demander un accès par formulaire papier,
téléchargeable ici : http://hpcave.upmc.fr/index.php/usage/open-an-account/

En utilisant la commande donnée en intro pour se connecter, on atterrit dans son
dossier personnel. Comme dit en intro, on peut ensuite créer des dossier,
fichiers, modifier ses fichiers et naviguer entre les dossier en utilisant les
lignes de commande classique du langage bash.  qq exemples ici : Aide-mémoire —
Commandes et scripts Bash 1 Commandes de gestion des fichiers et répertoires.

Ensuite, pour exécuter un script R, il nous faut deux choses, le script R en
question et un script shell, qui va permettre de lancer le script R. Si le
script R utilise des données, il faut bien évidemment les données. Pour passer
le script R et le script shell (et les données) de votre ordinateur sur le
cluster, il faut : 

1. ouvrir une nouvelle console pour linux et mac, ou un
nouveau terminal sous moba pour windows

2. utiliser la chaîne de commandes suivantes :

```{bash}
> cd “path_to_the_directory_containing_your_file_on_your_computer” 
> scp you_file_name.r identifiant@mesu.dsi.upmc.fr:/home/cluster_id/yourdirectoryonthecluster/

```

Retournez dans la console (ou le terminal) sur laquelle vous êtes connectés au
cluster, vos fichiers ont dû apparaître. Si vous les avez rédigés sous windows,
n’oubliez pas de les transcrire en UTF-8 en utilisant la commande (dans le
cluster) :

```
> dos2unix your_file_name.r
```


 Ensuite pour envoyer votre job, utilisez la commande suivante :

```
> qsub your_script_shell.sh
```

Pour voir vos jobs :

```
> qstat –u identifiant
```

Pour voir tous les jobs :

```
> qstat
```

Pour supprimer un job :

```
> qdel your_job_id
```

- Exemple de script shell pour le cluster de l’UPMC pour lancer le script R “test-dompi.R”:

```
#PBS -S /bin/bash
#PBS -N your_job_name #nom du job
#PBS -q beta #queue à utiliser, alpha ou beta
#PBS -l select=2:ncpus=24:mpiprocs=24 #utiliser deux lots de 24 coeurs, structurés en 24 coeurs
#PBS -l walltime=72:00:00 #temps maximum d’execution, après quoi le job sera killé
#PBS -j oe

cd $PBS_O_WORKDIR
module load R/3.4.3

mpirun /bin/bash $(which R) --slave -f test-dompi.R

exit 0
```

```{r}
#-------------------------------------------------------------------------------------------------------------------------------------------
#Le script “test-dompi.R” étant parallélisé:
#-------------------------------------------------------------------------------------------------------------------------------------------
library(doMPI)
cl <- startMPIcluster()
registerDoMPI(cl)

x <- foreach(i=1:96, .combine="c") %dopar% {sqrt(i)}
x

closeCluster(cl)
mpi.quit()
#-------------------------------------------------------------------------------------------------------------------------------------------
```

Pour les jobs nécessitant beaucoup de mémoire vive (type stats spatiales), privilégiez la queue alpha, pour les autres la beta. Si vous utilisez la queue alpha, le nombre de cœur (x) se précise de la manière suivante :

```
#PBS -l select=1:ncpus=x
```

Si vous utilisez la queue beta (plus appropriée pour nos usage), le nombre de
cœur (x) est organisé en paquet de 24 cœurs, pour en prendre 48 il faut donc
écrire :

```
#PBS -l select=2:ncpus=24:mpiprocs=24
```

Attention contrairement à sur un ordi, la moindre petite erreur (chargement de
package pas installé sur le cluster..), va arrêter votre job, il faut donc
s’assurer avant que toutes les fonctions et/ou packages utilisés sont bien sur
le cluster et nettoyer un maximum vos scripts (enlever les bouts qui ne servent
plus ou qui ne sont plus utiles) !!  Si vos scripts ne sont pas parallélisés,
lancez les en utilisant un seul coeur.


# Cluster PCIA du MNHN :

ATTENTION, ce cluster n’est accessible que sur le réseau MNHN donc il faut que
vous ayez un accès VPN ou TeamViewer ou autre Autre inconvénient, si vous êtes
inactif pendant plus de 10 minutes, vous vous faites déconnecté.es. Il faut donc
relancer la connexion à chaque fois qu’on s’y remet.
Par contre, a priori, c’est le seul cluster où les scripts qui utilisent les
packages spatiaux de R (sp, raster, rgdal & co) fonctionnent bien !

Envoyer un mail à pcia@mnhn.fr pour demander un compte et Faycal Allouti 

Il vous répond qu’il a créé un espace /trinity/home/dupont avec un exemple de
script R à lancer avec un fichier qui vous montre comment soumettre un script R
avec la commande sbatch exemple.slurm

Pour accéder à cette espace il faut avoir installer un logiciel de “client SSH”
(cf. plus haut, perso j’utilise SmarTTY, en interface “advanced”, la copie de
fichiers de et vers le cluster y est très simple avec l’onglet “SCP”). Le nom de
domaine à indiquer est pciaclusterlogin.mnhn.fr et votre login et mot de passe
vous aura été donné par mail.

Vous pouvez uploader vos fichiers de données sur votre espace personnel s’ils ne
sont pas trop gros (<1 Go ?). Pour les gros fichiers, préférez l’espace de
stockage /mnt/beegfs/dupont (si votre login est “dupont”). Je vous conseille de
créer un raccourci de cette espace de stockage sur votre espace perso (= là où
vous atterrissez quand vous vous connectez). Pour cela taper la commande ln -s
/mnt/beegfs/XXX STOCKAGE
(avec XXX=votre login qu’on vous a donné)

Pour celleux qui ont besoin d’utiliser les packages spatiaux de R (rgdal, sp,
sf, raster, etc), il faut ajouter les lignes module load gdal/2.2.2 et module
load udunits/2.2.26 sous la ligne module load biology dans vos fichiers .slurm
Une fois toutes ces étapes faites, vous pouvez copier tous vos scripts R
préférés et créer des fichiers .slurm (sur le modèle d’exemple.slurm) et juste
éditer la dernière ligne par un Rscript MonScriptPrefere.R
Pour lancer le script il faut juste taper sbatch Machin.slurm, un numéro de
“job” s’affiche, et vous trouverez aussi les logs (contenu de la console de R)
qui s’écrivent au fur et à mesure dans un fichier slurm-XXXX.out (avec XXXX le
numéro du job). Utile pour voir ce qui se passe bien ou mal… On peut aussi voir
si nos scripts sont encore en train de tourner en tapant la commande squeue -u
dupont (avec dupont son propre login)

Dans un fichier .slurm, on peut aussi ajouter des arguments externes (avec la
commande R commandArgs), enchainer plusieurs scripts, etc Dites-moi si besoin
d’aide
S’il manque un package, il faut demander son install à pcia@mnhn.fr mais ils
sont assez réactifs (a priori la plupart des packages courants sont déjà
installés)

## Commandes basiques

## Exemples de script


# Cluster GENOUEST :

Ce tutoriel reprend en partie les informations disponibles sur le site de
genouest et sur la page https://www.genouest.org/howto/#cluster. Cette page
fournit également des informations sur les commandes que vous pouvez utiliser
pour lancer les scripts.  Pour installer vos propres outils / librairies R sur
le cluster, VOUS POUVEZ le faire, si le package R par exemple est sous conda.
Pour l’utiliser, voir la section https://www.genouest.org/howto/#conda 
Pour toute question :  support@genouest.org
 
## 1. Créer un compte genouest
 
Membres du CESCO : ouvrir un compte genouest accessible avec un login et mot de
passe (les mêmes que ceux utilisés pour se connecter au cloud du labo) Aller sur
https://my.genouest.org/manager/#/register
en précisant
'cesco' dans la rubrique 'Laboratory'
'Julliard Romain' dans la rubrique 'Manager'
'umr7204' dans la rubrique 'Team'
 
et en utilisant votre email et votre adresse professionnelle.
Section 'Why do you need an account' : coller 'Need to share documents (scripts,
drafts, datasets,...) with colleagues.' OU autre chose si plus pertinent
(notamment pour du calcul).  Vous avez alors accès à
https://data-access.cesgo.org sur lequel vous trouverez un espace vous
permettant de deposer vos fichiers et dossiers et de les partager
 
 
## 2. Installation des logiciels pour utiliser le cluster
 
Le logiciel SSH : Putty (un peu obsolète, préférer Mobaxterm -> tuto générique
mobaxterm https://www.marseo.fr/se-connecter-en-ssh-avec-mobaxterm/ et procédure
plus bas)
 
### Première connexion

Si besoin : tutoriel video genouest: https://www.youtube.com/watch?v=N3BzP7KiOvg
Installer putty depuis https://www.putty.org/ (putty key generator et Pageant sont installé en même temps)
Ouvrir Puttygen (Windows -> Start Menu -> All Programs -> PuTTY -> PuTTYgen).
Cliquer sur Generate
Entrer un mot de passe (Key passphrase) et le confirmer
Cliquer sur «Save public key» et la sauver avec l’extension .pub (exemple «mykey.pub»)
Cliquer sur «Save private key» et la sauver avec l’extension .ppk (exemple «mykey.ppk»)
Enregistrer dans un fichier word la public key (contenu de la fenêtre «Public key for pasting…») et le mdp
Ajouter par copier coller la clé publique (celle du fichier word) à son compte genouest (https://my.genouest.org): onglet SSH, fenêtre «Add public SSH key» et cliquer sur «add»
 
### A chaque connexion au cluster :

- Ouvrir Pageant : cliquer sur Pageant dans le menu démarrer, puis aller
  l’ouvrir dans les icônes cachées à droite de la barre des tâches
- Charger sa clé privé (le fichier avec l’extension .ppk) dans pageant via le
  bouton «add key» et le mot de passe associé (key passphrase de l’étape
  précédente). 
- Fermer Pageant
- Lancer putty
- Dans «putty configuration», onglet Session : rentrer «genossh.genouest.org»
  dans la rubrique «Host Name (or IP adress)» ; rentrer 22 dans la rubrique
  «Port»; cliquer sur « Open » tout en bas.
- Rentrer votre login genouest dans la fenêtre après : «login as»
- Vous pouvez alors rentrer les lignes de commandes pour dialoguer avec le
  cluster après « -bash- **numero de version**$ »
 
#### Avec MobaXterm

- Installer MobaXterm (https://mobaxterm.mobatek.net/download.html)
- Dans l’onglet Tools, cliquer sur MobaKeyGen
- Cliquer sur Generate
- Entrer un mot de passe (Key passphrase) et le confirmer
- Cliquer sur «Save public key» et la sauver avec l’extension .pub (exemple
  «mykey.pub»)
- Cliquer sur «Save private key» et la sauver avec l’extension .ppk (exemple
  «mykey.ppk»)
- Enregistrer dans un fichier word la public key (contenu de la fenêtre «Public
  key for pasting…») et le mdp
- Ajouter par copier coller la clé publique (celle du fichier word) à son compte
  genouest (https://my.genouest.org): onglet SSH, fenêtre «Add public SSH key»
  et cliquer sur «add»
- Dans l’onglet Sessions, cliquer sur New session
-  Cliquer sur l’onglet SSH
- Dans Remote host, rentrer “genossh.genouest.org”
- Cocher Specify username et rentrer votre login genouest
- Dans l’onglet Advanced SSH settings, cocher Use private key et sélectionner le
  fichier correspondant à la clé privée (celui avec l’extension .ppk)
- Cliquer sur OK pour valider les modifications de la session et lancer la
  session
- Le cluster vous demande le mot de passe (passphrase) associé à votre clé pour
  valider votre connexion

#### Avec Linux (non testé):

- Création d'une clé ssh, en local dans un terminal : ssh-keygen -t rsa -b 4096
- Cette commande vous demandera un mot de passe (qui permet de protéger votre
  clé ssh, également appelé passphrase). Elle va créer deux fichiers :
  $HOME/.ssh/id_rsa et $HOME/.ssh/id_rsa.pub
- id_rsa est votre clé privée et ne doit en aucun cas être divulguée. id_rsa.pub
  est votre clé publique. Elle doit être copiée et collée sur
  https://my.genouest.org/ (après s'être connecté, dans l'onglet "Add public SSH
  key", ne pas oublier de cliquer sur le bouton Add)
- Cette clé doit aussi être rentrée dans votre agent SSH via la commande :
  ssh-add $HOME/.ssh/id_rsa
- Ensuite vous pouvez vous connecter, à partir d'un terminal en utilisant cette
  commande : $ssh <your-login>@genossh.genouest.org
 
 
### Le logiciel FTP (pour envoyer des grosses données en étant sûr de ne pas le corrompre) : Filezilla :

- Installer Filezilla.
- Dans l’onglet « fichier », menu« Gestionnaires de sites », rentrer les informations suivantes:
  – host: ftps://gridftp.genouest.org
  – port: 990
  – login: [votre login-genouest]
  – password: [votre mot de passe-genouest]
- Puis cliquer sur “connexion” et “ok”.
- La fenêtre de gauche représente ce qu’il y a sur votre pc ; la fenêtre de
  droite ce qu’il y a sur le serveur de genouest et votre espace personnel. Pour
  accéder directement à son espace personnel coller
  /home/genouest/mnhn_cesco/***votre login*** dans la fenêtre « Site distant »
- Vous pouvez maintenant faire passer des fichiers de votre PC à votre espace
  perso en les faisant glisser les fichiers d’une fenêtre à l’autre. Les
  résultats des analyses réalisées sur le cluster sortiront dans votre espace
  perso.
-  3. Utilisation du cluster, avec création de son propre environnement avec ces
   librairies R par exemple

- Exemple de l’utilisation de Genouest pour utiliser R et des packages
  rgdal/sp/raster :
- Vidéo 1 https://youtu.be/hUCrKIyOrSY pour se connecter sur GenOuest en ssh via
  mobaxterm, puis se connecter à un noeud de calcul en interactif, et y créer un
  environnement conda avec R et des packages spatiaux, puis activer cet
  environnement pour pouvoir se rendre dans R.

```
ssh pnom@genossh.genouest.org #connexion ssh sur GenOuest puis :
srun --pty bash # pour se connecter sur un noeud de calcul
. /local/env/envconda.sh # pour créer un environnement conda dédié à votre analyse (donc avec vos packages R préférés, ainsi, à chaque fois que vous souhaiterez utiliser ces packages, vous aurez “juste” à activer cet environnement avant d’exécuter R et de charge vos librairies.
# puis création de votre environnement (pour savoir si votre package R préféré est sous conda, vous pouvez taper sous Gogol conda + nom du package et voir le nom. Attention, la version de package R dans conda peut-être un peu plus vieille que celle dispo directement sous R.
conda create -p ~/my_env_r_spatial r-sp r-raster r-rgdal
# une fois l'environnement créé, vous pourrez réutiliser cet environnement R avec ces packages dispo en faisant :
conda activate ~/my_env_r_spatial
R # pour entrer dans R
#et voilà, vous devez voir R avec les 3 librairies prêtes à être utilisées ;)
```

# Cluster MIGALE de l’INRAE :

L’INRAE met à disposition un cluster de calcul sur sa plateforme MIGALE
(https://migale.inrae.fr/). 

Pour avoir accès il faut remplir un formulaire numérique
(https://migale.inrae.fr/ask-account)  et attendre la validation (assez rapide
de souvenir) Ils avaient une FAQ intéressante mais ils changent de site et je ne
retrouve plus les pages utiles…

En tout cas avant le confinement ils avaient une réactivité assez importante
pour aider et résoudre des problèmes.

Il y a quand même une vidéo pour se connecter via MoBaXTerm
(https://migale.inra.fr/faq).

Après le fonctionnement est similaire aux infos données plus haut pour le
cluster de l’UPMC (script R + script shell, conversion avec dos2unix si script
écrit sur windows, commande qsub etc…), pas de surprise.


