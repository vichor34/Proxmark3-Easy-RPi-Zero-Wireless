# 🔧 Étape 6 — Maintenance, sauvegardes et mises à jour

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](../en/06-maintenance.md)

[🏠 Retour au sommaire](../../README.fr.md)

---

## 📖 Présentation

Notre installation **Proxmark3 Easy + Raspberry Pi Zero 2 W** est maintenant opérationnelle.

Nous pouvons utiliser le Proxmark3 à distance depuis un ordinateur, un iPhone, un iPad ou un appareil Android.

Pour conserver une installation fiable, il est important de savoir :

- Vérifier l'état du Raspberry Pi.
- Sauvegarder les fichiers importants.
- Sauvegarder entièrement la carte microSD.
- Mettre à jour Raspberry Pi OS.
- Mettre à jour le client Proxmark3 Iceman.
- Mettre à jour le firmware du Proxmark3 lorsque cela est nécessaire.
- Restaurer une configuration fonctionnelle.
- Diagnostiquer les problèmes matériels et logiciels.

Ce chapitre présente une méthode de maintenance progressive, adaptée à notre matériel.

> ⚠️ **Important**
>
> Une mise à jour du système Linux, une mise à jour du client Iceman et un flashage du firmware Proxmark3 sont trois opérations différentes.
>
> Elles ne doivent pas être confondues.
>
> **Si votre installation fonctionne correctement, il n'est pas nécessaire de flasher régulièrement le Proxmark3.**

### 🎯 Objectifs

À la fin de ce chapitre, vous saurez :

- Identifier les versions actuellement installées.
- Vérifier l'espace disque et l'état du système.
- Sauvegarder la configuration du Raspberry Pi.
- Créer une image de sauvegarde de la carte microSD.
- Mettre à jour les paquets Linux.
- Préparer et compiler une nouvelle version d'Iceman.
- Comprendre les précautions nécessaires avant un flashage.
- Restaurer une version précédente.
- Effectuer les principaux diagnostics en cas de panne.

---

## 1. Configuration de référence

Ce tutoriel concerne l'installation réalisée dans les chapitres précédents.

| Élément | Configuration |
|---|---|
| Raspberry Pi | Zero 2 W |
| Système | Raspberry Pi OS Lite 64 bits |
| Proxmark3 | Easy générique |
| Microcontrôleur | AT91SAM7S512 |
| Mémoire Flash du microcontrôleur | 512 Ko |
| FPGA | Xilinx XC2S30 |
| Firmware | Iceman — RfidResearchGroup |
| Plateforme de compilation | `PM3GENERIC` |
| Ordre des LED | `PM3EASY` |
| Répertoire du projet | `~/proxmark3-new` |
| Accès distant | SSH |
| Sessions persistantes | tmux |

### 📚 Prérequis

Vous devez avoir terminé :

- [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md)
- [Étape 3 — Installation de Proxmark3 Iceman](03-installation-proxmark3.md)
- [Étape 4 — Utilisation quotidienne](04-utilisation-quotidienne.md)

> 💡 **Remarque**
>
> Les commandes supposent que votre nom d'utilisateur est `piuser` et que le projet Iceman est installé dans `~/proxmark3-new`.
>
> Adaptez les chemins si votre configuration est différente.

---

## 2. Vérifier l'état du système

Avant toute maintenance, nous allons vérifier que le Raspberry Pi fonctionne normalement.

### 🔍 2.1 — Vérifier la version du système

Connectez-vous en SSH :

```bash
ssh piuser@proxmark.local
```

Affichez les informations du système :

```bash
cat /etc/os-release
```

Pour connaître la version du noyau Linux :

```bash
uname -a
```

Ces informations sont utiles lors d'un dépannage.

### 💾 2.2 — Vérifier l'espace disque

Exécutez :

```bash
df -h
```

Cette commande affiche l'espace disponible sur les systèmes de fichiers.

Surveillez particulièrement la partition racine :

```text
/
```

Un espace disque insuffisant peut empêcher :

- L'installation de mises à jour.
- La compilation d'Iceman.
- La création de fichiers.
- Le fonctionnement normal de certains services.

### 🧠 2.3 — Vérifier la mémoire

Exécutez :

```bash
free -h
```

Cette commande affiche la mémoire RAM et l'espace d'échange disponibles.

Le Raspberry Pi Zero 2 W dispose de ressources limitées.

