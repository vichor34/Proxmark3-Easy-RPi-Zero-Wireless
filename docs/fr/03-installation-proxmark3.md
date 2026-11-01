# 🔧 Étape 3 — Installation et mise à jour de Proxmark3 Iceman

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](../en/03-proxmark3-installation.md)

[🏠 Retour au sommaire](../../README.fr.md)

---

## 📖 Présentation

Ce troisième chapitre explique comment installer, compiler et utiliser le firmware **Proxmark3 Iceman**, développé par la communauté **RfidResearchGroup**, sur un **Proxmark3 Easy** connecté à un **Raspberry Pi Zero 2 W**.

Le Raspberry Pi communique avec le Proxmark3 par USB et permet de le contrôler à distance depuis un ordinateur ou un smartphone grâce à une connexion Wi-Fi et SSH.

Nous allons également expliquer comment mettre à jour une installation existante en conservant l'ancien bootloader.

La procédure présentée correspond à la configuration matérielle utilisée et testée dans ce projet.

### 🎯 Objectifs de cette étape

À la fin de ce chapitre, vous saurez :

- Vérifier la connexion USB du Proxmark3 Easy.
- Installer les outils nécessaires à la compilation.
- Télécharger le code source officiel Iceman.
- Configurer la compilation pour un Proxmark3 Easy générique.
- Compiler le client et le firmware.
- Sauvegarder une installation existante.
- Mettre à jour uniquement le firmware ARM.
- Vérifier le fonctionnement du Proxmark3.
- Configurer la commande `pm3`.
- Utiliser le client Iceman dans une session `tmux`.

> ⚠️ **Avertissement important**
>
> Le flashage d'un firmware incompatible peut rendre le Proxmark3 inutilisable.
>
> Vérifiez impérativement votre modèle et sa configuration matérielle avant toute écriture.
>
> Ne débranchez jamais le Proxmark3 pendant une opération de flashage.
>
> Cette procédure a été testée sur le matériel utilisé dans ce projet. Elle ne garantit pas la compatibilité avec toutes les variantes de Proxmark3 Easy.

---

## 1. Configuration de référence

Le tutoriel repose sur la configuration suivante :

| Élément | Configuration |
|---|---|
| Raspberry Pi | Zero 2 W |
| Système | Raspberry Pi OS Lite |
| Architecture | ARM64 / aarch64 |
| Base Debian | Debian 13 (Trixie) |
| Proxmark3 | Easy générique |
| Microcontrôleur | AT91SAM7S512 |
| Mémoire flash interne | 512 Ko |
| FPGA | Xilinx XC2S30 |
| Firmware | Iceman — RfidResearchGroup |
| Connexion | USB via adaptateur OTG alimenté |
| Accès distant | SSH |
| Sessions persistantes | tmux |

### 💡 Pourquoi utiliser Iceman ?

Iceman est un firmware alternatif pour Proxmark3, maintenu par la communauté RfidResearchGroup.

Il apporte notamment :

- Une interface en ligne de commande complète.
- De nombreuses fonctions d'analyse RFID et NFC.
- Des outils de diagnostic du matériel.
- Des fonctionnalités de lecture et d'analyse des protocoles.
- Des mises à jour et améliorations régulières.

Le client Iceman fonctionne directement dans le terminal Linux du Raspberry Pi.

Il n'est donc pas nécessaire d'installer une interface graphique.

### 📚 Prérequis

Avant de commencer, vous devez avoir terminé :

- [Étape 1 — Matériel et branchements](01-materiel.md)
- [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md)

Le Raspberry Pi doit être accessible en SSH et disposer d'une connexion Internet.

---

## 2. Vérifier la connexion USB

Le Proxmark3 Easy doit être connecté au port USB de données du Raspberry Pi Zero 2 W.

Les branchements sont décrits dans le chapitre 1.

### 🔍 2.1 — Afficher les périphériques USB

Depuis le terminal SSH :

```bash
lsusb
```

Cette commande affiche les périphériques USB détectés par Linux.

### 🔍 2.2 — Vérifier le port série

Exécuter :

```bash
ls -l /dev/ttyACM*
```

Dans notre installation, le Proxmark3 apparaît sous :

```text
/dev/ttyACM0
```

Le numéro peut varier selon les périphériques connectés.

