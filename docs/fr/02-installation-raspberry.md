# 💻 Installation de Raspberry Pi OS

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](../en/02-raspberry-installation.md)

[🏠 Retour à l'accueil](../../README.fr.md)

---

## 1. Présentation

Ce chapitre explique comment installer et configurer Raspberry Pi OS sur un Raspberry Pi Zero 2 W afin de préparer son utilisation avec le Proxmark3 Easy.

L'installation s'effectue sans écran, clavier ni souris. Le Raspberry Pi sera administré à distance via SSH.

### Configuration de référence

| Élément               | Configuration        |
| --------------------- | -------------------- |
| Raspberry Pi          | Zero 2 W             |
| Système               | Raspberry Pi OS Lite |
| Architecture          | 64 bits              |
| Base Debian           | Debian 13 (Trixie)   |
| Interface graphique   | Aucune               |
| Gestionnaire réseau   | NetworkManager       |
| Accès distant         | SSH                  |
| Sessions persistantes | tmux                 |

La configuration présentée correspond au système utilisé pour ce projet.

---

## 2. Télécharger Raspberry Pi Imager

Télécharger et installer **Raspberry Pi Imager**, l'outil officiel permettant de préparer la carte microSD.

🔗 [Télécharger Raspberry Pi Imager](https://www.raspberrypi.com/software/)

Insérer ensuite la carte microSD dans l'ordinateur.

Une carte de 16 Go ou plus est recommandée.

---

## 3. Préparer la carte microSD

Lancer Raspberry Pi Imager.

### Étape 1 — Choisir le matériel

Sélectionner :

**Raspberry Pi Zero 2 W**

### Étape 2 — Choisir le système

Sélectionner :

**Raspberry Pi OS Lite (64-bit)**

Cette version ne possède pas d'interface graphique et convient particulièrement à une utilisation à distance.

### Étape 3 — Choisir le stockage

Sélectionner la carte microSD.

⚠️ Attention : toutes les données présentes sur la carte seront effacées.

---

## 4. Configuration initiale

Avant de lancer l'écriture, Raspberry Pi Imager permet de personnaliser le système.

### Nom de la machine

Définir le nom :

```text
proxmark
```

Le Raspberry Pi sera ainsi accessible sur le réseau local à l'adresse :

```text
proxmark.local
```

Cette résolution de nom dépend de la prise en charge de mDNS par le réseau et l'appareil utilisé.

### Compte utilisateur

Créer un utilisateur et définir un mot de passe sécurisé.

Dans ce tutoriel, nous utiliserons le nom générique :

```text
piuser
```

Il est possible de choisir un autre nom.

### Configuration Wi-Fi

Renseigner :

* Le nom du réseau Wi-Fi (SSID).
* Le mot de passe Wi-Fi.
* Le pays : France (FR).

Le Raspberry Pi Zero 2 W utilise le Wi-Fi 2,4 GHz. Vérifier que le réseau choisi est compatible.

### Activer SSH

Dans les paramètres de Raspberry Pi Imager :

1. Activer SSH.
2. Sélectionner l'authentification par mot de passe pour une première installation.
3. Vérifier que le compte utilisateur possède un mot de passe sécurisé.

L'authentification par clé SSH pourra être configurée ultérieurement pour renforcer la sécurité.

---

## 5. Écriture de la carte microSD

Une fois la configuration terminée :

1. Lancer l'écriture.
2. Attendre la fin de l'opération.
3. Laisser Raspberry Pi Imager vérifier les données.
4. Retirer proprement la carte microSD.

---

## 6. Premier démarrage

1. Insérer la carte microSD dans le Raspberry Pi Zero 2 W.
2. Connecter l'alimentation.
3. Attendre quelques minutes pendant le premier démarrage.

Le Raspberry Pi doit automatiquement rejoindre le réseau Wi-Fi configuré.

---

## 7. Première connexion SSH

Depuis un ordinateur connecté au même réseau, ouvrir un terminal.

Sous Windows, utiliser PowerShell.

Exécuter :

```bash
ssh piuser@proxmark.local
```

Remplacer `piuser` par le nom choisi lors de l'installation.

Lors de la première connexion, confirmer l'empreinte du serveur après avoir vérifié qu'il s'agit bien de votre Raspberry Pi.

Saisir ensuite le mot de passe.

Une fois connecté, le terminal affiche une invite de commande similaire à :

```text
piuser@proxmark:~ $
```

Le Raspberry Pi peut maintenant être administré à distance.

### En cas d'échec

Si `proxmark.local` ne fonctionne pas :

1. Vérifier que le Raspberry Pi est connecté au Wi-Fi.
2. Rechercher son adresse IP dans l'interface d'administration du routeur.
3. Se connecter directement à cette adresse :

```bash
ssh piuser@192.168.1.100
```

L'adresse `192.168.1.100` est un exemple à remplacer par l'adresse réelle du Raspberry Pi.

---

## 8. Mise à jour du système

Une fois connecté en SSH, mettre à jour les paquets :

```bash
sudo apt update
```

Puis :

```bash
sudo apt full-upgrade -y
```

Redémarrer si nécessaire :

```bash
sudo reboot
```

Attendre le redémarrage avant de se reconnecter.

---

## 9. Vérifier l'installation

### Version du système

```bash
cat /etc/os-release
```

La configuration de référence utilise Debian 13 (Trixie).

### Architecture

```bash
uname -m
```

Résultat attendu :

```text
aarch64
```

Ce résultat indique un système ARM 64 bits.

### Vérifier le réseau

```bash
nmcli device status
```

L'interface `wlan0` doit apparaître comme connectée.

### Vérifier SSH

```bash
systemctl status ssh
```

Le service doit être actif (`active (running)`).

---

## 10. Ajouter un réseau Wi-Fi secondaire

Il est possible d'enregistrer plusieurs réseaux Wi-Fi.

Pour ajouter un nouveau réseau :

```bash
sudo nmcli device wifi connect "NOM_DU_WIFI" --ask
```

NetworkManager demandera le mot de passe si nécessaire.

⚠️ Attention : cette commande peut interrompre la connexion SSH actuelle si le Raspberry Pi change immédiatement de réseau.

Pour afficher les connexions enregistrées :

```bash
nmcli connection show
```

Le Raspberry Pi pourra ensuite se reconnecter aux réseaux enregistrés lorsqu'ils seront disponibles, selon les priorités et les paramètres de NetworkManager.

---

## 11. Maintenir une session active avec tmux

Lorsqu'on utilise le Raspberry Pi à distance, une connexion SSH peut être interrompue si :

* L'ordinateur passe en veille.
* Le réseau Wi-Fi se déconnecte.
* On ferme accidentellement le terminal.
* On change de réseau ou d'appareil.

Sans précaution particulière, les programmes exécutés dans le terminal peuvent alors être interrompus.

**tmux** permet de conserver une session de terminal active sur le Raspberry Pi, indépendamment de la connexion SSH.

C'est particulièrement utile pour compiler le firmware Proxmark3 Iceman ou effectuer des opérations longues.

### 11.1 Installer tmux

Depuis le terminal SSH :

```bash
sudo apt update
sudo apt install tmux -y
```

Vérifier l'installation :

```bash
tmux -V
```

La commande doit afficher la version installée.

### 11.2 Créer une session

Exécuter :

```bash
tmux new -s proxmark
```

Une nouvelle session nommée `proxmark` est créée.

Il est désormais possible de lancer des commandes dans cette session, par exemple :

```bash
pm3
```

La commande `pm3` sera disponible après l'installation du client Iceman décrite au chapitre 3.

### 11.3 Détacher la session

Pour quitter temporairement la session sans arrêter les programmes :

1. Appuyer sur `Ctrl + B`.
2. Relâcher les touches.
3. Appuyer sur `D`.

Le terminal revient à la session SSH classique.

Les programmes lancés dans `tmux` continuent de fonctionner sur le Raspberry Pi.

### 11.4 Retrouver une session

Après une nouvelle connexion SSH :

```bash
tmux attach -t proxmark
```

Cette commande permet de retrouver la session précédente.

Pour afficher les sessions existantes :

```bash
tmux ls
```

### 11.5 Simplifier avec la commande `pm`

Pour éviter de saisir une longue commande à chaque connexion, nous allons créer un alias Bash.

Exécuter :

```bash
echo "alias pm='tmux new-session -A -s proxmark'" >> ~/.bashrc
```

Recharger la configuration :

```bash
source ~/.bashrc
```

Désormais, une seule commande suffit :

```bash
pm
```

Son fonctionnement est automatique :

* Si la session `proxmark` existe, elle est retrouvée.
* Si elle n'existe pas, une nouvelle session est créée.

L'alias sera disponible lors des prochaines connexions SSH avec Bash.

### 11.6 Utilisation quotidienne

La procédure devient très simple.

**1. Se connecter au Raspberry Pi :**

```bash
ssh piuser@proxmark.local
```

**2. Ouvrir ou retrouver la session :**

```bash
pm
```

**3. Lancer Proxmark3 :**

```bash
pm3
```

**4. Pour quitter temporairement :**

`Ctrl + B`, puis `D`.

Lors de la prochaine connexion, saisir simplement `pm` pour retrouver la session.

### 11.7 Fermer définitivement une session

Pour fermer la session, quitter d'abord les programmes en cours, puis exécuter :

```bash
exit
```

La session `tmux` sera terminée lorsqu'aucun programme ni fenêtre ne restera actif.

### 11.8 Limitations importantes

**tmux ne maintient pas la connexion SSH elle-même.**

Il conserve les programmes en fonctionnement sur le Raspberry Pi lorsque la connexion SSH est interrompue.

Le Raspberry Pi doit rester allumé et les périphériques USB nécessaires doivent rester connectés.

Une coupure d'alimentation ou un redémarrage du Raspberry Pi interrompt les sessions `tmux`.

---

## ✅ Installation terminée

Le Raspberry Pi Zero 2 W est maintenant prêt pour l'installation du client Proxmark3 Iceman.

Nous disposons désormais :

* D'un système Linux léger et à jour.
* D'un accès distant via SSH.
* D'une connexion Wi-Fi configurable.
* De sessions persistantes avec `tmux`.
* D'une commande simplifiée `pm` pour reprendre le travail après une déconnexion.

➡️ [Chapitre 3 — Installation du Proxmark3 et mise à jour du firmware](03-installation-proxmark3.md)