Il est donc préférable d'éviter de lancer plusieurs compilations simultanément.

### 🌡️ 2.4 — Vérifier la température

Exécutez :

```bash
vcgencmd measure_temp
```

Cette commande affiche la température du processeur.

Exemple :

```text
temp=45.0'C
```

La température dépend de la charge du système et des conditions de refroidissement.

### ⚡ 2.5 — Vérifier les alertes d'alimentation

Exécutez :

```bash
vcgencmd get_throttled
```

Exemple :

```text
throttled=0x0
```

Une valeur `0x0` indique qu'aucun indicateur de limitation ou de sous-tension n'est signalé dans cet état.

Une valeur différente doit être interprétée à l'aide des indicateurs correspondants.

> ⚠️ **Attention**
>
> Une alimentation insuffisante peut provoquer des erreurs USB et des interruptions de communication avec le Proxmark3.
>
> Il est préférable de résoudre ces problèmes avant toute opération de flashage.

### 📡 2.6 — Vérifier le Proxmark3

Ouvrez la session tmux :

```bash
pm
```

Lancez Iceman si nécessaire :

```bash
pm3
```

Dans le client :

```text
hw version
```

Puis :

```text
hw status
```

Conservez les informations de version avant une mise à jour.

---

## 3. Comprendre les différents types de mises à jour

Notre installation comporte plusieurs éléments logiciels indépendants.

| Élément | Fonction | Méthode de mise à jour |
|---|---|---|
| Raspberry Pi OS | Système d'exploitation | `apt` |
| Client Iceman | Logiciel exécuté sur le Raspberry Pi | Git + compilation |
| Firmware ARM | Programme exécuté sur le Proxmark3 | Flashage |
| FPGA | Logique programmable du Proxmark3 | Selon les composants du firmware |
| Bootloader | Programme de démarrage du Proxmark3 | Procédure spécifique |

### 🐧 3.1 — Raspberry Pi OS

La mise à jour du système concerne notamment :

- Le noyau Linux.
- Les bibliothèques.
- Les outils système.
- Les correctifs de sécurité.

Elle ne met pas automatiquement à jour le firmware du Proxmark3.

### 💻 3.2 — Client Iceman

Le client Iceman est le logiciel exécuté sur le Raspberry Pi.

Il permet d'envoyer des commandes au Proxmark3.

Une nouvelle version peut apporter :

- Des corrections de bugs.
- De nouvelles commandes.
- Des améliorations de compatibilité.
- Des modifications des protocoles de communication.

### 📡 3.3 — Firmware Proxmark3

Le firmware est exécuté directement sur le Proxmark3.

Il doit être compatible avec :

- Le matériel utilisé.
- La configuration de compilation.
- Le client Iceman.

> ⚠️ **Important**
>
> Il est recommandé d'utiliser un client et un firmware provenant de la même compilation.
>
> Mélanger des versions différentes peut entraîner des erreurs ou des incompatibilités.

### 🔒 3.4 — Bootloader

Le bootloader permet notamment de démarrer le périphérique et d'effectuer certaines opérations de programmation.

Il ne doit pas être modifié sans nécessité clairement identifiée.

**Dans notre configuration, nous conservons le bootloader existant.**

---

## 4. Sauvegarder les informations importantes

Avant une mise à jour, nous allons créer une sauvegarde des fichiers et paramètres utiles.

### 📁 4.1 — Créer un répertoire de sauvegarde

Sur le Raspberry Pi :

```bash
mkdir -p ~/backups
chmod 700 ~/backups
```

Créez un répertoire daté :

```bash
BACKUP_DIR="$HOME/backups/$(date +%Y-%m-%d_%H-%M-%S)"
mkdir -p "$BACKUP_DIR"
chmod 700 "$BACKUP_DIR"
```

> 🔐 **Sécurité**
>
> La commande `chmod 700` limite l'accès à ces répertoires à leur propriétaire.
>
> Cette précaution est importante, car les sauvegardes peuvent contenir des identifiants réseau et d'autres informations sensibles.

Vérifiez :

```bash
echo "$BACKUP_DIR"
```

> 💡 **Conseil**
>
> Conservez le chemin affiché.
>
> Il permettra de retrouver facilement cette sauvegarde.

### 📋 4.2 — Enregistrer les informations système

Exécutez :