> 💡 **Remarque**
>
> Si aucun périphérique `/dev/ttyACM*` n'apparaît, vérifiez :
>
> - Le câble USB.
> - L'adaptateur OTG alimenté.
> - L'alimentation du Raspberry Pi.
> - Le port USB utilisé.
> - Le mode de fonctionnement du Proxmark3.
>
> L'absence de ce périphérique ne signifie pas nécessairement que le Proxmark3 est défectueux.

---

## 3. Installer les outils de compilation

La compilation d'Iceman nécessite plusieurs outils de développement.

### 🔄 3.1 — Mettre à jour les dépôts

Exécuter :

```bash
sudo apt update
```

### 📦 3.2 — Installer les dépendances

Exécuter :

```bash
sudo apt install -y git build-essential make gcc-arm-none-eabi \
  libreadline-dev libusb-0.1-4-dev libpcsclite-dev pkg-config \
  libbz2-dev liblz4-dev libssl-dev python3 python3-dev
```

Ces paquets permettent notamment :

- De télécharger les sources avec Git.
- De compiler les programmes Linux.
- De compiler le firmware ARM.
- De gérer les bibliothèques nécessaires au client.
- D'utiliser les outils Python requis.

> 💡 **Remarque**
>
> Les dépendances peuvent évoluer avec les nouvelles versions d'Iceman.
>
> Si la compilation signale une bibliothèque manquante, consultez les instructions du dépôt officiel.

### 🔍 3.3 — Vérifier le compilateur ARM

Exécuter :

```bash
arm-none-eabi-gcc --version
```

Le terminal doit afficher la version du compilateur.

Cela confirme que l'outil nécessaire à la compilation du firmware ARM est disponible.

---

## 4. Télécharger Proxmark3 Iceman

Le firmware Iceman est développé et maintenu par la communauté RfidResearchGroup.

