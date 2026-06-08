---
title: "SAÉ 2.03 — Administration système et virtualisation"
subtitle: "Rapport final : Installation Debian, Git et Forgejo"
author:
  - GEORGE Mylan
  - POUGHEON Enzo
  - RINGARD Mathéo
date: "2025–2026"
lang: fr
toc: true
toc-depth: 3
number-sections: true
colorlinks: true
linkcolor: blue
urlcolor: blue
toccolor: blue
highlight-style: tango

# Extensions Pandoc activées :
# - listes de tâches GitHub (gfm)
# - blocs spéciaux (fenced_divs) pour les notes/avertissements
# - math inline et display (tex_math_dollars)
# - notes de bas de page (footnotes)
# - tableaux avec pipe (pipe_tables)
# - smart (guillemets typographiques, tirets)
---

# Réponses SAÉ 2.03

Ce document compile les réponses détaillées aux questions posées lors des séances de la SAÉ 2.03, portant sur l'administration système sous Debian, la virtualisation, la gestion de version avec Git, et le déploiement d'un service web (Forgejo). Il rend également compte des difficultés rencontrées au fil des semaines et des solutions mises en œuvre.

---

## Installation et configuration de Debian sous VirtualBox

### Architecture 64 bits

L'appellation "Debian 64-bit" signifie que le système d'exploitation est compilé pour une architecture 64 bits (amd64), par opposition à l'ancienne architecture 32 bits (i386). Cette distinction permet au système de gérer des espaces mémoire supérieurs à 4 Go de RAM, ce qui est le standard sur tous les processeurs modernes.

