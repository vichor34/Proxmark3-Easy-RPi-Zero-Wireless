# 📡 Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](README.md)

---

## 📖 Présentation

**Transformez votre Proxmark3 Easy en outil RFID portable et sans fil grâce à un Raspberry Pi Zero 2 W !**

Ce projet explique étape par étape comment connecter un **Proxmark3 Easy** à un **Raspberry Pi Zero 2 W** afin de le piloter à distance depuis un smartphone ou un ordinateur.

L'objectif est de créer un système RFID compact, portable et autonome, permettant d'exploiter les fonctionnalités du Proxmark3 sans avoir besoin de le connecter directement à un ordinateur.

Le Raspberry Pi assure la communication avec le Proxmark3, tandis que le smartphone ou l'ordinateur permet d'accéder à son interface de commande à travers une connexion Wi-Fi.

### ✨ Fonctionnalités

- 🔌 Connexion du Proxmark3 Easy au Raspberry Pi via USB.
- 💻 Installation du firmware et du client Proxmark3 Iceman.
- 📶 Communication sans fil via Wi-Fi.
- 📱 Pilotage depuis un iPhone ou un iPad grâce à une connexion SSH.
- 🤖 Compatibilité avec les smartphones Android.
- 🖥️ Accès distant depuis Windows, macOS et Linux.
- 🛠️ Exécution des commandes Proxmark3 à distance.
- 🔄 Utilisation de sessions persistantes avec tmux.
- 💾 Sauvegarde et maintenance de l'installation.
- 🎒 Possibilité de créer un système portable et autonome.
- 📶 **Hotspot Wi-Fi automatique** : connexion prioritaire aux réseaux connus et création d'un point d'accès autonome lorsqu'aucun réseau n'est disponible au démarrage.

---

## 🔧 Matériel nécessaire

| Matériel | Description |
|---|---|
| Raspberry Pi Zero 2 W | Ordinateur assurant le pilotage du Proxmark3 |
| Proxmark3 Easy | Lecteur et outil d'analyse RFID/NFC |
| Carte microSD | Stockage du système Raspberry Pi OS |
| Adaptateur USB OTG | Connexion du Proxmark3 au Raspberry Pi |
| Alimentation 5 V adaptée | Alimentation du Raspberry Pi et du Proxmark3 |
| Smartphone ou ordinateur | Interface de contrôle à distance |

> 💡 **Remarque**
>
> Ce tutoriel est basé sur un **Proxmark3 Easy générique** et un **Raspberry Pi Zero 2 W**.
>
> Certaines variantes matérielles du Proxmark3 peuvent nécessiter une configuration différente.
>
> Les caractéristiques du matériel et les précautions d'alimentation sont détaillées dans le premier chapitre.

---

## 🔗 Architecture du système

```text
       📱 iPhone / Android / PC
                  |
                  |
                Wi-Fi
                  |
                  |
         Raspberry Pi Zero 2 W
                  |
                  |
                 USB
                  |
                  |
            Proxmark3 Easy
                  |
                  |
               RFID / NFC
```

Le Raspberry Pi Zero 2 W communique directement avec le Proxmark3 Easy via USB.

L'utilisateur se connecte au Raspberry Pi en Wi-Fi et utilise une session SSH pour accéder au client Proxmark3 Iceman.

Les commandes sont exécutées sur le Raspberry Pi, puis transmises au Proxmark3.

### 🖥️ Accès distant et sessions persistantes

L'installation utilise principalement trois outils :

| Outil | Fonction |
|---|---|
| SSH | Accéder au Raspberry Pi à distance |
| tmux | Conserver une session de terminal active après une déconnexion SSH |
| Proxmark3 Iceman | Communiquer avec le Proxmark3 Easy |

Deux commandes facilitent l'utilisation quotidienne :

| Commande | Fonction |
|---|---|
| `pm` | Créer ou retrouver une session tmux nommée `proxmark` |
| `pm3` | Lancer le client Proxmark3 Iceman |

> 💡 **Important**
>
> Si une session tmux contient déjà un client Iceman actif, il n'est pas nécessaire de lancer une seconde instance avec `pm3`.
>
> Les sessions tmux survivent à une déconnexion SSH, mais pas au redémarrage du Raspberry Pi.