```bash
cat /etc/os-release > "$BACKUP_DIR/os-release.txt"
```

Puis :

```bash
uname -a > "$BACKUP_DIR/kernel.txt"
```

Enregistrez les paquets installés :

```bash
dpkg --get-selections > "$BACKUP_DIR/packages.txt"
```

### 📶 4.3 — Sauvegarder la configuration réseau

Enregistrez la liste des connexions NetworkManager :

```bash
nmcli connection show > "$BACKUP_DIR/network-connections.txt"
```

Cette commande conserve un inventaire des connexions, mais pas nécessairement tous leurs paramètres.

Pour sauvegarder les profils réseau :

```bash
sudo tar -czf "$BACKUP_DIR/networkmanager.tar.gz" \
  -C /etc NetworkManager/system-connections
```

Cette commande utilise les droits administrateur pour lire les profils réseau protégés.

Le fichier créé peut appartenir à `root`. Pour permettre à votre utilisateur de gérer cette sauvegarde :

```bash
sudo chown "$(id -u):$(id -g)" "$BACKUP_DIR/networkmanager.tar.gz"
```

Protégez ensuite son contenu :

```bash
chmod 600 "$BACKUP_DIR/networkmanager.tar.gz"
```

> 🔐 **Sécurité**
>
> `chmod 600` autorise uniquement le propriétaire à lire et modifier cette archive.
>
> Les profils NetworkManager peuvent contenir des mots de passe Wi-Fi. Ne publiez jamais cette archive dans un dépôt GitHub.

> 🔐 **Sécurité**
>
> Les profils NetworkManager peuvent contenir des mots de passe Wi-Fi et d'autres informations sensibles.
>
> Conservez cette archive dans un emplacement privé et protégé.

### 🖥️ 4.4 — Sauvegarder la configuration Bash

Notre fichier `~/.bashrc` contient notamment l'alias `pm` et la configuration du `PATH`.

Sauvegardez-le :

```bash
cp ~/.bashrc "$BACKUP_DIR/bashrc"
```

### 📡 4.5 — Enregistrer la version Git d'Iceman

Placez-vous dans le répertoire du projet :

```bash
cd ~/proxmark3-new
```

Enregistrez le commit actuel :

```bash
git rev-parse HEAD > "$BACKUP_DIR/iceman-commit.txt"
```

Enregistrez également l'état du dépôt :

```bash
git status --short > "$BACKUP_DIR/iceman-status.txt"
```

Et la configuration de compilation :

```bash
cp Makefile.platform "$BACKUP_DIR/Makefile.platform"
```

> ⚠️ **Important**
>
> Si `git status --short` affiche des fichiers modifiés, identifiez ces modifications avant toute mise à jour.
>
> Ne les écrasez pas sans les avoir sauvegardées.

### 📦 4.6 — Créer une archive de sauvegarde

Revenez dans votre répertoire personnel :

```bash
cd ~
```

Créez une archive du dossier de sauvegarde :

```bash
tar -czf "${BACKUP_DIR}.tar.gz" -C "$HOME/backups" "$(basename "$BACKUP_DIR")"
```

Protégez cette archive :

```bash
chmod 600 "${BACKUP_DIR}.tar.gz"
```

> 🔐 **Important**
>
> Cette archive contient les fichiers sauvegardés précédemment, y compris les éventuels profils Wi-Fi.
>
> Elle doit rester privée et ne doit jamais être publiée sur GitHub ou partagée publiquement.

Vérifiez que l'archive existe :

```bash
ls -lh "${BACKUP_DIR}.tar.gz"
```

### 💻 4.7 — Copier la sauvegarde sur un ordinateur

Depuis un terminal ouvert **sur votre ordinateur**, et non dans la session SSH du Raspberry Pi :

```bash
scp piuser@proxmark.local:/home/piuser/backups/AAAA-MM-JJ_HH-MM-SS.tar.gz .
```

Remplacez `AAAA-MM-JJ_HH-MM-SS` par le nom réel de l'archive.

> 💡 **Conseil**
>
> Une sauvegarde conservée uniquement sur la carte microSD ne protège pas contre une panne de cette carte.
>
> Copiez toujours les sauvegardes importantes sur un autre support.

> 🔐 **Protection des sauvegardes**
>
> Le transfert avec `scp` utilise une connexion SSH chiffrée.
>
> Cependant, le fichier reste une archive non chiffrée sur l'ordinateur de destination.
>
> Conservez-le dans un emplacement privé, idéalement sur un support chiffré, et supprimez les copies temporaires inutiles.