🔗 [Dépôt officiel Proxmark3 Iceman](https://github.com/RfidResearchGroup/proxmark3)

### 📥 4.1 — Télécharger les sources

Se placer dans le répertoire personnel :

```bash
cd ~
```

Cloner le dépôt officiel :

```bash
git clone https://github.com/RfidResearchGroup/proxmark3.git proxmark3-new
```

Accéder au répertoire :

```bash
cd ~/proxmark3-new
```

> 💡 **Pourquoi utiliser `proxmark3-new` ?**
>
> Ce nom permet de conserver une éventuelle ancienne installation dans `~/proxmark3`.
>
> Nous pouvons ainsi préparer et tester la nouvelle version sans supprimer immédiatement les anciens fichiers.

### 🔍 4.2 — Vérifier la version téléchargée

Exécuter :

```bash
git log -1 --oneline
```

Cette commande affiche le dernier commit du dépôt téléchargé.

### 📌 Version utilisée dans notre projet

Notre installation testée utilisait le commit :

```text
754bbae3d
```

Daté du 30 octobre 2026.

Les versions ultérieures peuvent présenter des différences de compilation ou de fonctionnement.

> ⚠️ **Important**
>
> La commande `git clone` télécharge la version disponible au moment de son exécution.
>
> Elle ne garantit donc pas l'utilisation du même commit que notre installation testée.
>
> Pour reproduire exactement cette version, il est possible de sélectionner le commit correspondant :
>
> ```bash
> git checkout 754bbae3d
> ```
>
> Cette commande doit être exécutée depuis le dossier `~/proxmark3-new`.
>
> Pour une nouvelle installation, privilégiez la version validée pour votre matériel.

---

## 5. Configurer le Proxmark3 Easy

Le firmware doit être compilé pour la bonne configuration matérielle.

### 🔧 5.1 — Créer le fichier de configuration

Accéder au répertoire :

```bash
cd ~/proxmark3-new
```

Créer le fichier `Makefile.platform` :

```bash
cat > Makefile.platform <<'EOF_CONFIG'
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
EOF_CONFIG
```

### 📋 5.2 — Paramètres de compilation

| Paramètre | Valeur | Description |
|---|---|---|
| PLATFORM | PM3GENERIC | Plateforme Proxmark3 générique |
| LED_ORDER | PM3EASY | Configuration des LED du Proxmark3 Easy |

Ces paramètres correspondent au Proxmark3 Easy générique utilisé dans ce projet.

Notre compilation n'activait pas :

```text
PLATFORM_EXTRAS=FLASH
```

Même si le matériel détectait une mémoire flash externe.

> ⚠️ **Attention**
>
> Ne copiez pas automatiquement ces paramètres sur un autre modèle de Proxmark3.
>
> Les variantes matérielles peuvent utiliser des composants ou des configurations différents.

### 🔍 5.3 — Vérifier la configuration

Exécuter :

```bash
cat Makefile.platform
```

Résultat attendu :

```text
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
```

Si ces deux lignes apparaissent correctement, nous pouvons passer à la compilation.

---

## 6. Compiler le client et le firmware

La compilation peut prendre plusieurs minutes sur un Raspberry Pi Zero 2 W.

### 🛠️ 6.1 — Lancer la compilation

Depuis le dossier du projet :

```bash
cd ~/proxmark3-new
```

Exécuter :

```bash
make -j2
```

L'option `-j2` permet de lancer deux tâches de compilation simultanément.

Ce réglage limite le parallélisme afin de tenir compte des ressources du Raspberry Pi Zero 2 W.

> 💡 **Conseil**
>
> Pour une compilation longue, utilisez une session `tmux`, comme expliqué au chapitre 2.
>
> Cela permet de retrouver le terminal après une déconnexion SSH.

### 🔍 6.2 — Vérifier le résultat

Attendre la fin de la compilation.

Vérifier qu'aucune erreur n'est affichée.

Le code de retour de `make` doit être égal à `0` pour indiquer une compilation réussie.

Si la compilation échoue, il faut résoudre l'erreur avant de poursuivre.

### 📦 6.3 — Fichiers générés

Les principaux fichiers utilisés sont :

| Fichier | Description |
|---|---|
| `armsrc/obj/fullimage.elf` | Firmware ARM |
| `bootrom/obj/bootrom.elf` | Bootloader |
| `pm3` | Script de lancement du client |
| `pm3-flash` | Outil de flashage |

### ⚠️ Précaution concernant le bootloader

La compilation peut générer un nouveau bootloader.

**Cela ne signifie pas qu'il faut obligatoirement le flasher.**

Dans notre projet, nous avons choisi de conserver le bootloader existant.

Cette décision permet d'éviter une modification inutile de cette partie sensible du Proxmark3.

---

## 7. Sauvegarder une installation précédente

Cette section concerne les utilisateurs possédant déjà une installation Iceman fonctionnelle.

Pour une première installation, vous pouvez passer directement à la section suivante.

### 💾 7.1 — Créer une sauvegarde

Si l'ancienne installation se trouve dans :

```text
~/proxmark3
```

Se placer dans le répertoire personnel :

```bash
cd ~
```

Créer une archive :

```bash
tar -czf proxmark3-backup-$(date +%F).tar.gz proxmark3
```

Cette commande crée une archive compressée contenant l'ancien dossier.

### 🔍 7.2 — Vérifier la sauvegarde

Exécuter :

```bash
tar -tzf proxmark3-backup-$(date +%F).tar.gz > /dev/null && echo "Archive OK"
```

Résultat attendu :

```text
Archive OK
```

### ⚠️ Limites de cette sauvegarde

Cette archive contient les fichiers logiciels de l'ancienne installation.

Elle ne constitue **pas une sauvegarde complète de la mémoire interne du Proxmark3**.

Conservez séparément vos données et configurations personnelles.

Ne publiez jamais de dumps RFID privés ou de clés d'authentification sur GitHub.

---

## 8. Mettre à jour le firmware ARM

Nous allons maintenant transférer le nouveau firmware vers le Proxmark3 Easy.

Dans notre installation testée, nous avons choisi de mettre à jour uniquement le firmware ARM, en conservant le bootloader existant.

### ⚠️ Avertissement avant le flashage

Le flashage comporte des risques.

Avant de poursuivre :

- Vérifiez le modèle exact de votre Proxmark3.
- Vérifiez les paramètres de compilation.
- Assurez-vous que la compilation s'est terminée correctement.
- Utilisez une alimentation stable.
- Ne débranchez pas le Proxmark3 pendant l'opération.
- Ne flashez pas le bootloader sans nécessité clairement identifiée.

### 🔍 8.1 — Vérifier le périphérique

Depuis le nouveau dépôt :

```bash
cd ~/proxmark3-new
```

Exécuter :

```bash
./pm3-flash --list
```

Vérifier que le Proxmark3 est détecté.

Dans notre installation, il apparaissait sous :

```text
/dev/ttyACM0
```

Le numéro peut varier.

### ⚡ 8.2 — Flasher uniquement le firmware ARM

**Ne poursuivez que si votre matériel correspond à la configuration documentée et que le firmware a été compilé correctement.**

Exécuter :

```bash
./pm3-flash armsrc/obj/fullimage.elf
```

Attendre la fin du transfert.

Dans notre installation, l'opération s'est terminée par :

```text
All done
```

### 🔒 8.3 — Conserver le bootloader

Dans cette procédure, nous n'utilisons pas l'option :

```text
-b
```

Cette option autorise l'écriture dans la zone du bootloader.

Notre mise à jour réussie a remplacé uniquement le firmware ARM, tout en conservant le bootloader précédent.

> ⚠️ **Important**
>
> Ne flashez pas automatiquement le fichier `bootrom.elf`.
>
> Une mise à jour du bootloader doit faire l'objet d'une procédure spécifique, adaptée au modèle matériel concerné.

---

## 9. Vérifier le fonctionnement

Une fois le flashage terminé, nous allons vérifier la communication entre le Raspberry Pi et le Proxmark3.

### 💻 9.1 — Lancer le client

Depuis le nouveau dépôt :

```bash
cd ~/proxmark3-new
```

Exécuter :

```bash
./pm3
```

Le client Iceman doit démarrer et établir la communication avec le Proxmark3.

### 🔍 9.2 — Vérifier les versions

Dans le terminal du client Proxmark3 :

```text
hw version
```

Puis :

```text
hw status
```

Ces commandes permettent de consulter les informations du matériel et du firmware.

### 📋 9.3 — Vérifier la cohérence des versions

Vérifier notamment :

- La version du client.
- La version du firmware ARM.
- La version du bootloader.
- Les informations matérielles.
- L'état général du périphérique.

Les versions **Client** et **OS** doivent correspondre au nouveau code utilisé.

La version **Bootrom** peut être plus ancienne.

C'est normal puisque nous avons volontairement conservé le bootloader précédent.

> 💡 **Remarque**
>
> La présence du mot `suspect` dans certaines informations de version ne signifie pas nécessairement une panne matérielle.
>
> Il faut interpréter cette indication avec les autres informations de diagnostic.

### 📡 9.4 — Tester la lecture NFC

Pour vérifier le fonctionnement de la partie haute fréquence, utiliser un badge NFC ISO 14443-A vous appartenant.

Dans le client Proxmark3 :

```text
hf 14a reader
```

Approcher le badge de l'antenne HF.

Le Proxmark3 doit tenter de détecter le badge et afficher les informations disponibles.

Cette opération est une lecture non destructive.

> ⚠️ **Utilisation responsable**
>
> Effectuez les tests uniquement sur des badges et équipements vous appartenant ou pour lesquels vous disposez d'une autorisation.

### ✅ Résultat de notre installation

Lors de nos essais, nous avons confirmé :

- Le fonctionnement de la communication USB.
- Le chargement des images FPGA.
- Le démarrage du client Iceman.
- La communication avec le firmware ARM.
- La lecture NFC après flashage.

---

## 10. Utiliser le nouveau client avec la commande `pm3`

Nous allons maintenant simplifier le lancement du client Iceman.

L'objectif est de pouvoir exécuter :

```bash
pm3
```

depuis n'importe quel répertoire du terminal.

### 🔧 10.1 — Ajouter le nouveau dossier au PATH

Si une ancienne installation existe, nous souhaitons donner la priorité au nouveau client.

Exécuter :

```bash
if ! grep -Fqx 'export PATH="$HOME/proxmark3-new:$PATH"' ~/.bashrc; then
  echo 'export PATH="$HOME/proxmark3-new:$PATH"' >> ~/.bashrc
fi
```

Cette commande ajoute le nouveau dossier au `PATH` uniquement si la ligne n'existe pas déjà.

### 🔄 10.2 — Recharger la configuration

Exécuter :

```bash
source ~/.bashrc
```

### 🔍 10.3 — Vérifier la commande

Exécuter :

```bash
command -v pm3
```

Le résultat doit pointer vers le nouveau dossier.

Par exemple :

```text
/home/piuser/proxmark3-new/pm3
```

Le nom `piuser` est celui utilisé dans notre tutoriel.

Si vous avez choisi un autre utilisateur, le chemin sera différent.

### 💡 Conserver l'ancienne installation

Si une ancienne installation existe dans :

```text
~/proxmark3
```

Elle reste accessible explicitement avec :

```bash
~/proxmark3/pm3
```

> ⚠️ **Attention**
>
> Évitez d'utiliser un ancien client Iceman avec un firmware plus récent.
>
> Des différences de version peuvent provoquer des incompatibilités.

---

## 11. Utiliser Proxmark3 avec tmux

Au chapitre 2, nous avons installé `tmux` et créé l'alias :

```bash
pm
```

Cet alias permet de créer ou retrouver automatiquement une session persistante nommée `proxmark`.

### 💻 11.1 — Se connecter au Raspberry Pi

Depuis un ordinateur ou un smartphone disposant d'un client SSH :

```bash
ssh piuser@proxmark.local
```

### 🔄 11.2 — Ouvrir ou retrouver la session

Exécuter :

```bash
pm
```

Si une session existe déjà, elle sera retrouvée.

Sinon, une nouvelle session sera créée.

### 📡 11.3 — Lancer Proxmark3 Iceman

Dans la session `tmux` :

```bash
pm3
```

Le client Iceman doit démarrer.

### 🔌 11.4 — Détacher la session

Pour quitter temporairement sans arrêter le programme :

1. Appuyer sur `Ctrl + B`.
2. Relâcher les touches.
3. Appuyer sur `D`.

La session continue de fonctionner sur le Raspberry Pi.

### 🔄 11.5 — Reprendre le travail

Lors de la prochaine connexion SSH :

```bash
pm
```

Vous retrouverez la session précédente.

> 💡 **Important**
>
> `tmux` conserve les programmes exécutés dans sa session lorsque la connexion SSH est interrompue.
>
> Il ne maintient pas la connexion réseau elle-même.
>
> Le Raspberry Pi doit rester allumé et le Proxmark3 connecté.

Pour plus de détails, consultez :

[Étape 2 — Configuration du Raspberry Pi et utilisation de tmux](02-installation-raspberry.md)

---

## 12. Vérification avant de continuer

Avant de considérer l'installation comme terminée, assurez-vous que :

- [ ] Le Proxmark3 Easy est détecté par le Raspberry Pi.
- [ ] Les dépendances de compilation sont installées.
- [ ] Le dépôt Iceman a été téléchargé.
- [ ] Le fichier `Makefile.platform` correspond au matériel.
- [ ] La compilation s'est terminée sans erreur.
- [ ] L'ancienne installation a été sauvegardée si nécessaire.
- [ ] Le firmware ARM a été mis à jour avec succès, si un flashage était nécessaire.
- [ ] Le bootloader existant a été conservé.
- [ ] Le client Iceman démarre correctement.
- [ ] Les commandes `hw version` et `hw status` fonctionnent.
- [ ] Un test NFC autorisé a été effectué.
- [ ] La commande `pm3` pointe vers le nouveau client.
- [ ] La commande `pm` permet de retrouver une session `tmux`.

---

## ✅ Étape 3 terminée — Proxmark3 Iceman installé

Le Raspberry Pi Zero 2 W est maintenant configuré pour utiliser le Proxmark3 Easy à distance.

Nous disposons désormais :

- D'un Raspberry Pi accessible en Wi-Fi et SSH.
- D'un Proxmark3 Easy connecté par USB.
- Du client et du firmware Iceman.
- D'une configuration adaptée à notre matériel.
- D'une installation précédente conservée si nécessaire.
- D'une commande `pm3` pour lancer le client.
- D'une commande `pm` pour retrouver une session persistante.

### 🚀 Pour aller plus loin

Le système est maintenant prêt pour une utilisation à distance depuis un ordinateur ou un smartphone.

Les prochains chapitres pourront notamment aborder :

- L'utilisation quotidienne du Proxmark3.
- Les connexions SSH depuis différents appareils.
- Les outils de diagnostic.
- Les opérations de maintenance.
- Les procédures de sauvegarde et de mise à jour.

---

## 📚 Références

- [Dépôt officiel Proxmark3 Iceman — RfidResearchGroup](https://github.com/RfidResearchGroup/proxmark3)
- [Étape 1 — Matériel et branchements](01-materiel.md)
- [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md)

---

## 🧭 Navigation

| ⬅️ Étape précédente | 🏠 Sommaire | 🌐 English |
|---|---|---|
| [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md) | [Retour à l'accueil](../../README.fr.md) | [Step 3 — Proxmark3 Iceman Installation](../en/03-proxmark3-installation.md) |

---

📡 **Projet : Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