---

## 📚 Documentation

Le tutoriel est organisé en **6 chapitres**, disponibles en français et en anglais.

Chaque chapitre présente une étape de l'installation, de la configuration ou de l'utilisation du système.

| Étape | Chapitre | Description |
|---|---|---|
| 1 | [📖 Matériel nécessaire et branchements](docs/fr/01-materiel.md) | Présentation du matériel, des connexions USB et de l'alimentation |
| 2 | [💻 Installation du Raspberry Pi](docs/fr/02-installation-raspberry.md) | Installation de Raspberry Pi OS, configuration du Wi-Fi, de SSH et de tmux |
| 3 | [📡 Installation du Proxmark3 Iceman](docs/fr/03-installation-proxmark3.md) | Installation, compilation et configuration du client et du firmware Iceman |
| 4 | [🖥️ Utilisation quotidienne](docs/fr/04-utilisation-quotidienne.md) | Connexion SSH, utilisation de tmux, commandes Iceman et diagnostics |
| 5 | [📱 Connexion mobile](docs/fr/05-connexion-mobile.md) | Utilisation depuis un iPhone, un iPad ou un smartphone Android |
| 6 | [🔧 Maintenance, sauvegardes et mises à jour](docs/fr/06-maintenance.md) | Sauvegardes, mises à jour, restauration et dépannage |
| 7 | [Hotspot Wi-Fi automatique](docs/fr/07-hotspot-wifi.md) | Connexion aux réseaux connus ou création d'un hotspot autonome |

### 🌐 Documentation en anglais

La version anglaise complète est également disponible :

| Étape | English documentation |
|---|---|
| 1 | [Required Hardware and Connections](docs/en/01-hardware.md) |
| 2 | [Raspberry Pi Installation](docs/en/02-raspberry-installation.md) |
| 3 | [Proxmark3 Iceman Installation](docs/en/03-proxmark3-installation.md) |
| 4 | [Daily Usage](docs/en/04-daily-usage.md) |
| 5 | [Mobile Connection](docs/en/05-mobile-connection.md) |
| 6 | [Maintenance, Backups and Updates](docs/en/06-maintenance.md) |

---

## 🚀 Démarrage rapide

Pour réaliser l'installation complète, suivez les chapitres dans l'ordre.

### 1. Préparer le matériel

Connectez le Proxmark3 Easy au Raspberry Pi Zero 2 W à l'aide d'un adaptateur USB OTG.

Vérifiez que l'alimentation est adaptée aux besoins du système.

Consultez le [chapitre 1 — Matériel nécessaire et branchements](docs/fr/01-materiel.md).

### 2. Installer Raspberry Pi OS

Installez Raspberry Pi OS Lite 64 bits sur la carte microSD.

Configurez notamment :

- Le nom du Raspberry Pi.
- Le compte utilisateur.
- Le réseau Wi-Fi.
- L'accès SSH.
- L'environnement tmux.

Consultez le [chapitre 2 — Installation du Raspberry Pi](docs/fr/02-installation-raspberry.md).

### 3. Installer Proxmark3 Iceman

Installez et compilez le projet Iceman en utilisant la configuration adaptée au Proxmark3 Easy générique.

La configuration de référence utilise :

```text
PLATFORM=PM3GENERIC
LED_ORDER=PM3EASY
```

> ⚠️ **Attention**
>
> Le flashage du firmware doit être réalisé uniquement avec des fichiers compatibles avec votre matériel.
>
> Ce tutoriel privilégie la conservation du bootloader existant.
>
> Ne flashez pas le Proxmark3 sans avoir lu les précautions du chapitre 3.

Consultez le [chapitre 3 — Installation du Proxmark3 Iceman](docs/fr/03-installation-proxmark3.md).

### 4. Se connecter à distance

Une fois le système installé, connectez-vous au Raspberry Pi depuis un terminal :

```bash
ssh piuser@proxmark.local
```

> 💡 **Remarque**
>
> `piuser` et `proxmark` sont les valeurs utilisées comme exemples dans ce tutoriel.
>
> Adaptez-les à votre propre configuration.
>
> Si `proxmark.local` n'est pas accessible, utilisez l'adresse IP du Raspberry Pi.

Ouvrez ensuite la session tmux :