---

## 5. Sauvegarder entièrement la carte microSD

La méthode la plus complète consiste à créer une image de la carte microSD.

Cette image peut permettre de restaurer le Raspberry Pi dans un état antérieur.

### 💾 5.1 — Pourquoi créer une image ?

Une image complète conserve notamment :

- Le système d'exploitation.
- Les logiciels installés.
- Les paramètres réseau.
- Les comptes utilisateurs.
- Le client Iceman.
- Les configurations SSH et tmux.

> ⚠️ **Attention**
>
> Une image de la carte microSD ne sauvegarde pas le firmware déjà installé dans la mémoire du Proxmark3.
>
> Elle sauvegarde uniquement le contenu de la carte du Raspberry Pi.

### 🛑 5.2 — Arrêter correctement le Raspberry Pi

Avant de retirer la carte microSD :

```bash
sudo shutdown -h now
```

Attendez l'arrêt du système.

Débranchez ensuite l'alimentation.

Retirez la carte microSD.

### 💻 5.3 — Créer une image depuis un ordinateur

Insérez la carte microSD dans un lecteur connecté à votre ordinateur.

Utilisez un logiciel capable de créer une image brute d'un support.

La procédure dépend du système d'exploitation.

Sous Linux, il est possible d'utiliser `dd`, mais cette commande nécessite une grande vigilance.

Une erreur dans le choix du périphérique peut entraîner la perte de données.

Pour un débutant, il est préférable d'utiliser un outil graphique permettant de sélectionner clairement la carte source.

### 🔍 5.4 — Vérifier la sauvegarde

Une fois l'image créée :

1. Vérifiez que le fichier existe.
2. Vérifiez que sa taille est cohérente.
3. Conservez-le sur un support distinct.
4. Identifiez clairement la date et la version sauvegardée.

Exemple de nom :

```text
proxmark3-rpi-zero2w-2026-11-01.img
```

> 💡 **Conseil**
>
> La meilleure vérification consiste à tester la restauration sur une autre carte microSD compatible.
>
> Une sauvegarde non testée peut être inutilisable au moment où vous en avez besoin.

---

## 6. Mettre à jour Raspberry Pi OS

Les mises à jour du système sont indépendantes de celles du Proxmark3.

### 🔐 6.1 — Se connecter en SSH

Depuis votre ordinateur :

```bash
ssh piuser@proxmark.local
```

### 📋 6.2 — Actualiser la liste des paquets

Exécutez :

```bash
sudo apt update
```

Cette commande actualise la liste des versions disponibles.

Elle n'installe pas encore les mises à jour.

### 🔄 6.3 — Installer les mises à jour

Exécutez :

```bash
sudo apt full-upgrade
```

Lisez la liste des modifications proposées avant de confirmer.

> ⚠️ **Important**
>
> Une mise à jour importante peut nécessiter un redémarrage.
>
> Il est préférable de ne pas effectuer cette opération pendant une session Iceman active.

### 🔁 6.4 — Redémarrer si nécessaire

```bash
sudo reboot
```

Attendez le redémarrage du Raspberry Pi.

Reconnectez-vous :

```bash
ssh piuser@proxmark.local
```

### ✅ 6.5 — Vérifier le fonctionnement

Contrôlez le système :

```bash
cat /etc/os-release
```

Puis :

```bash
lsusb
```

Vérifiez ensuite que le client Iceman fonctionne toujours :

```bash
pm
```

Si le client Iceman n'est pas déjà actif dans la session tmux, lancez-le :

```bash
pm3
```

> 💡 **Remarque**
>
> Si `pm` vous reconnecte à une session tmux dans laquelle Iceman fonctionne déjà, ne lancez pas une seconde instance du client.
>
> Après un redémarrage du Raspberry Pi, les anciennes sessions tmux n'existent plus.

Dans Iceman :

```text
hw version
```

> 💡 **Remarque**
>
> `apt full-upgrade` met à jour les paquets de la distribution actuellement configurée.
>
> Le passage à une nouvelle version majeure de Debian ou de Raspberry Pi OS est une opération distincte qui nécessite une procédure spécifique.

---

## 7. Préparer une mise à jour d'Iceman

La mise à jour d'Iceman demande davantage de précautions.

