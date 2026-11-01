# 💻 Étape 2 — Installation et configuration du Raspberry Pi

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](../en/02-raspberry-installation.md)

[🏠 Retour au sommaire](../../README.fr.md)

---

## 📖 Présentation

Ce deuxième chapitre explique comment installer et configurer **Raspberry Pi OS** sur un **Raspberry Pi Zero 2 W**, afin de préparer son utilisation avec le **Proxmark3 Easy**.

L'installation s'effectue entièrement sans écran, clavier ni souris (*headless*).

Le Raspberry Pi sera administré à distance depuis un ordinateur ou un smartphone grâce à une connexion **Wi-Fi et SSH**.

Nous installerons également **tmux**, un outil permettant de conserver les sessions de terminal actives même lorsque la connexion SSH est interrompue.

### 🎯 Objectifs de cette étape

À la fin de ce chapitre, vous saurez :

- Installer Raspberry Pi OS Lite (64 bits).
- Configurer une carte microSD avec Raspberry Pi Imager.
- Définir le nom du Raspberry Pi et son compte utilisateur.
- Configurer une connexion Wi-Fi.
- Activer et utiliser SSH.
- Mettre à jour le système.
- Vérifier le fonctionnement du réseau.
- Ajouter des réseaux Wi-Fi supplémentaires.
- Installer et utiliser `tmux`.
- Créer la commande `pm` pour retrouver facilement une session persistante.

---

## 1. Configuration de référence

Le tutoriel repose sur la configuration suivante :

| Élément | Configuration |
|---|---|
| Raspberry Pi | Zero 2 W |
| Système | Raspberry Pi OS Lite |
| Architecture | 64 bits |
| Base Debian | Debian 13 (Trixie) |
| Interface graphique | Aucune |
| Gestionnaire réseau | NetworkManager |
| Connexion réseau | Wi-Fi 2,4 GHz |
| Accès distant | SSH |
| Sessions persistantes | tmux |
| Nom de la machine | proxmark |
| Utilisateur d'exemple | piuser |

Cette configuration correspond au système utilisé pour ce projet.

> 💡 **Remarque**
>
> Les commandes présentées dans ce tutoriel utilisent le nom d'utilisateur `piuser` et le nom de machine `proxmark`.
>
> Si vous choisissez d'autres noms lors de l'installation, vous devrez les adapter dans les commandes.

---

## 2. Télécharger Raspberry Pi Imager

### 📥 Présentation

**Raspberry Pi Imager** est l'outil officiel permettant de préparer une carte microSD pour un Raspberry Pi.

Il permet notamment :

- De sélectionner le modèle de Raspberry Pi.
- De télécharger le système d'exploitation.
- De préparer la carte microSD.
- De configurer le Wi-Fi.
- De créer un compte utilisateur.
- D'activer SSH avant le premier démarrage.

### 🛠️ Installation

Télécharger Raspberry Pi Imager depuis le site officiel :