```bash
pm
```

Si Iceman n'est pas déjà actif dans cette session, lancez le client :

```bash
pm3
```

Vous pouvez alors utiliser les commandes Iceman.

Consultez le [chapitre 4 — Utilisation quotidienne](docs/fr/04-utilisation-quotidienne.md).

### 5. Utiliser un smartphone

Le système peut également être contrôlé depuis :

- Un iPhone.
- Un iPad.
- Un smartphone Android.

La connexion s'effectue en SSH, à travers un réseau Wi-Fi accessible aux deux appareils.

Le partage de connexion d'un smartphone peut également être utilisé, sous réserve des possibilités de communication entre appareils offertes par ce réseau.

Consultez le [chapitre 5 — Connexion mobile](docs/fr/05-connexion-mobile.md).

---

## 🛠️ Maintenance et sauvegardes

Une fois l'installation fonctionnelle, il est recommandé de conserver une sauvegarde de votre configuration.

Le tutoriel explique notamment comment :

- Vérifier l'état du Raspberry Pi.
- Contrôler les versions installées.
- Sauvegarder les paramètres réseau.
- Sauvegarder la configuration du client Iceman.
- Créer une image complète de la carte microSD.
- Mettre à jour Raspberry Pi OS.
- Préparer une mise à jour du client Iceman.
- Restaurer une installation précédente.
- Diagnostiquer les problèmes courants.

> ⚠️ **Important**
>
> La mise à jour de Raspberry Pi OS, la compilation du client Iceman et le flashage du firmware Proxmark3 sont des opérations différentes.
>
> Si votre installation fonctionne correctement, il n'est pas nécessaire de flasher régulièrement le Proxmark3.
>
> Les sauvegardes contenant des identifiants réseau doivent être conservées dans un emplacement privé et protégé.

Consultez le [chapitre 6 — Maintenance, sauvegardes et mises à jour](docs/fr/06-maintenance.md).

---

## 🚧 État du projet

🟢 **Les six chapitres du tutoriel sont disponibles en français et en anglais.**

Le projet repose sur un montage personnel utilisant un Raspberry Pi Zero 2 W et un Proxmark3 Easy générique.

La documentation décrit la configuration de référence, les procédures d'installation, l'utilisation à distance et les opérations de maintenance.

Les retours d'expérience, corrections et améliorations restent les bienvenus.

> 💡 **Remarque**
>
> Les commandes et procédures sont documentées pour la configuration matérielle présentée.
>
> Leur fonctionnement peut varier selon la version du matériel, du système d'exploitation ou du firmware utilisé.

---

## 🤝 Contributions

Les contributions sont les bienvenues !

Vous pouvez participer au projet en :

- Signalant des erreurs ou des problèmes.
- Proposant des améliorations.
- Partageant vos expériences.
- Contribuant à la traduction de la documentation.
- Proposant des corrections techniques.

N'hésitez pas à ouvrir une *Issue* ou à proposer une *Pull Request*.

---

## 📜 Licence et utilisation

© 2026 Victor HANSEMANN.

La documentation originale de ce projet est distribuée sous licence **Creative Commons BY-NC-SA 4.0** (Attribution – Pas d'Utilisation Commerciale – Partage dans les Mêmes Conditions).

Cette licence permet de consulter, partager et modifier la documentation, sous réserve de respecter ses conditions.

Une **autorisation complémentaire gratuite** permet également l'utilisation et l'adaptation de la documentation pour les besoins internes des entreprises, associations et établissements d'enseignement.

Les adaptations diffusées sous la licence Creative Commons doivent respecter les conditions de partage à l'identique.

Toute exploitation commerciale des contenus protégés dépassant les autorisations accordées nécessite un accord distinct.

La licence documentaire ne restreint pas, à elle seule, la fabrication ou la vente de matériels indépendants utilisant les connaissances techniques présentées.

### 📄 Documents de licence

- [Licence CC BY-NC-SA 4.0](LICENSE)
- [Autorisations complémentaires](PERMISSIONS.md)
- [Conditions d'exploitation commerciale](COMMERCIAL.md)
- [Contribuer au projet](CONTRIBUTING.md)

---

📡 **Projet : Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**

🌐 [🇬🇧 Read this documentation in English](README.md)