Notre Proxmark3 Easy générique possède une configuration matérielle spécifique.

Nous devons conserver les paramètres adaptés à ce modèle.

### ⚠️ 7.1 — Ne pas modifier directement l'installation fonctionnelle

La méthode recommandée consiste à conserver le répertoire actuel :

```text
~/proxmark3-new
```

Nous allons préparer une nouvelle compilation dans un autre répertoire.

Cette méthode permet de revenir facilement au client précédent.

### 📁 7.2 — Créer un nouveau répertoire

Depuis le Raspberry Pi :

```bash
cd ~
```

Clonez le dépôt officiel :

```bash
git clone https://github.com/RfidResearchGroup/proxmark3.git proxmark3-update
```

Entrez dans le répertoire :

```bash
cd ~/proxmark3-update
```

### 🔍 7.3 — Identifier la version téléchargée

Exécutez :

```bash
git rev-parse HEAD
```

Cette commande affiche l'identifiant exact du commit.

Conservez cette information.

> 💡 **Conseil**
>
> Pour une installation reproductible, utilisez un commit identifié et testé plutôt qu'une version choisie uniquement parce qu'elle est la plus récente.

### ⚙️ 7.4 — Vérifier la configuration de compilation

Ouvrez le fichier :

```bash
nano Makefile.platform
```

Notre configuration de référence utilise :

```makefile
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
```

Vérifiez également que la configuration ne contient pas :

```makefile
PLATFORM_EXTRAS=FLASH
```

> ⚠️ **Important**
>
> Ces paramètres correspondent à notre installation de référence.
>
> Ils ne doivent pas être appliqués automatiquement à tous les modèles de Proxmark3.
>
> Vérifiez les instructions du dépôt Iceman pour la révision que vous souhaitez compiler.

Enregistrez le fichier si vous avez effectué des modifications.

### 🔧 7.5 — Compiler Iceman

Exécutez :

```bash
make -j2
```

Immédiatement après la compilation, vérifiez son code de retour :

```bash
echo $?
```

La valeur `0` indique que la compilation s'est terminée correctement.

> ⚠️ **Important**
>
> Exécutez `echo $?` immédiatement après `make -j2`, sans lancer d'autre commande entre les deux.
>
> Si la compilation échoue, ne poursuivez pas vers une opération de flashage.

Le Raspberry Pi Zero 2 W possède des ressources limitées.

L'option `-j2` permet de limiter le nombre de tâches de compilation simultanées.

La compilation peut prendre plusieurs minutes.

### ✅ 7.6 — Vérifier la compilation

Si la compilation se termine sans erreur, vérifiez la présence des fichiers attendus :

```bash
ls -lh pm3
```

Puis :

```bash
ls -lh armsrc/obj/fullimage.elf
```

> ⚠️ **Attention**
>
> La présence de ces fichiers ne garantit pas à elle seule que le firmware est compatible avec votre Proxmark3.
>
> Elle indique seulement que les fichiers attendus ont été produits.

---

## 8. Comprendre le flashage du Proxmark3

Le flashage consiste à écrire un nouveau programme dans la mémoire du Proxmark3.

Cette opération est différente de la compilation du client Iceman.

### 📡 8.1 — Pourquoi flasher le firmware ?

Un flashage peut être nécessaire lorsque :

- Une nouvelle version du client exige un firmware compatible.
- Une fonctionnalité nécessite une mise à jour.
- Une correction importante concerne le firmware.
- Le client et le firmware sont incompatibles.

### ⚠️ 8.2 — Risques du flashage

Un flashage incorrect peut provoquer :

- Une perte de communication avec le Proxmark3.
- Un firmware inutilisable.
- Une incompatibilité matérielle.
- La nécessité d'une procédure de récupération.

Avant de flasher :

1. Vérifiez le modèle exact du Proxmark3.
2. Vérifiez la configuration de compilation.
3. Vérifiez que le firmware a été compilé sans erreur.
4. Vérifiez que l'alimentation est stable.
5. Vérifiez que le câble USB fonctionne correctement.
6. Conservez une copie de votre ancienne installation.
7. Consultez les instructions de la version d'Iceman utilisée.

### 🔒 8.3 — Préserver le bootloader

Notre installation utilise un Proxmark3 Easy générique.

Lors de l'installation initiale, nous avons conservé le bootloader existant.