🔗 [Télécharger Raspberry Pi Imager](https://www.raspberrypi.com/software/)

Installer ensuite le logiciel sur votre ordinateur.

Insérer la carte microSD dans le lecteur de cartes.

Une capacité de **16 Go minimum** est recommandée.

---

## 3. Préparer la carte microSD

Lancer **Raspberry Pi Imager**.

### 🔧 3.1 — Choisir le matériel

Dans la liste des appareils, sélectionner :

**Raspberry Pi Zero 2 W**

### 💻 3.2 — Choisir le système d'exploitation

Sélectionner :

**Raspberry Pi OS Lite (64-bit)**

Cette version ne possède pas d'interface graphique.

Elle convient particulièrement à notre projet car elle permet :

- De limiter l'utilisation de la mémoire.
- De réduire la consommation des ressources.
- De fonctionner sans écran.
- D'administrer le système à distance.
- D'exécuter les outils Proxmark3 depuis un terminal.

> 💡 **Conseil**
>
> Il n'est pas nécessaire d'installer une version Desktop de Raspberry Pi OS.
>
> Le client Proxmark3 Iceman fonctionne directement dans un terminal Linux.

### 💾 3.3 — Choisir le stockage

Sélectionner la carte microSD destinée au Raspberry Pi.

> ⚠️ **Attention**
>
> Toutes les données présentes sur la carte sélectionnée seront effacées pendant l'installation.
>
> Vérifier soigneusement le périphérique choisi avant de poursuivre.

---

## 4. Configuration initiale

Avant de lancer l'écriture de la carte microSD, Raspberry Pi Imager permet de personnaliser le système.

Nous allons préparer les paramètres nécessaires à une utilisation sans écran.

### 🖥️ 4.1 — Nom de la machine

Définir le nom :

```text
proxmark
```

Le Raspberry Pi pourra ainsi être accessible sur le réseau local avec le nom :

```text
proxmark.local
```

> 💡 **Remarque**
>
> La résolution de `proxmark.local` repose sur le protocole mDNS.
>
> Son fonctionnement dépend du réseau local et de la compatibilité de l'appareil utilisé.
>
> Si ce nom ne fonctionne pas, nous pourrons utiliser directement l'adresse IP du Raspberry Pi.

### 👤 4.2 — Compte utilisateur

Créer un compte utilisateur et définir un mot de passe sécurisé.

Dans ce tutoriel, nous utiliserons le nom générique :

```text
piuser
```

Vous pouvez choisir un autre nom.

Le mot de passe doit être suffisamment robuste pour protéger l'accès SSH.

> ⚠️ **Sécurité**
>
> Ne laissez pas le compte utilisateur sans mot de passe.
>
> Évitez également les mots de passe simples ou identiques à ceux de vos autres appareils.

### 📶 4.3 — Configuration Wi-Fi

Renseigner les paramètres du réseau :

| Paramètre | Valeur |
|---|---|
| SSID | Nom du réseau Wi-Fi |
| Mot de passe | Mot de passe du réseau |
| Pays | France (FR) |

Le Raspberry Pi Zero 2 W utilise le **Wi-Fi 2,4 GHz**.

Vérifier que votre réseau Wi-Fi est compatible.

> 💡 **Conseil**
>
> Si votre routeur utilise un réseau commun pour les bandes 2,4 GHz et 5 GHz, le Raspberry Pi se connectera sur la bande compatible.
>
> En cas de problème, vérifier que la bande 2,4 GHz est bien activée sur le routeur.

### 🔐 4.4 — Activer SSH

Dans les paramètres de personnalisation de Raspberry Pi Imager :

1. Activer **SSH**.
2. Sélectionner l'authentification par mot de passe pour cette première installation.
3. Vérifier que le compte utilisateur possède un mot de passe sécurisé.
4. Enregistrer les paramètres.

L'authentification par clé SSH pourra être configurée ultérieurement pour renforcer la sécurité.

> 💡 **Pourquoi SSH ?**
>
> SSH permet de contrôler le Raspberry Pi à distance depuis un terminal.
>
> Nous pourrons ainsi installer les logiciels, modifier la configuration et utiliser le Proxmark3 sans connecter d'écran au Raspberry Pi.

---

## 5. Écriture de la carte microSD

Une fois les paramètres configurés :

1. Lancer l'écriture de la carte.
2. Confirmer l'effacement des données.
3. Attendre la fin de l'installation.
4. Laisser Raspberry Pi Imager vérifier les données écrites.
5. Retirer proprement la carte microSD.

La carte est maintenant prête pour le premier démarrage.

---

## 6. Premier démarrage

### 🔌 6.1 — Démarrer le Raspberry Pi

1. Insérer la carte microSD dans le Raspberry Pi Zero 2 W.
2. Vérifier les branchements réalisés au chapitre 1.
3. Connecter l'alimentation.
4. Attendre quelques minutes pendant le premier démarrage.

Le Raspberry Pi doit automatiquement rejoindre le réseau Wi-Fi configuré.

> 💡 **Conseil**
>
> Le premier démarrage peut être plus long que les suivants.
>
> Attendre quelques minutes avant de conclure à un problème de connexion.

### 📶 6.2 — Vérifier la connexion réseau

Depuis l'interface d'administration de votre routeur, vous pouvez vérifier si le Raspberry Pi apparaît parmi les appareils connectés.

Son nom devrait être :

```text
proxmark
```

Il est également possible de relever son adresse IP locale.

Cette adresse sera utile si `proxmark.local` ne fonctionne pas.

---

## 7. Première connexion SSH

### 💻 7.1 — Ouvrir un terminal

Depuis un ordinateur connecté au même réseau local, ouvrir un terminal.

Sous Windows, utiliser **PowerShell**.

Sous Linux ou macOS, utiliser le terminal du système.

### 🔐 7.2 — Se connecter au Raspberry Pi

Exécuter :

```bash
ssh piuser@proxmark.local
```

Remplacer `piuser` par le nom choisi lors de l'installation si nécessaire.

Lors de la première connexion, SSH peut demander de confirmer l'empreinte du serveur.

Après avoir vérifié qu'il s'agit bien de votre Raspberry Pi, confirmer la connexion.

Saisir ensuite le mot de passe du compte utilisateur.

Une fois connecté, le terminal affiche une invite similaire à :

```text
piuser@proxmark:~ $
```

Le Raspberry Pi peut maintenant être administré à distance.

### 🔍 7.3 — En cas d'échec de connexion

Si `proxmark.local` ne fonctionne pas :

1. Vérifier que le Raspberry Pi est allumé.
2. Vérifier qu'il est connecté au Wi-Fi.
3. Rechercher son adresse IP dans l'interface du routeur.
4. Utiliser directement cette adresse pour la connexion SSH.

Exemple :

```bash
ssh piuser@192.168.1.100
```

> 💡 **Remarque**
>
> L'adresse `192.168.1.100` est uniquement un exemple.
>
> Elle doit être remplacée par l'adresse IP réelle du Raspberry Pi.

### ⚠️ Problèmes fréquents

| Problème | Cause possible | Vérification |
|---|---|---|
| `Could not resolve hostname` | Nom mDNS non résolu | Utiliser l'adresse IP |
| `Connection refused` | SSH indisponible | Vérifier l'activation de SSH |
| `Connection timed out` | Raspberry Pi inaccessible | Vérifier le Wi-Fi et l'adresse IP |
| `Permission denied` | Identifiants incorrects | Vérifier utilisateur et mot de passe |

---

## 8. Mise à jour du système

Une fois connecté en SSH, il est recommandé de mettre à jour Raspberry Pi OS.

### 🔄 8.1 — Actualiser la liste des paquets

Exécuter :

```bash
sudo apt update
```

Cette commande actualise la liste des logiciels disponibles dans les dépôts.

### 📦 8.2 — Installer les mises à jour

Exécuter :

```bash
sudo apt full-upgrade -y
```

Cette commande installe les mises à jour disponibles et gère les éventuelles modifications de dépendances.

L'opération peut prendre plusieurs minutes.

### 🔁 8.3 — Redémarrer si nécessaire

Si une mise à jour nécessite un redémarrage :

```bash
sudo reboot
```

La connexion SSH sera interrompue pendant le redémarrage.

Attendre que le Raspberry Pi soit de nouveau disponible avant de se reconnecter.

```bash
ssh piuser@proxmark.local
```

---

## 9. Vérifier l'installation

Nous allons maintenant vérifier que le système est correctement installé et configuré.

### 🔍 9.1 — Vérifier la version du système

Exécuter :

```bash
cat /etc/os-release
```

Cette commande affiche les informations concernant la distribution Linux installée.

La configuration de référence du projet utilise :

```text
Debian GNU/Linux 13 (trixie)
```

### 🔍 9.2 — Vérifier l'architecture

Exécuter :

```bash
uname -m
```

Résultat attendu :

```text
aarch64
```

Ce résultat confirme que le système utilise une architecture ARM 64 bits.

### 📶 9.3 — Vérifier le réseau

Exécuter :

```bash
nmcli device status
```

Cette commande affiche l'état des interfaces réseau gérées par NetworkManager.

L'interface Wi-Fi :

```text
wlan0
```

doit normalement apparaître comme connectée.

### 🔐 9.4 — Vérifier le service SSH

Exécuter :

```bash
systemctl status ssh
```

Le service doit apparaître comme actif :

```text
active (running)
```

Pour quitter l'affichage détaillé du service, appuyer sur :

```text
Q
```

---

## 10. Ajouter un réseau Wi-Fi secondaire

Le Raspberry Pi peut mémoriser plusieurs réseaux Wi-Fi.

Cette fonctionnalité est particulièrement utile pour notre projet, car elle permet d'utiliser le Proxmark3 dans différents environnements.

Par exemple :

- Réseau Wi-Fi du domicile.
- Réseau Wi-Fi du bureau.
- Point d'accès d'un smartphone.
- Autre réseau autorisé.

### 📶 10.1 — Ajouter un réseau

Pour ajouter un réseau Wi-Fi supplémentaire :

```bash
sudo nmcli device wifi connect "NOM_DU_WIFI" --ask
```

Remplacer `NOM_DU_WIFI` par le SSID du réseau.

NetworkManager demandera le mot de passe si nécessaire.

> ⚠️ **Attention**
>
> Cette commande peut interrompre immédiatement la connexion SSH si le Raspberry Pi change de réseau Wi-Fi.
>
> Il est préférable d'effectuer cette opération lorsque vous pouvez facilement retrouver l'adresse IP du Raspberry Pi.

### 🔍 10.2 — Afficher les connexions enregistrées

Exécuter :

```bash
nmcli connection show
```

Cette commande affiche les profils de connexion connus par NetworkManager.

### 💡 Fonctionnement

Le Raspberry Pi pourra ensuite se reconnecter aux réseaux enregistrés lorsqu'ils seront disponibles.

Le choix du réseau dépend notamment :

- De la disponibilité des réseaux.
- Des paramètres des connexions.
- Des priorités configurées dans NetworkManager.

> 💡 **Conseil**
>
> L'utilisation de plusieurs réseaux Wi-Fi est pratique pour rendre le système transportable.
>
> Cependant, le Raspberry Pi et l'appareil utilisé pour SSH doivent pouvoir communiquer sur le réseau sélectionné.

---

## 11. Maintenir une session active avec tmux

### 📖 Présentation

Lorsqu'on utilise le Raspberry Pi à distance, une connexion SSH peut être interrompue si :

- L'ordinateur passe en veille.
- Le réseau Wi-Fi se déconnecte.
- On ferme accidentellement le terminal.
- On change de réseau.
- On change d'appareil.

Sans précaution particulière, les programmes exécutés dans le terminal peuvent alors être interrompus.

**tmux** permet de conserver une session de terminal active sur le Raspberry Pi, indépendamment de la connexion SSH.

C'est particulièrement utile pour :

- Compiler le firmware Proxmark3 Iceman.
- Exécuter des commandes longues.
- Conserver une session de travail.
- Reprendre une opération après une déconnexion.
- Passer d'un ordinateur à un smartphone sans perdre le terminal.

> 💡 **Important**
>
> `tmux` conserve les programmes exécutés dans sa session même si la connexion SSH est interrompue.
>
> En revanche, il ne maintient pas la connexion SSH elle-même.

### 🛠️ 11.1 — Installer tmux

Depuis le terminal SSH :

```bash
sudo apt update
sudo apt install tmux -y
```

Vérifier ensuite l'installation :

```bash
tmux -V
```

La commande doit afficher le numéro de version installé.

### 💻 11.2 — Créer une session

Exécuter :

```bash
tmux new -s proxmark
```

Une nouvelle session nommée `proxmark` est créée.

Il est désormais possible d'exécuter des commandes dans cette session.

Par exemple, après l'installation du client Iceman :

```bash
pm3
```

> ⚠️ **Attention**
>
> La commande `pm3` ne sera disponible qu'après l'installation du client Proxmark3 Iceman, décrite au chapitre 3.
>
> Il est donc normal qu'elle ne fonctionne pas encore à cette étape du tutoriel.

### 🔌 11.3 — Détacher la session

Pour quitter temporairement la session sans arrêter les programmes :

1. Appuyer sur `Ctrl + B`.
2. Relâcher les touches.
3. Appuyer sur `D`.

Le terminal revient alors à la session SSH classique.

Les programmes lancés dans `tmux` continuent de fonctionner sur le Raspberry Pi.

> 💡 **À retenir**
>
> `Ctrl + B`, puis `D` permet de détacher la session.
>
> Cette combinaison ne ferme pas les programmes en cours.

### 🔄 11.4 — Retrouver une session

Après une nouvelle connexion SSH, exécuter :

```bash
tmux attach -t proxmark
```

Cette commande permet de retrouver la session précédente.

Pour afficher les sessions existantes :

```bash
tmux ls
```

La liste des sessions actives apparaît dans le terminal.

### ⚡ 11.5 — Simplifier avec la commande `pm`

Pour éviter de saisir une longue commande à chaque connexion, nous allons créer un alias Bash.

L'objectif est de pouvoir ouvrir ou retrouver notre session avec une seule commande :

```bash
pm
```

#### Créer l'alias

Exécuter :

```bash
echo "alias pm='tmux new-session -A -s proxmark'" >> ~/.bashrc
```

Cette commande ajoute l'alias au fichier de configuration Bash de l'utilisateur.

#### Recharger la configuration

Exécuter :

```bash
source ~/.bashrc
```

#### Utiliser la commande

Désormais, il suffit de saisir :

```bash
pm
```

Son fonctionnement est automatique :

| Situation | Résultat |
|---|---|
| La session `proxmark` existe | La session est retrouvée |
| La session `proxmark` n'existe pas | Une nouvelle session est créée |

L'alias sera également disponible lors des prochaines connexions SSH utilisant Bash.

> 💡 **Pourquoi utiliser `new-session -A` ?**
>
> L'option `-A` permet de se rattacher à une session existante ou d'en créer une nouvelle si nécessaire.
>
> Cela évite d'avoir à choisir manuellement entre les commandes de création et de récupération d'une session.

### 📱 11.6 — Utilisation quotidienne

Une fois le système entièrement installé, la procédure devient très simple.

#### 1. Se connecter au Raspberry Pi

Depuis un ordinateur ou un terminal SSH compatible :

```bash
ssh piuser@proxmark.local
```

#### 2. Ouvrir ou retrouver la session

```bash
pm
```

#### 3. Lancer le client Proxmark3

Après son installation au chapitre 3 :

```bash
pm3
```

#### 4. Quitter temporairement

Utiliser :

```text
Ctrl + B, puis D
```

La session reste active.

Lors de la prochaine connexion, saisir simplement :

```bash
pm
```

pour retrouver le terminal précédent.

### 🛑 11.7 — Fermer définitivement une session

Pour fermer une session `tmux`, quitter d'abord les programmes en cours.

Exécuter ensuite :

```bash
exit
```

La session sera terminée lorsqu'aucun programme ni fenêtre ne restera actif.

> ⚠️ **Attention**
>
> Fermer une session `tmux` n'est pas la même chose que la détacher.
>
> Une session fermée ne peut pas être récupérée.

### ⚠️ 11.8 — Limitations importantes

`tmux` ne maintient pas la connexion SSH elle-même.

Il permet de conserver les programmes en fonctionnement sur le Raspberry Pi lorsque la connexion SSH est interrompue.

Pour cela :

- Le Raspberry Pi doit rester allumé.
- Les périphériques USB nécessaires doivent rester connectés.
- Le programme doit être exécuté dans la session `tmux`.
- La session ne doit pas être fermée.

Une coupure d'alimentation ou un redémarrage du Raspberry Pi interrompt les sessions `tmux`.

> 💡 **Conseil**
>
> Pour notre projet, l'utilisation de `tmux` est particulièrement recommandée lors des opérations longues et des sessions de travail avec le Proxmark3.
>
> Elle permet notamment de reprendre facilement le terminal après une déconnexion Wi-Fi.

---

## 12. Vérification avant de continuer

Avant de passer à l'étape suivante, assurez-vous que :

- [ ] Raspberry Pi OS Lite (64 bits) est installé.
- [ ] Le Raspberry Pi démarre correctement.
- [ ] Le réseau Wi-Fi fonctionne.
- [ ] Le nom `proxmark.local` ou l'adresse IP permet de joindre le Raspberry Pi.
- [ ] La connexion SSH fonctionne.
- [ ] Le système a été mis à jour.
- [ ] L'architecture `aarch64` a été vérifiée.
- [ ] NetworkManager fonctionne.
- [ ] Le service SSH est actif.
- [ ] `tmux` est installé.
- [ ] La commande `pm` permet d'ouvrir ou retrouver une session.

---

## ✅ Étape 2 terminée — Raspberry Pi configuré

Le Raspberry Pi Zero 2 W est maintenant prêt pour l'installation du client **Proxmark3 Iceman**.

Nous disposons désormais :

- D'un système Linux léger et à jour.
- D'un accès distant via SSH.
- D'une connexion Wi-Fi configurable.
- De la possibilité d'enregistrer plusieurs réseaux Wi-Fi.
- De sessions persistantes avec `tmux`.
- D'une commande simplifiée `pm` pour reprendre le travail après une déconnexion.

### 🚀 Dans l'étape suivante

Nous allons :

- Préparer le système pour Proxmark3 Iceman.
- Installer les dépendances nécessaires.
- Récupérer les sources du projet Iceman.
- Compiler les outils nécessaires.
- Installer le client Proxmark3.
- Vérifier la communication avec le matériel.
- Aborder la mise à jour du firmware.

---

## 🧭 Navigation

| ⬅️ Étape précédente | 🏠 Sommaire | ➡️ Étape suivante |
|---|---|---|
| [Étape 1 — Matériel et branchements](01-materiel.md) | [Retour à l'accueil](../../README.fr.md) | [Étape 3 — Installation de Proxmark3 Iceman](03-installation-proxmark3.md) |

---

📡 **Projet : Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
