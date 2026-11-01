# 🔧 Installation de Proxmark3 Iceman

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](../en/03-proxmark3-installation.md)

[🏠 Retour à l'accueil](../../README.fr.md)

---

## 1. Présentation

Ce chapitre explique comment installer, compiler et mettre à jour le firmware **Proxmark3 Iceman** sur un Raspberry Pi Zero 2 W.

Le Proxmark3 Easy est connecté au Raspberry Pi par USB. Son interface de commande est accessible à distance via SSH.

La procédure présentée a été testée sur le matériel utilisé dans ce projet.

### Configuration de référence

| Élément               | Configuration                   |
| --------------------- | ------------------------------- |
| Raspberry Pi          | Zero 2 W                        |
| Système               | Raspberry Pi OS Lite 64 bits    |
| Architecture          | ARM64 / aarch64                 |
| Proxmark3             | Easy                            |
| Microcontrôleur       | AT91SAM7S512                    |
| Mémoire flash interne | 512 Ko                          |
| FPGA                  | Xilinx XC2S30                   |
| Firmware              | Iceman — RfidResearchGroup      |
| Connexion             | USB via adaptateur OTG alimenté |
| Accès distant         | SSH                             |

### Objectifs

Nous allons :

1. Vérifier la connexion USB.
2. Installer les outils nécessaires.
3. Télécharger le firmware Iceman.
4. Configurer la compilation pour le Proxmark3 Easy.
5. Compiler le client et le firmware.
6. Sauvegarder une éventuelle ancienne installation.
7. Flasher le nouveau firmware.
8. Vérifier son fonctionnement.
9. Simplifier le lancement du client.

> ⚠️ **Attention :** le flashage comporte un risque si un firmware incompatible est utilisé. Vérifiez votre modèle et ne débranchez jamais le Proxmark3 pendant l'écriture.

---

## 2. Vérifier la connexion USB

Le Proxmark3 Easy doit être relié au port USB de données du Raspberry Pi Zero 2 W.

Le branchement est détaillé dans le [chapitre 1 — Matériel nécessaire et branchements](01-materiel.md).

### Détecter le périphérique

Depuis le terminal SSH :

```bash
lsusb
```

Puis :

```bash
ls /dev/ttyACM*
```

Dans notre installation, le Proxmark3 apparaît sous :

```text
/dev/ttyACM0
```

Le numéro du périphérique peut varier selon les équipements connectés.

Si aucun périphérique n'apparaît, vérifier le câble USB, l'adaptateur OTG et l'alimentation.

---

## 3. Installer les outils nécessaires

La compilation d'Iceman nécessite plusieurs outils de développement.

### Mettre à jour le système

```bash
sudo apt update
```

### Installer les dépendances

```bash
sudo apt install -y git build-essential make gcc-arm-none-eabi \
  libreadline-dev libusb-0.1-4-dev libpcsclite-dev pkg-config \
  libbz2-dev liblz4-dev libssl-dev python3 python3-dev
```

Les dépendances peuvent évoluer avec les nouvelles versions d'Iceman.

Si la compilation signale une bibliothèque manquante, consulter les instructions du dépôt officiel.

### Vérifier le compilateur ARM

```bash
arm-none-eabi-gcc --version
```

Le compilateur doit afficher sa version.

---

## 4. Télécharger Proxmark3 Iceman

Le firmware utilisé est développé et maintenu par la communauté **RfidResearchGroup**.

🔗 [Dépôt officiel Proxmark3 Iceman](https://github.com/RfidResearchGroup/proxmark3)

### Télécharger le code source

Depuis le dossier personnel :

```bash
cd ~
```

Cloner le dépôt :

```bash
git clone https://github.com/RfidResearchGroup/proxmark3.git proxmark3-new
```

Entrer dans le dossier :

```bash
cd ~/proxmark3-new
```

### Vérifier la version téléchargée

```bash
git log -1 --oneline
```

Cette commande affiche le dernier commit du dépôt.

### Version utilisée pour ce projet

Notre installation a été réalisée avec le commit :

```text
754bbae3d
```

Daté du 30 octobre 2026.

Les versions ultérieures peuvent présenter des différences.

> 💡 **Conseil :** si une ancienne installation existe déjà dans `~/proxmark3`, conserver ce dossier et utiliser `~/proxmark3-new` pour la nouvelle version.

---

## 5. Configurer le Proxmark3 Easy

Le firmware doit être compilé pour le matériel utilisé.

### Créer le fichier de configuration

Depuis le dossier du projet :

```bash
cd ~/proxmark3-new
```

Créer le fichier `Makefile.platform` :

```bash
cat > Makefile.platform <<'EOF'
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
EOF
```

### Explication des paramètres

| Paramètre | Valeur     | Description                             |
| --------- | ---------- | --------------------------------------- |
| PLATFORM  | PM3GENERIC | Plateforme Proxmark3 générique          |
| LED_ORDER | PM3EASY    | Configuration des LED du Proxmark3 Easy |

Cette configuration correspond au Proxmark3 Easy utilisé dans ce projet.

Notre compilation n'activait pas l'option `PLATFORM_EXTRAS=FLASH`, même si une mémoire flash externe était détectée.

> ⚠️ **Important :** ne pas utiliser automatiquement cette configuration pour un autre modèle de Proxmark3.

---

## 6. Compiler le firmware

La compilation peut prendre plusieurs minutes sur un Raspberry Pi Zero 2 W.

### Lancer la compilation

```bash
cd ~/proxmark3-new
```

Puis :

```bash
make -j2
```

L'option `-j2` permet d'utiliser deux tâches de compilation simultanées.

Ce réglage limite la charge sur le Raspberry Pi Zero 2 W.

### Vérifier le résultat

Une fois la compilation terminée :

```bash
echo $?
```

Un résultat de `0` indique que la dernière commande s'est terminée sans erreur.

### Fichiers générés

| Fichier                    | Description                   |
| -------------------------- | ----------------------------- |
| `armsrc/obj/fullimage.elf` | Firmware ARM                  |
| `bootrom/obj/bootrom.elf`  | Bootloader                    |
| `pm3`                      | Script de lancement du client |
| `pm3-flash`                | Outil de flashage             |

> ⚠️ **Attention :** la présence d'un nouveau bootloader compilé ne signifie pas qu'il faut obligatoirement l'installer.

Dans notre projet, nous avons conservé l'ancien bootloader.

---

## 7. Sauvegarder l'ancienne installation

Cette étape concerne les utilisateurs possédant déjà une installation fonctionnelle d'Iceman.

Pour une première installation, passer directement à l'étape suivante.

### Créer une archive

Depuis le dossier personnel :

```bash
cd ~
```

Puis :

```bash
tar -czf proxmark3-backup-$(date +%F).tar.gz proxmark3
```

Cette commande crée une archive compressée du dossier `proxmark3`.

### Vérifier l'archive

```bash
tar -tzf proxmark3-backup-$(date +%F).tar.gz > /dev/null && echo "Archive OK"
```

Si tout fonctionne, le terminal affiche :

```text
Archive OK
```

### Important

Cette sauvegarde contient les fichiers logiciels de l'ancienne installation.

Elle ne constitue **pas une sauvegarde complète de la mémoire interne du Proxmark3**.

Conserver séparément les données personnelles nécessaires.

Ne jamais publier de clés ou de dumps RFID personnels sur GitHub.

---

## 8. Flasher le nouveau firmware

Nous allons maintenant transférer le nouveau firmware vers le Proxmark3 Easy.

Dans notre installation, nous avons choisi de **mettre à jour uniquement le firmware ARM**, sans modifier le bootloader.

### Vérifier la détection du Proxmark3

```bash
cd ~/proxmark3-new
```

Puis :

```bash
./pm3-flash --list
```

Le Proxmark3 doit être détecté.

Dans notre cas :

```text
1: /dev/ttyACM0
```

### Lancer le flashage

```bash
./pm3-flash armsrc/obj/fullimage.elf
```

Attendre la fin de l'opération.

### Résultat attendu

```text
Flashing [===============================================================] 100%

[+] All done
[=] Have a nice day!
```

Le firmware est maintenant installé.

### Pourquoi conserver le bootloader ?

Le bootloader est le programme qui permet notamment de charger le firmware.

Lors de notre mise à jour :

| Composant    | Version                           |
| ------------ | --------------------------------- |
| Client       | Octobre 2026                      |
| Firmware ARM | Octobre 2026                      |
| Images FPGA  | Novembre 2026, compilation locale |
| Bootloader   | Juin 2026, conservé               |

Cette combinaison a fonctionné sur notre matériel.

> ⚠️ **Important :** ne pas ajouter l'option `-b` à la commande de flashage. Cette option autorise l'écriture dans la zone du bootloader.

Ne jamais interrompre l'alimentation pendant le flashage.

---

## 9. Vérifier le fonctionnement

Après le flashage, nous allons vérifier que le client communique correctement avec le Proxmark3.

### Lancer le nouveau client

```bash
cd ~/proxmark3-new
```

Puis :

```bash
./pm3
```

L'interface du client Proxmark3 doit apparaître.

### Vérifier les versions

Dans le terminal Proxmark3 :

```text
hw version
```

Cette commande affiche les versions du client, du firmware ARM, du bootloader et des images FPGA.

Le client et le firmware ARM doivent correspondre à la nouvelle version.

Le bootloader peut afficher une version plus ancienne si celui-ci a été conservé.

### Vérifier l'état du matériel

```text
hw status
```

Cette commande permet notamment de vérifier :

* La mémoire disponible.
* La configuration FPGA.
* Les paramètres radio.
* La communication entre le Raspberry Pi et le Proxmark3.

### Tester la lecture NFC

Placer un badge NFC personnel compatible ISO 14443-A sur l'antenne.

Exécuter :

```text
hf 14a reader
```

Le Proxmark3 doit détecter le badge et afficher ses informations disponibles.

Cette commande effectue une lecture sans modifier le badge.

### Résultat de notre installation

Les tests réalisés ont confirmé :

* Le fonctionnement du nouveau client.
* La communication USB.
* Le chargement des images FPGA.
* La lecture NFC.
* La compatibilité avec le bootloader conservé.

> 💡 La mention `suspect` peut apparaître dans les informations de version. Elle ne signifie pas, à elle seule, que le matériel est défectueux.

---

## 10. Simplifier le lancement de Proxmark3

Après l'installation, nous souhaitons pouvoir lancer le nouveau client depuis n'importe quel dossier avec une seule commande :

```bash
pm3
```

### Vérifier la commande actuelle

Quitter d'abord le client Proxmark3 :

```text
quit
```

Dans le terminal Linux :

```bash
type -a pm3
```

Si une ancienne installation existe, la commande peut encore pointer vers :

```text
/home/piuser/proxmark3/pm3
```

### Donner la priorité à la nouvelle installation

Nous allons ajouter le nouveau dossier au début de la variable `PATH`.

Pour éviter les doublons :

```bash
if ! grep -Fqx 'export PATH="$HOME/proxmark3-new:$PATH"' ~/.bashrc; then
  echo 'export PATH="$HOME/proxmark3-new:$PATH"' >> ~/.bashrc
fi
```

Recharger la configuration :

```bash
source ~/.bashrc
```

### Vérifier

```bash
command -v pm3
```

Résultat attendu :

```text
/home/piuser/proxmark3-new/pm3
```

Remplacer `piuser` par le nom de votre utilisateur.

### Utilisation quotidienne

Désormais, une seule commande suffit :

```bash
pm3
```

L'ancienne installation reste disponible dans son dossier d'origine.

> 💡 Pour conserver la session Proxmark3 lors d'une déconnexion SSH, utiliser `tmux` et la commande simplifiée `pm`, présentés dans le [chapitre 2 — Installation de Raspberry Pi OS](02-installation-raspberry.md).

---

## 11. Récapitulatif des commandes

| Commande             | Fonction                                                 |
| -------------------- | -------------------------------------------------------- |
| `pm3`                | Lancer le client Proxmark3                               |
| `hw version`         | Afficher les versions                                    |
| `hw status`          | Vérifier l'état du matériel                              |
| `hf 14a reader`      | Détecter un badge NFC compatible                         |
| `./pm3-flash --list` | Détecter le Proxmark3 pour le flashage                   |
| `make -j2`           | Compiler Iceman                                          |
| `pm`                 | Créer ou retrouver une session tmux, après configuration |

Les commandes `hw version`, `hw status` et `hf 14a reader` s'exécutent dans le client Proxmark3.

Les autres commandes s'exécutent dans le terminal Linux.

---

## ✅ Installation terminée

Le Proxmark3 Easy fonctionne maintenant avec le firmware Iceman installé sur le Raspberry Pi Zero 2 W.

Nous disposons désormais :

* D'un firmware Iceman récent.
* D'un client compilé pour Raspberry Pi OS 64 bits.
* D'une communication USB fonctionnelle.
* D'une lecture RFID/NFC opérationnelle.
* D'une ancienne installation conservée en sauvegarde.
* D'un lancement simplifié avec `pm3`.
* De sessions SSH persistantes grâce à `tmux`.

---

## ➡️ Étape suivante

**Utilisation du Proxmark3 Easy à distance**

Nous verrons comment accéder au Proxmark3 depuis un ordinateur ou un smartphone, utiliser les principales commandes et organiser les sessions de travail.

[🏠 Retour à l'accueil](../../README.fr.md)