La procédure de référence utilisait uniquement le firmware ARM :

```bash
./pm3-flash armsrc/obj/fullimage.elf
```

Cette commande est rappelée ici pour identifier la méthode utilisée dans notre installation.

**Ne l'exécutez pas automatiquement après chaque compilation.**

> ⚠️ **Attention**
>
> Ne flashez pas le bootloader sans nécessité et sans procédure de récupération adaptée à votre matériel.
>
> La commande de flashage, les fichiers attendus et les précautions peuvent évoluer selon la version du dépôt.
>
> Avant toute mise à jour réelle, vérifiez la documentation correspondant au commit compilé.

### 🛑 8.4 — Ne pas interrompre un flashage

Pendant une opération de flashage :

- Ne débranchez pas le câble USB.
- Ne coupez pas l'alimentation.
- Ne redémarrez pas le Raspberry Pi.
- Ne lancez pas un second client Iceman.
- N'interrompez pas volontairement le programme de flashage.

> 💡 **Recommandation**
>
> Si votre installation fonctionne et que vous n'avez pas besoin d'une nouvelle fonctionnalité, conservez votre version actuelle.
>
> La stabilité est généralement préférable à une mise à jour inutile.

---

## 9. Vérifier une nouvelle version d'Iceman

Après une mise à jour réussie, nous devons vérifier le fonctionnement du système.

### 🔍 9.1 — Vérifier les versions

Dans le client Iceman correspondant à la nouvelle compilation :

```text
hw version
```

Vérifiez les informations affichées.

### 🩺 9.2 — Vérifier le matériel

```text
hw status
```

### 📡 9.3 — Effectuer un test NFC

Avec un badge ISO 14443-A vous appartenant :

```text
hf 14a reader
```

Approchez le badge de l'antenne HF.

Vérifiez que le Proxmark3 le détecte correctement.

### 🧪 9.4 — Vérifier les antennes

Si nécessaire :

```text
hw tune
```

Comparez les résultats avec ceux obtenus avant la mise à jour, si vous les avez conservés.

### ✅ 9.5 — Valider la mise à jour

Considérez la mise à jour comme validée uniquement lorsque :

- Le client démarre correctement.
- Le Proxmark3 est détecté.
- Les versions sont cohérentes.
- Les commandes de diagnostic fonctionnent.
- Les tests RFID/NFC autorisés fonctionnent.
- Aucune erreur nouvelle n'apparaît.

---

## 10. Revenir à une version précédente

La méthode de restauration dépend de ce qui a été modifié.

### 🐧 10.1 — Si seul Raspberry Pi OS a été modifié

Une image complète de la carte microSD réalisée avant la mise à jour est la méthode de restauration la plus simple.

Elle permet de retrouver le système tel qu'il était au moment de la sauvegarde.

### 💻 10.2 — Si seul le client Iceman a été modifié

Si vous avez conservé l'ancien répertoire :

```text
~/proxmark3-new
```

Vous pouvez relancer l'ancien client depuis ce répertoire :

```bash
cd ~/proxmark3-new
```

Puis :

```bash
./pm3
```

> ⚠️ **Attention**
>
> Cette méthode suppose que le firmware présent sur le Proxmark3 reste compatible avec l'ancien client.
>
> Si le firmware a également été modifié, restaurer uniquement le client peut ne pas suffire.

### 📡 10.3 — Si le firmware a été modifié

Une restauration du firmware nécessite une procédure de flashage compatible avec votre matériel.

Elle peut demander :

- Une ancienne compilation fonctionnelle.
- Les fichiers firmware correspondants.
- Un bootloader opérationnel.
- Une procédure de récupération adaptée.

> ⚠️ **Important**
>
> Une sauvegarde de la carte microSD ne restaure pas automatiquement le firmware du Proxmark3.
>
> En cas d'échec du flashage, évitez de multiplier les tentatives avec des fichiers non vérifiés.

### 💾 10.4 — Restaurer une image microSD

Depuis un ordinateur :

1. Arrêtez et débranchez le Raspberry Pi.
2. Retirez la carte microSD.
3. Insérez-la dans un lecteur.
4. Utilisez un logiciel adapté pour écrire l'image sauvegardée.
5. Vérifiez soigneusement le support de destination.
6. Attendez la fin de l'écriture.
7. Réinsérez la carte dans le Raspberry Pi.
8. Redémarrez et vérifiez le fonctionnement.

