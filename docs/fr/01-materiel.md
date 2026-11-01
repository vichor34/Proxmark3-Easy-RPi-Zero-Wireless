# 🔧 Étape 1 — Matériel nécessaire et branchements

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](../en/01-hardware.md)

[🏠 Retour au sommaire](../../README.fr.md)

---

## 📖 Présentation

Ce premier chapitre présente le matériel nécessaire pour transformer un **Proxmark3 Easy** en outil RFID/NFC accessible à distance grâce à un **Raspberry Pi Zero 2 W**.

Le Raspberry Pi assure la communication avec le Proxmark3 via USB, tandis qu'un smartphone ou un ordinateur permet de contrôler l'ensemble à distance grâce à une connexion Wi-Fi et SSH.

Le montage utilise un **adaptateur USB OTG avec alimentation externe**, permettant de connecter le Proxmark3 au Raspberry Pi tout en alimentant le système.

### 🎯 Objectifs de cette étape

À la fin de ce chapitre, vous saurez :

- Identifier les composants nécessaires au projet.
- Comprendre le rôle de l'adaptateur USB OTG alimenté.
- Réaliser correctement les branchements.
- Identifier les ports USB du Raspberry Pi Zero 2 W.
- Vérifier l'alimentation du montage.
- Préparer le matériel pour l'installation de Raspberry Pi OS.

---

## 1. Matériel nécessaire

### 📦 Liste des composants

| Composant | Quantité | Description |
|---|:---:|---|
| Raspberry Pi Zero 2 W | 1 | Ordinateur miniature avec Wi-Fi intégré |
| Proxmark3 Easy | 1 | Outil d'analyse RFID/NFC |
| Adaptateur USB OTG alimenté | 1 | Adaptateur USB avec alimentation externe |
| Câble USB-A vers micro-USB | 1 | Connexion du Proxmark3 à l'adaptateur OTG |
| Alimentation USB 5 V | 1 | Alimentation électrique du montage |
| Carte microSD | 1 | 16 Go minimum recommandés |
| Smartphone ou ordinateur | 1 | Accès à distance via SSH |

### 💡 Pourquoi utiliser un Raspberry Pi Zero 2 W ?

Le Raspberry Pi Zero 2 W présente plusieurs avantages :

- Format compact et faible encombrement.
- Wi-Fi 2,4 GHz intégré.
- Faible consommation électrique.
- Compatibilité avec Raspberry Pi OS.
- Possibilité d'administration à distance via SSH.
- Fonctionnement sans écran, clavier ni souris.

Il constitue ainsi une solution adaptée pour réaliser un système RFID portable et sans fil.

---

## 2. Adaptateur USB OTG avec alimentation

### 🔌 Présentation

Le Raspberry Pi Zero 2 W dispose d'un port USB compatible OTG (*On-The-Go*).

L'adaptateur utilisé dans ce projet possède trois connecteurs :

| Connecteur | Fonction |
|---|---|
| Micro-USB mâle | Connexion au port USB du Raspberry Pi |
| USB-A femelle | Connexion du Proxmark3 Easy |
| Micro-USB femelle | Connexion de l'alimentation externe 5 V |

Cet adaptateur permet de conserver un montage compact sans utiliser de hub USB classique.

### ⚠️ Attention à la compatibilité

Tous les adaptateurs USB OTG alimentés ne fonctionnent pas de la même manière.

Certains modèles :

- Ne transmettent pas correctement les données USB.
- N'alimentent pas le Raspberry Pi par le port USB.
- Ne permettent pas l'utilisation simultanée de l'alimentation et des données.
- Présentent des incompatibilités avec le fonctionnement USB OTG.

Il est donc important de choisir un adaptateur adapté et de vérifier son fonctionnement.

> ⚠️ **Important**
>
> La présence d'une prise d'alimentation sur un adaptateur OTG ne garantit pas que celle-ci alimente également le Raspberry Pi.
>
> Le montage présenté correspond à la configuration utilisée pour ce projet. Le fonctionnement d'autres adaptateurs n'est pas garanti.

---

## 3. Schéma de branchement

Le montage repose sur une connexion USB directe entre le Raspberry Pi et le Proxmark3 Easy.

L'alimentation 5 V est raccordée à l'adaptateur USB OTG alimenté.

### 🔗 Schéma de connexion

```text
                Alimentation USB 5 V
                         |
                         |
                 Micro-USB femelle
                         |
              ┌────────────────────┐
              │    Adaptateur      │
              │   USB OTG alimenté │
              └────────────────────┘
                   |           |
                   |           |
             Micro-USB       USB-A
               mâle         femelle
                   |           |
                   |           |
            ┌────────────┐    |
            │ Raspberry  │    |
            │ Pi Zero 2 W│    |
            │            │    |
            │ Port USB   │    |
            └────────────┘    |
                              |
                       Câble USB-A
                       vers micro-USB
                              |
                              |
                       ┌────────────┐
                       │ Proxmark3  │
                       │    Easy    │
                       └────────────┘
```

### 🛠️ Procédure de connexion

**Avant de commencer, débranchez l'alimentation électrique.**

