# TP#2: Déploiement de HETIC-tac-toe 

Ce TP cible les étudiants de l'école HETIC pour le cours d'infrastructure.

Dans ce TP, vous allez:

- Déployer une application web constitué d'un `frontend` et d'un `web service` (comme `backend`)
- Configurer `Nginx` pour à la fois servir le `frontend` et *reverse-proxy* les requêtes vers un web service


## Étape 1: Récupérez le code d'HETIC-tac-toe

**Objectif**: Récupérer le code de l'application HETIC-tac-toe pour pouvoir le déployer dans l'étape d'après.

- Récupére le code en clonant le répertoire `git` hébergé sur [`Github`](https://github.com/ffauchille/hetic-tac-toe):

```sh
# Depuis votre poste de travail
git clone git@github.com:ffauchille/hetic-tac-toe.git
# Si vous avez une erreur, essayez en https:
git clone https://github.com/ffauchille/hetic-tac-toe.git
```

Vous devriez avoir récupérer 2 dossiers: `frontend` et `backend` sur votre poste de travail.

## Étape 2: Build de l'application

**Objectif**: Packager le frontend et le backend de l'application avant de déployer l'application sur le serveur.

### Build du frontend

- En pré-requis, vous devez avoir `NodeJS` d'installé sur votre poste de travail
  - (Conseillé): installez [`nvm`](https://github.com/nvm-sh/nvm) sur votre poste de travail puis utilisez la commande `nvm use` depuis le dossier `frontend/` pour être sur la version correcte de NodeJS (spécifié par le fichier `frontend/.nvmrc`)
  - Autre moyen: installez la version de `NodeJS` manuellement (version spécifiée dans le fichier `frontend/.nvmrc`)

- Changez le fichier `frontend/public/env-config.json` pour configurer les appels au web service correctement (remplacez `localhost:8000` par l'adresse correspondante à votre identifiant d'étudiant):

```json
{
    "apiDomain": "api.hetic-tac-toe.<etudiant>.floless.fr"
}
```

- Adaptez la configuration `Nginx` du frontend avec votre identifiant d'étudiant:
  - renomez `frontend/hetic-tac-toe.etudiant.floless.fr.conf` en utilisant votre identifiant d'étudiant (e.g. pour `etudiant42` => `hetic-tac-toe.etudiant42.floless.fr.conf`)
  - remplacez `<etudiant>` à la ligne `server_name hetic-tac-toe.<etudiant>.floless.fr;` par votre identifiant d'étudiant

- Vous pouvez packager le frontend via:

```sh
# Déplacement vers le dossier frontend/
cd frontend/
# Si vous avez nvm, assurez vous que vous utilisez la bonne version
# Regardez bien la sortie de la commande. Il est possible que vous deviez
# installer la version de node du projet via nvm install 19
nvm use
# Installation des node_modules
npm install
# Packaging de l'application frontend
npm run build
```

- Assurez-vous qu'un dossier `frontend/dist` a bien été créé sur votre poste de travail
- le contenu du dossier `frontend/dist` devrait ressembler à:

```plain
dist
├── assets
│   └── index-fba14db4.js
├── env-config.json
├── index.html
└── logo.svg

2 directories, 4 files
```

> Note: le suffix `fba14db4` est susceptible d'être différent sur vos poste de travails

### Build du web service (backend)

Pour ce qui est de notre web service, adaptez les configurations suivantes:

- Adaptez la configuration nginx `backend/api.hetic-tac-toe.<etudiant>.floless.fr.conf` avec votre idenifiant d'étudiant:
    - renomez le fichier correctement (e.g en remplaçant `<etudiant>` par votre identifiant d'étudiant)
    - remplacez `<etudiant>` à la ligne `server_name api.hetic-tac-toe.<etudiant>.floless.fr;` par votre identifiant d'étudiant

## Etape 3: Déploiement l'application

**Objectif**: Envoyer les fichiers de code *build* de l'application HETIC-tac-toe et lancer le web service (backend)

### Envoie des fichiers de code

Vous avez utilisé le protocole `ssh` pour vous connecter à votre serveur distant. `scp` est la commande qui permet d'envoyer des fichiers vers votre serveur distant via `ssh` (en remplaçant `<etudiant>` par votre identifiant d'étudiant).

- Depuis le répertoire d'HETIC-tac-toe:

```sh
# envoi des fichiers build du frontend
scp -i <chemin_vers_votre_cle_ssh_privee> -r frontend/dist ubuntu@<etdudiant>.floless.fr:
# envoi de la configuration nginx du frontend
scp -i <chemin_vers_votre_cle_ssh_privee> -r frontend/hetic-tac-toe.<etudiant>.floless.fr.conf ubuntu@<etdudiant>.floless.fr:
# envoi des fichiers source du backend
scp -i <chemin_vers_votre_cle_ssh_privee> -r backend ubuntu@<etdudiant>.floless.fr:
```

Assurez-vous de bien avoir les fichiers sur votre serveur distant:

- connectez-vous via `ssh` à votre serveur distant (comme au TP précédent)

```sh
ssh -i <chemin_vers_votre_cle_ssh_privee> ubuntu@<etudiant>.floless.fr
```

### Installer le web service

Le code du backend est écrit en `python` et parce que l'installation de certains module `python` nécessitent d'être compilés, nous allons installer les dépendances directement sur le serveur distant.

- Connectez vous en `ssh` à votre serveur distant
- Assurez-vous d'avoir la bonne version de `python`

```sh
python3 -V
> Python 3.10.12
```

- Créez un et activez un nouvel environment `python` avec le nom `hetic-tac-toe-venv` pour gérer les dépendances plus simplement:

```sh
# création du virtual env
python3 -m venv hetic-tac-toe-venv

# activation du virtual env
source hetic-tac-toe-venv/bin/activate
```

- Installez les dépendances du backend

```sh
# le dossier backend doit se trouver à /home/ubuntu/backend
cd ~/backend
pip install -r requirements.txt
```

### Lancer le web service

`systemd` est le programme par défault sur linux qui vous permets de lancer des processus en mode *`daemon`*. Le processus persistera lorsque vous quitterez vos session `ssh` et sera relancer automatiquement si la machine doit redémarrer

- Copiez la description du `service` `hetic-tac-toe-backend.service` depuis le répertoire `backend/` vers `/etc/systemd/system/hetic-tac-toe-backend.service`

```sh
# copie du fichier de description du service dans le dossier
# de configuration de systemd
sudo cp ~/backend/hetic-tac-toe-backend.service /etc/systemd/system/
```

- Lancez le web service 

```sh
# lancement du backend d'HETIC-tac-toe en utilisant le service
sudo service hetic-tac-toe-backend start

# Verification que le service est bien actif
sudo service hetic-tac-toe-backend status
> 
> hetic-tac-toe-backend.service - Hetic tac toe web service
> ...
>     Active: active (running) since ...
```


## Étape 4: Configuration du serveur web (Nginx)

- Copiez le dossier `dist` contenant le frontend d'HETIC-tac-toe vers le dossier `/var/www` (un dossier ou nginx a le droit de lecture par défaut):

```sh
# Déplacement du dossier dist vers /var/www
sudo mv ~/dist /var/www/
```

- Copiez les deux fichiers de configuration (`backend` et `frontend`) dans le dossier de configuration d'NGinx. Remplacez `<etudiant>` par votre identifiant d'étudiant

```sh
# copie du fichier de configuration nginx pour exposer le backend via api.<etudiant>.floless.fr
sudo cp ~/backend/api.hetic-tac-toe.<etudiant>.floless.fr.conf /etc/nginx/conf.d/
# copie du fichier de configuration nginx pour exposer le frontend via <etudiant>.floless.fr
sudo cp ~/hetic-tac-toe.<etudiant>.floless.fr.conf /etc/nginx/conf.d/
# rafraîchissement de la nouvelle configuration de nginx
sudo service nginx reload
```

Désormais, vous devriez avoir accès à HETIC-tac-toe servie par votre serveur distant.

Visitez l'adresse `http://hetic-tac-toe.<etudiant>.floless.fr` (en remplaçant `<etudiant>` par votre identifiant d'étudiant).

## Étape 5 (bonus): Configuration SSL (HTTPS)

Cette étape est optionnelle.

**Objectif**: configurer la connexion HTTPS sur HETIC-tac-toe.

- [`certbot`](https://certbot.eff.org) est une application CLI qui vous permets de gérer vos certificats SSL 

- installez `certbot` via `snapd` (déjà installé sur vos machines):

```sh
sudo snap install --classic certbot
```

- Installez les certificats pour les 2 noms de domaines:

```sh
sudo certbot --nginx
```

Plusieurs prompts vont venir vous poser des questions:

1. Premier prompt: entrez une addresse email qui sera référencé dans le certificat. Elle ne doit pas forcément être valide. Mettez `<etudiant>@floless.fr` (en remplaçant `<etudiant>` par votre identifiant d'étudiant)
1. Second prompt: tapez `Y` puis entrée
1. Troisième prompt: tapez entrée directement (cela selectionnera tout vos domaines)

Après cela, vous devriez pouvoir accéder à votre application en https.

Félicitations!