> ⚠️ **Attention**
>
> Restaurer une image écrase les données présentes sur la carte de destination.
>
> Vérifiez toujours que vous avez sélectionné le bon support.

---

## 11. Dépannage avancé

### ❌ 11.1 — Le Proxmark3 n'apparaît plus en USB

Exécutez :

```bash
lsusb
```

Puis :

```bash
ls -l /dev/ttyACM*
```

Consultez les messages système :

```bash
sudo dmesg | tail -n 50
```

Vérifiez :

- Le câble USB.
- L'adaptateur OTG.
- L'alimentation.
- Le port USB utilisé.
- L'état du Proxmark3.

### ❌ 11.2 — Le client Iceman ne démarre plus

Vérifiez :

```bash
command -v pm3
```

Puis :

```bash
ls -lh ~/proxmark3-new/pm3
```

Essayez :

```bash
cd ~/proxmark3-new
```

Puis :

```bash
./pm3
```

Si le programme démarre mais ne communique pas avec le Proxmark3, vérifiez les messages affichés.

### ❌ 11.3 — Le port série est inaccessible

Vérifiez :

```bash
ls -l /dev/ttyACM*
```

Puis les groupes de votre utilisateur :

```bash
groups
```

Sur Raspberry Pi OS, l'accès aux ports série est généralement géré par les permissions du périphérique et l'appartenance à certains groupes, notamment `dialout`.

Si nécessaire, consultez la configuration des permissions avant de la modifier.

> ⚠️ **Conseil**
>
> Évitez d'utiliser systématiquement `sudo` pour lancer Iceman.
>
> Il est préférable de résoudre correctement les problèmes de permissions.

### ❌ 11.4 — Le Raspberry Pi manque de mémoire

Vérifiez :

```bash
free -h
```

Puis :

```bash
df -h
```

Lors d'une compilation, réduisez le nombre de tâches simultanées.

Par exemple :

```bash
make -j1
```

La compilation sera plus lente, mais pourra nécessiter moins de mémoire.

### ❌ 11.5 — La compilation échoue

Vérifiez :

1. Que les dépendances sont installées.
2. Que la configuration matérielle est correcte.
3. Que l'espace disque est suffisant.
4. Que la mémoire disponible est suffisante.
5. Que le dépôt ne contient pas de modifications inattendues.

Affichez l'état du dépôt :

```bash
git status
```

Vérifiez le commit :

```bash
git rev-parse HEAD
```

> 💡 **Conseil**
>
> Conservez le message d'erreur complet.
>
> Les dernières lignes d'une compilation ne suffisent pas toujours à identifier la véritable cause du problème.

### ❌ 11.6 — Le Raspberry Pi ne répond plus en SSH

Vérifiez :

- L'alimentation.
- Le réseau Wi-Fi.
- L'adresse IP.
- Le nom `proxmark.local`.
- Le fonctionnement du service SSH.

Si vous disposez d'un accès local :

```bash
systemctl status ssh
```

Vérifiez également la connexion réseau :

```bash
nmcli device status
```

### ❌ 11.7 — Une session tmux a disparu

Listez les sessions :

```bash
tmux ls
```

Si aucune session n'existe, créez-en une :

```bash
pm
```

Puis relancez Iceman :

```bash
pm3
```

> 💡 **Rappel**
>
> tmux ne conserve pas les sessions après un redémarrage ou une coupure d'alimentation.

---

## 12. Bonnes pratiques de maintenance

Pour conserver une installation stable, nous recommandons les pratiques suivantes.

### 🔒 12.1 — Conserver une version fonctionnelle

Avant une mise à jour importante :

- Notez le commit Git utilisé.
- Sauvegardez `Makefile.platform`.
- Conservez les fichiers de configuration.
- Créez une image de la carte microSD.
- Gardez une copie de l'ancienne compilation.

### 📋 12.2 — Modifier un seul élément à la fois

Évitez de mettre à jour simultanément :

- Raspberry Pi OS.
- Le client Iceman.
- Le firmware Proxmark3.

En procédant étape par étape, il est plus facile d'identifier la cause d'un problème.

### ⚡ 12.3 — Vérifier l'alimentation

Une alimentation stable est particulièrement importante pendant :

- Les compilations.
- Les mises à jour.
- Les opérations de flashage.