> **Source :** [lemagit — définition 64 bits](https://www.lemagit.fr/definition/64-bits)

![Création de la VM Debian 64 bits dans VirtualBox](1.png)

### Configuration réseau et DHCP

La configuration réseau par défaut de Debian utilise le protocole **DHCP** (*Dynamic Host Configuration Protocol*), qui attribue automatiquement une adresse IP et des paramètres réseau à la machine lors de son démarrage.

Les paramètres obtenus sur la VM sont les suivants :

- Mode réseau VirtualBox : **NAT** (*Network Address Translation*)
- Adresse IPv4 obtenue : `10.0.2.15/24` (vérifiable via `ip a`)
- Adresse de la passerelle (gateway) : `10.0.2.2`

### Prérequis matériels (RAM)

D'après la documentation officielle Debian :

- **Avec interface graphique** : 2 Go (2048 Mo) de RAM recommandés.
- **Sans bureau (mode console)** : 256 Mo de RAM suffisent si la partition *swap* est active, mais le système sera lent car il s'appuiera sur le disque pour compenser le manque de mémoire vive.

> **Source :** [Debian — Prérequis matériels](https://www.debian.org/releases/stable/amd64/ch03s04.fr.html)

![Récapitulatif de la configuration matérielle (RAM 2048 Mo, disque 20 Go)](5.png)

---

## Concepts et services

### Virtualisation et fichiers

Un **fichier ISO** est une image disque, c'est-à-dire une copie conforme d'un support optique (CD/DVD), utilisée ici comme support d'installation.

![Chargement de l'installateur Debian depuis l'image ISO](4.png)

**MATE** et **GNOME** sont des *environnements de bureau* : ils fournissent l'interface graphique du système (fenêtres, icônes, barre des tâches).

![Bureau MATE de Debian après installation complète](fin.png)

### Serveurs et proxy

- **Serveur web** : logiciel qui répond aux requêtes HTTP/HTTPS pour fournir des ressources web. Source : [MDN Web Docs](https://developer.mozilla.org/fr/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_web_server)
- **Proxy (serveur mandataire)** : composant intermédiaire entre un client et un serveur, utilisé pour filtrer, contrôler ou anonymiser les échanges. Source : [Wikipedia — Proxy](https://fr.wikipedia.org/wiki/Proxy)

### Accès sécurisé (SSH)

Le serveur SSH installé est **OpenBSD Secure Shell server**, vérifiable avec `systemctl status ssh`. En cas d'impossibilité de connexion, la cause la plus fréquente est l'absence de la clef hôte dans le fichier `known_hosts` du client.

---

## Administration et gestion des droits

![Interface de connexion TTY (console texte)](7.jpg)

### Gestion des groupes

La commande `groups user` liste l'ensemble des groupes auxquels appartient un utilisateur, et donc ses droits sur le système.

### Différence entre `su` et `sudo`

- `su` : ouvre une session complète en tant que *root* (demande le mot de passe root).
- `sudo` : exécute une unique commande avec les droits administrateur (demande le mot de passe de l'utilisateur courant). Méthode recommandée car plus sûre et traçable.

---

## Système, noyau et projet Debian

### Noyau et VirtualBox Guest Additions

- **Version du noyau** obtenue via `uname -r` : `6.12.69+deb13-amd64` (noyau 6.12.69, Debian 13, architecture 64 bits).
- **Suppléments invités (Guest Additions)** : pilotes améliorant l'intégration VM/hôte (résolution automatique, presse-papier partagé, dossiers partagés).
- **La commande `mount`** : permet d'attacher un système de fichiers à un point de montage. Utilisée ici pour monter l'ISO des Guest Additions et lancer `VBoxLinuxAdditions.run`.

### Le projet Debian : versions et cycle de vie

Le projet Debian est une communauté internationale créant un système d'exploitation entièrement libre. Les noms de code des versions sont issus du film *Toy Story*.

- Versions maintenues simultanément : **Stable** (Trixie), **Testing** (Forky), **Unstable** (Sid).
- Trixie supporte 6 architectures matérielles.
- Dernier nom de code annoncé (2026) : **Macaroni**.

Cycle de support :

- Mises à jour de sécurité : durée de vie de la version stable (~3 ans).
- **LTS** : prolonge le support à 5 ans.
- **ELTS** : prolonge la maintenance jusqu'à 10 ans.

> **Source :** [Debian releases](https://www.debian.org/releases/) et [Debian LTS](https://wiki.debian.org/fr/LTS)

---

## Automatisation de l'installation (Preseed & ISO)

Toute modification structurelle de l'installation automatique s'effectue dans le fichier `preseed-fr.cfg`.

![Édition du fichier preseed-fr.cfg dans GNU Emacs](8.jpg)

![Modification de l'image ISO avec la commande sed](2.png)

![Paramètres avancés d'édition de l'ISO](3.png)

### Configuration logicielle (Preseed)

- **`pkgsel/include`** : installe des paquets individuels (ex. : `git`, `sudo`).
- **`tasksel`** : installe des groupes cohérents de paquets (ex. : environnement de bureau, serveur SSH).

Le fichier preseed est sensible car il peut contenir des hashs de mots de passe : il doit être hébergé sur un réseau sécurisé pendant le déploiement.

### Modifications apportées

Dans la section `pkgsel` et `passwd` du fichier `preseed-fr.cfg` :

- Ajout de `sudo` dans `pkgsel/include`.
- Ajout de `sudo` dans `passwd/user-default-groups` pour garantir un accès administrateur dès le premier démarrage.

---

## Gestion de version avec Git

### Concepts théoriques

- **Git vs. plateformes (GitHub, GitLab…)** : Git est le moteur local de versioning ; GitHub/GitLab sont des hébergeurs distants offrant des services collaboratifs.
- **Dépôt local** : toutes les données et l'historique résident dans le répertoire caché `.git`.
- **Commit** : instantané immuable de l'état du projet.
- **Branche** : ligne de développement parallèle et indépendante.
- **Tag** : marqueur nommé sur un commit (ex. : `v1.0`).

### Architecture réseau

Git communique via `SSH` (port 22), `HTTPS` (port 443) ou le protocole natif `git` (port 9418). L'authentification SSH nécessite une paire de clés (`ssh-keygen`) et l'enregistrement de la clé publique sur le serveur.

### Pratique et collaboration

- **`.gitignore`** : fichier listant les éléments à exclure du suivi (ex. : `*.class` pour Java, `*.pdf` généré).
- **Pull Request** : mécanisme de revue de code avant fusion dans la branche principale.
- **`git blame`** : identifie l'auteur et le commit associé à chaque ligne d'un fichier.

### Outils graphiques

- **`gitk`** : visualisation de l'historique des commits.
- **`git-gui`** : interface de gestion des commits.
- La ligne de commande (CLI) reste plus précise pour les scripts ; les interfaces graphiques facilitent la résolution visuelle des conflits de fusion.

---

## Installation et configuration de Forgejo

### Préliminaire : redirection de port

Avant d'installer Forgejo, nous avons dû configurer une **redirection de port** dans VirtualBox pour rendre le service accessible depuis la machine hôte.

**Problème rencontré :** En mode NAT (configuration par défaut), la VM n'est pas directement joignable depuis l'extérieur. Il faut donc rediriger le port 3000 de la machine physique vers le port 3000 de la VM.

**Solution :** Via l'interface de VirtualBox → Configuration de la VM → Réseau → Adapter 1 → Avancé → Redirection de ports, nous avons ajouté une règle :

| Nom   | Protocole | IP hôte | Port hôte | IP invité | Port invité |
|-------|-----------|---------|-----------|-----------|-------------|
| forgejo | TCP    |         | 3000      |           | 3000        |

> **Référence :** [Documentation VirtualBox — Networking](https://www.virtualbox.org/manual/UserManual.html#networkingdetails)

::: {.callout-warning}
**Important :** Le type de carte réseau ne doit pas être modifié. Seules des règles de redirection de port sont ajoutées en mode NAT.
:::

Une fois Forgejo installé, l'accès se fait via un navigateur sur la machine hôte à l'adresse `http://localhost:3000`, sans avoir besoin d'ouvrir la fenêtre VirtualBox.

### À propos de Forgejo

**Qu'est-ce que Forgejo ?**
Forgejo est une forge logicielle légère, auto-hébergeable et libre, permettant de gérer des dépôts Git, des tickets, des revues de code et des pipelines CI/CD via une interface web. Source : [forgejo.org](https://forgejo.org)

**Qu'est-ce qu'un fork ?**
En développement logiciel, un fork est la création d'un nouveau projet à partir du code source d'un projet existant, afin de le faire évoluer de manière indépendante. Le projet d'origine continue généralement à exister en parallèle.

**Chaîne de forks de Forgejo :**

1. **Gogs** (2014) : première forge Git légère en Go, encore maintenue aujourd'hui.
2. **Gitea** (2016) : fork de Gogs pour accélérer le développement communautaire, toujours actif.
3. **Forgejo** (2022) : fork de Gitea, créé suite à des inquiétudes concernant la gouvernance de Gitea après son rachat par une entreprise commerciale (Lunny/Gitea Ltd).

**Différence de gouvernance :**
Forgejo est gouverné par une organisation indépendante à but non lucratif, avec un processus de décision communautaire transparent. Gitea est désormais contrôlé par une entité commerciale, ce qui a suscité des craintes sur l'avenir du logiciel libre.

> **Sources :** [Forgejo — About](https://forgejo.org/about/), [Wikipedia — Forgejo](https://en.wikipedia.org/wiki/Forgejo)

### Installation du binaire

Nous avons suivi la méthode officielle d'installation par binaire pré-compilé, conformément à la documentation : <https://forgejo.org/docs/next/admin/installation/binary>

**Problème rencontré :** La documentation officielle est en anglais et certaines étapes (création de l'utilisateur système `git`, permissions sur `/etc/forgejo`) ne sont pas explicitement liées entre elles. Nous avons pris soin de suivre chaque étape dans l'ordre et de vérifier les permissions à chaque fois.

Les grandes étapes suivies :

1. Téléchargement du binaire Forgejo pour `linux-amd64` et vérification de son intégrité (somme SHA256).
2. Placement du binaire dans `/usr/local/bin/forgejo` et attribution des droits d'exécution.
3. Création d'un utilisateur système dédié `git` (sans accès shell) pour faire tourner le service.
4. Création des répertoires de données (`/var/lib/forgejo`) et de configuration (`/etc/forgejo`).
5. Démarrage du service via le navigateur sur `http://localhost:3000` pour l'initialisation.
6. Création du compte administrateur : nom `forgejo`, email `git@localhost`.
7. Protection des fichiers de configuration avec les bonnes permissions.

### Architecture et protocoles réseau

**Pourquoi le port 3000 et non le port 80 ?**
Le port 80 est un port privilégié (< 1024) : sous Linux, seul le processus `root` peut y écouter directement. Forgejo s'exécutant sous l'utilisateur `git` (non-root) pour des raisons de sécurité, il utilise par défaut le port 3000, accessible sans privilèges particuliers. Pour exposer Forgejo sur le port 80, il faudrait utiliser un proxy inverse (nginx, Apache) ou attribuer la capacité `CAP_NET_BIND_SERVICE` au binaire.

> **Source :** [Documentation Linux — Ports privilegiés](https://man7.org/linux/man-pages/man7/ip.7.html)

### Base de données

**Pourquoi SQLite plutôt que PostgreSQL ou MySQL/MariaDB ?**
SQLite est un moteur de base de données embarqué, sans serveur à installer ni à configurer. Pour un usage pédagogique sur une seule machine avec peu d'utilisateurs simultanés, SQLite est amplement suffisant et bien plus simple à mettre en place. PostgreSQL ou MariaDB ne se justifient qu'en production avec de nombreux utilisateurs concurrents.

**Localisation du fichier SQLite de Forgejo :**
Le fichier de base de données se trouve dans `/var/lib/forgejo/data/forgejo.db`. Son propriétaire est l'utilisateur `git` (utilisateur système créé pour Forgejo), ce qui garantit que seul ce service y a accès en lecture/écriture.

### Gestion du service systemd

**Qu'est-ce qu'un service systemd ?**
Systemd est le gestionnaire de services d'initialisation des systèmes Linux modernes (dont Debian). Un service systemd est un processus géré par systemd : il peut être démarré, arrêté, relancé automatiquement au démarrage, et ses logs sont centralisés dans le journal système.

**Fichier de service Forgejo :**
Le fichier est situé dans `/etc/systemd/system/forgejo.service`. Ses sections principales :

- `[Unit]` : description du service et déclaration des dépendances (ex. : réseau disponible, MySQL si utilisé).
- `[Service]` : utilisateur d'exécution (`User=git`), répertoire de travail, commande de démarrage (`ExecStart`), comportement en cas de crash (`Restart=always`).
- `[Install]` : détermine à quel niveau de démarrage le service est activé (`WantedBy=multi-user.target`).

**Différence entre les commandes systemctl :**

- `systemctl start forgejo` : démarre le service immédiatement (non persistant après redémarrage).
- `systemctl enable forgejo` : configure le démarrage automatique du service à chaque boot (sans le démarrer immédiatement).
- `systemctl restart forgejo` : arrête puis redémarre le service (utile après modification de la configuration).

**Visualisation des logs avec journalctl :**

```bash
journalctl -u forgejo -f
```

L'option `-u forgejo` filtre les logs du service Forgejo, et `-f` affiche les nouveaux messages en temps réel (équivalent de `tail -f`).

### Sécurité

**Permissions de `/etc/forgejo` et `app.ini` :**
Le répertoire `/etc/forgejo` doit appartenir à `root:git` avec les permissions `770`, et le fichier `app.ini` à `forgejo:git` avec `640`. Ces restrictions garantissent que seul le service Forgejo (utilisateur `git`) peut lire la configuration, qui peut contenir des secrets (clés secrètes, identifiants SMTP…).

**Port SSH de Forgejo :**
Forgejo intègre son propre serveur SSH sur le port **22** (ou un port alternatif configuré dans `app.ini`, souvent **2222** pour éviter les conflits avec le SSH système). Ce choix permet aux utilisateurs de pousser leurs dépôts via SSH sans configuration supplémentaire côté client.

**Mesures de sécurité supplémentaires pour une exposition Internet :**

- Utiliser un proxy inverse (nginx/Apache) avec un certificat TLS (Let's Encrypt).
- Activer `fail2ban` pour bloquer les attaques par force brute.
- Désactiver l'inscription publique si le serveur est privé.
- Maintenir Forgejo à jour régulièrement.
- Sauvegarder régulièrement le répertoire `/var/lib/forgejo`.

### Tests d'utilisation

**Difficulté rencontrée avec la redirection de port :** Lors des premiers tests, l'accès à `http://localhost:3000` depuis la machine hôte échouait. Après vérification, la règle de redirection de port avait été ajoutée alors que la VM était démarrée ; nous avons dû relancer la VM pour que la règle soit prise en compte.

**Projets créés sur le serveur Forgejo :**

- Un dépôt de test créé directement depuis l'interface web (projet public).
- Le dépôt de ce rapport SAÉ (projet privé, accessible uniquement aux membres de l'équipe).
- Un dépôt de sources Java partagé entre les membres de l'équipe.

**Tests de droits utilisateurs :**

| Rôle       | Peut lire | Peut pousser | Peut gérer |
|------------|-----------|--------------|------------|
| Owner      | ✓         | ✓            | ✓          |
| Collaborateur (Write) | ✓ | ✓       | ✗          |
| Lecteur (Read)        | ✓ | ✗       | ✗          |
| Non-membre (public)   | ✓ | ✗       | ✗          |
| Non-membre (privé)    | ✗ | ✗       | ✗          |

Nous avons vérifié que chaque membre de l'équipe pouvait cloner, modifier et pousser des commits sur les dépôts partagés, et qu'un utilisateur en lecture seule ne pouvait pas pousser (erreur retournée par le serveur).

**Tests de clés SSH :**
Chaque membre a ajouté sa clé publique SSH (`~/.ssh/id_ed25519.pub`) dans son profil Forgejo (Paramètres → Clés SSH). Les clones et push en SSH ont été testés avec succès.

**Création d'une issue et liaison à un commit :**
Une issue de test a été créée dans l'interface web. En incluant `Fixes #1` dans le message d'un commit, l'issue a été automatiquement fermée lors du push, démontrant le lien entre gestion de tickets et historique Git.

---

## Pour aller plus loin : Intégration et livraison continues (CI/CD)

### Concepts CI/CD

- **Intégration continue (CI)** : pratique consistant à fusionner fréquemment les modifications de code dans une branche principale, en déclenchant automatiquement des tests pour détecter les régressions rapidement.
- **Livraison continue (CD)** : extension de la CI, permettant de déployer automatiquement les nouvelles versions validées vers un environnement de production ou de test.

**Avantages en équipe :**

- Détection précoce des conflits d'intégration.
- Réduction des erreurs humaines lors du déploiement.
- Meilleure traçabilité des changements.
- Gain de temps sur les tâches répétitives (compilation, tests, génération du rapport).

### Forgejo Actions

Forgejo intègre **Forgejo Actions**, un système de workflows CI/CD compatible avec la syntaxe GitHub Actions. Nous avons créé un workflow simple permettant de compiler automatiquement ce rapport Markdown en PDF à chaque push sur la branche principale.

Fichier `.forgejo/workflows/rapport.yml` :

```yaml
on:
  push:
    branches:
      - main

jobs:
  build-rapport:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Installer Pandoc
        run: apt-get update && apt-get install -y pandoc texlive-xetex
      - name: Compiler le rapport
        run: pandoc Rapport.md -o Rapport.pdf --pdf-engine=xelatex
      - name: Sauvegarder l'artefact
        uses: actions/upload-artifact@v3
        with:
          name: rapport-pdf
          path: Rapport.pdf
```

Ce workflow illustre comment la CI/CD peut automatiser la génération du rapport à chaque modification, garantissant que la version PDF est toujours à jour avec les sources Markdown.