1. Insérer la carte microSD dans le Raspberry Pi Zero 2 W.
2. Identifier le port **USB** du Raspberry Pi.
3. Brancher le connecteur micro-USB mâle de l'adaptateur OTG sur ce port.
4. Brancher le câble USB du Proxmark3 Easy sur la prise USB-A femelle de l'adaptateur.
5. Connecter l'autre extrémité du câble au Proxmark3 Easy.
6. Brancher l'alimentation 5 V sur la prise micro-USB femelle de l'adaptateur OTG.
7. Vérifier que le Raspberry Pi démarre et que le Proxmark3 est alimenté.

> 💡 **Conseil**
>
> Si Raspberry Pi OS n'est pas encore installé sur la carte microSD, le système ne pourra pas démarrer normalement.
>
> L'installation du système sera réalisée dans l'étape 2.

---

## 4. Points importants

### 🔌 4.1 — Ports USB du Raspberry Pi

Le Raspberry Pi Zero 2 W possède deux connecteurs micro-USB :

| Port | Fonction |
|---|---|
| USB | Port de données compatible USB OTG |
| PWR IN | Entrée d'alimentation électrique |

L'adaptateur OTG doit impérativement être connecté au port **USB** pour permettre la communication avec le Proxmark3.

Le port **PWR IN** est destiné à l'alimentation et ne permet pas la communication USB avec le Proxmark3.

> ⚠️ **Attention**
>
> Ne confondez pas les ports **USB** et **PWR IN**.
>
> Un adaptateur OTG connecté au port PWR IN ne permettra pas au Raspberry Pi de communiquer avec le Proxmark3.

### ⚡ 4.2 — Alimentation électrique

Utiliser une alimentation 5 V régulée, capable de fournir suffisamment de courant pour le Raspberry Pi et le Proxmark3.

Une alimentation **5 V / 2,5 A** constitue un bon point de départ, sous réserve de la compatibilité et des caractéristiques de l'adaptateur OTG.

Éviter les alimentations de mauvaise qualité, susceptibles de provoquer :

- Des déconnexions USB.
- Des redémarrages du Raspberry Pi.
- Des problèmes de communication avec le Proxmark3.
- Une instabilité générale du système.

### 🔧 4.3 — Compatibilité du montage

Le montage décrit correspond à la configuration matérielle utilisée pour ce projet.

Le fonctionnement d'autres modèles d'adaptateurs OTG alimentés n'est pas garanti.

Avant de poursuivre, vérifier que :

- Le Raspberry Pi reçoit une alimentation suffisante.
- Le Proxmark3 Easy est correctement connecté.
- Les connecteurs USB sont bien insérés.
- Le montage ne présente pas de déconnexions ou de redémarrages intempestifs.

---

## 5. Vérification du matériel

Une fois Raspberry Pi OS installé et le système démarré, il sera possible de vérifier que le Proxmark3 est correctement détecté.

Ces vérifications nécessitent un accès au terminal du Raspberry Pi.

La configuration du système et de la connexion SSH sera expliquée dans l'étape 2.

### 🔍 5.1 — Vérifier les périphériques USB

Exécuter la commande suivante :

```bash
lsusb
```

Cette commande affiche les périphériques USB détectés par le Raspberry Pi.

Le Proxmark3 doit apparaître dans la liste des périphériques connectés lorsqu'il est correctement reconnu.

### 🔍 5.2 — Vérifier le port série

Pour rechercher les ports série USB de type ACM :

```bash
ls /dev/ttyACM*
```

Selon le firmware et la configuration, le Proxmark3 peut apparaître sous :

```text
/dev/ttyACM0
```

Le numéro du port peut varier selon les périphériques connectés.

> 💡 **Remarque**
>
> L'absence de `/dev/ttyACM0` ne signifie pas nécessairement que le Proxmark3 est défectueux.
>
> La détection dépend notamment du firmware installé et du mode de fonctionnement du périphérique.
>
> Les vérifications complémentaires seront abordées lors de l'installation du logiciel Proxmark3 Iceman.

---

## 6. Vérification avant de continuer

Avant de passer à l'étape suivante, assurez-vous que :

- [ ] Tous les composants nécessaires sont disponibles.
- [ ] La carte microSD est prête à être utilisée.
- [ ] L'adaptateur OTG est connecté au port USB du Raspberry Pi.
- [ ] Le Proxmark3 Easy est correctement raccordé.
- [ ] L'alimentation 5 V est adaptée au montage.
- [ ] Les branchements ont été vérifiés.

---

## ✅ Étape 1 terminée — Matériel et branchements

Le matériel nécessaire est maintenant identifié et le montage USB est prêt.

Nous pouvons passer à l'installation et à la configuration du système d'exploitation du Raspberry Pi.

Dans l'étape suivante, nous allons :

- Installer Raspberry Pi OS Lite (64 bits).
- Configurer la carte microSD.
- Activer l'accès SSH.
- Configurer la connexion Wi-Fi.
- Effectuer le premier démarrage.
- Vérifier le fonctionnement du système.

---

## 🧭 Navigation

| 🏠 Sommaire | ➡️ Étape suivante |
|---|---|
| [Retour à l'accueil](../../README.fr.md) | [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md) |

---

📡 **Projet : Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