### 🔄 12.4 — Ne pas mettre à jour sans raison

Une nouvelle version n'est pas automatiquement plus adaptée à votre matériel.

Avant de mettre à jour, vérifiez :

- Les corrections apportées.
- Les nouvelles fonctionnalités.
- Les changements de compatibilité.
- Les éventuelles limitations matérielles.

### 📚 12.5 — Conserver une trace des modifications

Vous pouvez créer un fichier de suivi :

```bash
nano ~/maintenance.log
```

Exemple :

```text
Date : 2026-11-01

Raspberry Pi :
- Raspberry Pi OS Lite 64 bits
- Mise à jour système effectuée

Proxmark3 :
- Easy générique
- Plateforme PM3GENERIC
- LED_ORDER PM3EASY

Iceman :
- Commit Git : à renseigner
- Client testé : oui
- Firmware testé : oui

Remarques :
- Installation fonctionnelle
```

Ce fichier permettra de retrouver l'historique de votre installation.

---

## 13. Checklist de maintenance

Avant de considérer une opération de maintenance comme terminée :

- [ ] Le Raspberry Pi démarre correctement.
- [ ] La connexion Wi-Fi fonctionne.
- [ ] SSH est accessible.
- [ ] L'espace disque est suffisant.
- [ ] L'alimentation est stable.
- [ ] Le Proxmark3 est détecté en USB.
- [ ] Le client Iceman démarre.
- [ ] Les versions client et firmware sont compatibles.
- [ ] `hw version` fonctionne.
- [ ] `hw status` fonctionne.
- [ ] Un test RFID/NFC autorisé fonctionne.
- [ ] Les commandes `pm` et `pm3` fonctionnent.
- [ ] La configuration de compilation est sauvegardée.
- [ ] Le commit Git utilisé est connu.
- [ ] Une sauvegarde récente existe sur un autre support.
- [ ] Les modifications effectuées sont documentées.

---

## ✅ Étape 6 terminée — Installation sauvegardée et maintenable

Nous savons maintenant comment entretenir notre installation **Proxmark3 Easy + Raspberry Pi Zero 2 W**.

Nous avons appris à :

- Vérifier l'état du Raspberry Pi.
- Identifier les versions installées.
- Sauvegarder les paramètres importants.
- Créer une image complète de la carte microSD.
- Mettre à jour Raspberry Pi OS.
- Préparer une nouvelle compilation d'Iceman.
- Comprendre les risques du flashage.
- Préserver le bootloader.
- Restaurer une version précédente.
- Diagnostiquer les problèmes courants.

Notre Proxmark3 Easy peut désormais être utilisé et maintenu de manière plus fiable.

### 🎯 Pour aller plus loin

Le projet peut évoluer avec de nouvelles fonctionnalités :

- Amélioration de l'autonomie.
- Utilisation d'une batterie externe.
- Création d'un boîtier imprimé en 3D.
- [Configuration d'un hotspot Wi-Fi automatique](07-hotspot-wifi.md).
- Automatisation du démarrage d'Iceman.
- Amélioration de la sécurité SSH.
- Développement de scripts de diagnostic.

Ces améliorations pourront faire l'objet de guides complémentaires.

---

## 📚 Références

- [Proxmark3 Iceman — RfidResearchGroup](https://github.com/RfidResearchGroup/proxmark3)
- [Raspberry Pi — Documentation officielle](https://www.raspberrypi.com/documentation/)
- [NetworkManager — Documentation](https://networkmanager.dev/docs/)
- [GNU tmux — Dépôt officiel](https://github.com/tmux/tmux)
- [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md)
- [Étape 3 — Installation de Proxmark3 Iceman](03-installation-proxmark3.md)
- [Étape 4 — Utilisation quotidienne](04-utilisation-quotidienne.md)
- [Étape 7 — Hotspot Wi-Fi automatique](07-hotspot-wifi.md)

---

## 🧭 Navigation

| ⬅️ Étape précédente | 🏠 Sommaire | ➡️ Étape suivante | 🌐 English |
|---|---|---|---|
| [Étape 5 — Connexion mobile](05-connexion-mobile.md) | [Retour à l'accueil](../../README.fr.md) | [Étape 7 — Hotspot Wi-Fi automatique](07-hotspot-wifi.md) | [Step 6 — Maintenance, Backups and Updates](../en/06-maintenance.md) |

---

📡 **Projet : Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
