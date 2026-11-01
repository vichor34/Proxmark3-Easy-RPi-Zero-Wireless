# 03 — Installer et mettre à jour Proxmark3 Iceman

[English version](../en/03-proxmark3-installation.md)

Ce chapitre explique comment compiler et utiliser le firmware **Iceman (RfidResearchGroup)** sur un **Proxmark3 Easy** relié en USB à un **Raspberry Pi Zero 2 W** (Raspberry Pi OS Lite 64 bits). Il décrit aussi une mise à jour prudente d'une installation existante, **sans écraser le bootloader**.

> **Attention :** le flashage comporte un risque de rendre le Proxmark3 inutilisable si le mauvais firmware ou les mauvais paramètres sont employés. Vérifiez votre modèle avant toute écriture. Ne débranchez pas l'appareil pendant le flashage.

## 1. Vérifier la connexion USB

Branchez le Proxmark3 Easy sur le port USB de données du Raspberry Pi, avec un adaptateur OTG et une alimentation adaptés.

```bash
ls -l /dev/ttyACM*
```

Dans notre installation, le Proxmark3 apparaît sous `/dev/ttyACM0`. Le numéro peut varier.

## 2. Installer les outils de compilation

Mettez le système à jour et installez les outils principaux :

```bash
sudo apt update
sudo apt install -y git build-essential make gcc-arm-none-eabi \
  libreadline-dev libusb-0.1-4-dev libpcsclite-dev pkg-config \
  libbz2-dev liblz4-dev libssl-dev python3 python3-dev
```

La liste exacte des dépendances peut évoluer avec Iceman. Consultez les instructions de compilation du dépôt officiel si `make` signale une bibliothèque manquante.

## 3. Télécharger Iceman

Pour une **première installation**, utilisez un dossier `~/proxmark3-new` :

```bash
cd ~
git clone https://github.com/RfidResearchGroup/proxmark3.git proxmark3-new
cd ~/proxmark3-new
```

Pour une **mise à jour d'une installation existante**, conservez l'ancien dossier `~/proxmark3` et clonez le nouveau dépôt séparément. Cela permet de revenir aux anciens fichiers si nécessaire.

Dans notre installation testée, le nouveau code correspondait au commit `754bbae3d` (30 octobre 2026). Les versions ultérieures peuvent présenter des différences.

## 4. Configurer le Proxmark3 Easy

Créez le fichier `Makefile.platform` à la racine du nouveau dépôt :

```bash
cat > Makefile.platform <<'EOF_CONFIG'
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
EOF_CONFIG
```

Ces paramètres correspondent au **Proxmark3 Easy générique** utilisé dans ce projet (microcontrôleur AT91SAM7S512 et FPGA `xc2s30`). **Ne copiez pas ces réglages sur un autre modèle sans vérification.**

Notre compilation n'activait pas `PLATFORM_EXTRAS=FLASH`, même si le matériel détectait une mémoire flash externe.

## 5. Compiler le client et le firmware

```bash
cd ~/proxmark3-new
make -j2
```

Le Raspberry Pi Zero 2 W dispose de ressources limitées : `-j2` permet de limiter le parallélisme. Attendez la fin de la compilation et vérifiez qu'aucune erreur n'est affichée.

Les fichiers générés comprennent notamment :

- `armsrc/obj/fullimage.elf` : firmware ARM ;
- `bootrom/obj/bootrom.elf` : bootloader ;
- `pm3` : script de lancement du client.

**Ne flashez pas automatiquement le bootloader** simplement parce qu'il a été compilé.

## 6. Sauvegarder une installation précédente (mise à jour)

Si une ancienne installation existe dans `~/proxmark3`, vous pouvez archiver son contenu avant toute opération :

```bash
cd ~
tar -czf proxmark3-backup-$(date +%F).tar.gz proxmark3
```

Cette archive sauvegarde les **fichiers logiciels**, pas une image de la mémoire interne du Proxmark3. Conservez séparément vos données et configurations personnelles ; ne publiez jamais de dumps RFID ni de clés sur GitHub.

## 7. Vérifier le périphérique et flasher uniquement le firmware ARM

Depuis le **nouveau dépôt** :

```bash
cd ~/proxmark3-new
./pm3-flash --list
```

Vérifiez que votre appareil est détecté (dans notre cas `/dev/ttyACM0`). Puis lancez :

```bash
./pm3-flash armsrc/obj/fullimage.elf
```

Attendez la fin du transfert et le message `All done`.

**Important :** n'utilisez **pas** l'option `-b` dans cette procédure : elle autorise l'écriture dans la zone du bootloader. Notre mise à jour réussie a remplacé uniquement le firmware ARM, en conservant le bootloader précédent.

## 8. Vérifier le fonctionnement

Lancez le nouveau client :

```bash
cd ~/proxmark3-new
./pm3
```

Dans l'invite Proxmark3 :

```text
hw version
hw status
```

Vérifiez que les versions **Client** et **OS** correspondent au nouveau code. Le **Bootrom** peut afficher une version plus ancienne : c'est attendu si vous l'avez conservé.

Testez ensuite, avec **votre propre badge NFC ISO 14443-A**, une lecture non destructive :

```text
hf 14a reader
```

Dans notre test, la communication USB, les images FPGA et la lecture NFC fonctionnaient après le flashage. La mention `suspect` dans une chaîne de version n'indique pas à elle seule une panne matérielle.

## 9. Utiliser le nouveau client avec la commande `pm3`

Si votre `PATH` contient déjà `~/proxmark3`, placez le nouveau dossier **avant** l'ancien. Pour Bash :

```bash
# Ajoute la ligne uniquement si elle n'existe pas déjà
if ! grep -Fqx 'export PATH="$HOME/proxmark3-new:$PATH"' ~/.bashrc; then
  echo 'export PATH="$HOME/proxmark3-new:$PATH"' >> ~/.bashrc
fi
source ~/.bashrc
command -v pm3
```

Le résultat attendu est :

```text
/home/victor/proxmark3-new/pm3
```

Remplacez `victor` par votre nom d'utilisateur si nécessaire. L'ancienne installation reste accessible explicitement via `~/proxmark3/pm3`, mais évitez d'utiliser son ancien client avec le nouveau firmware.

## 10. Conserver la session lors d'une coupure SSH

Pour installer **tmux** et créer la commande courte `pm`, consultez le [chapitre 2 — Configuration du Raspberry Pi](02-installation-raspberry.md). `tmux` maintient les programmes sur le Raspberry Pi lors d'une déconnexion SSH ; il ne maintient pas la connexion réseau elle-même.

## Référence

Code source officiel : [RfidResearchGroup/proxmark3](https://github.com/RfidResearchGroup/proxmark3).
