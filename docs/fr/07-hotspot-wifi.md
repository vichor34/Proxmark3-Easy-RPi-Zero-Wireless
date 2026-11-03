# 📶 Étape 7 — Hotspot Wi-Fi automatique

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](../en/07-wifi-hotspot.md)

[🏠 Retour au sommaire](../../README.fr.md)

---

## 📖 Présentation

Notre installation **Proxmark3 Easy + Raspberry Pi Zero 2 W** fonctionne maintenant en Wi-Fi et peut être utilisée à distance depuis un ordinateur ou un smartphone.

Cependant, une connexion à un réseau Wi-Fi existant présente une limitation : lorsque nous nous déplaçons, le Raspberry Pi peut ne trouver aucun réseau connu.

Nous allons résoudre ce problème en configurant un **point d'accès Wi-Fi automatique**, également appelé *hotspot*.

Le Raspberry Pi pourra ainsi fonctionner de deux manières :

* **Mode client Wi-Fi :** connexion automatique à un réseau connu.
* **Mode hotspot :** création de son propre réseau Wi-Fi lorsqu'aucun réseau connu n'est disponible au démarrage.

Cette fonctionnalité permet d'utiliser le Proxmark3 Easy de manière autonome, même sans routeur ni connexion Internet.

> 💡 **Important**
>
> Le hotspot est une fonctionnalité optionnelle.
>
> Si vous utilisez toujours votre Proxmark3 sur un réseau Wi-Fi existant, vous pouvez conserver la configuration des chapitres précédents.

### 🎯 Objectifs de cette étape

À la fin de ce chapitre, vous saurez :

* Vérifier la compatibilité du Raspberry Pi avec le mode hotspot.
* Configurer les priorités des réseaux Wi-Fi connus.
* Créer un point d'accès Wi-Fi sécurisé.
* Attribuer une adresse IP fixe au Raspberry Pi en mode hotspot.
* Configurer un script de basculement automatique.
* Exécuter ce script au démarrage avec systemd.
* Tester les deux modes de connexion.
* Accéder au Proxmark3 depuis un smartphone sans réseau extérieur.
* Diagnostiquer les problèmes de connexion.
* Désactiver la fonctionnalité si nécessaire.

---

## 1. Principe de fonctionnement

### 📡 1.1 — Les deux modes Wi-Fi

Le Raspberry Pi Zero 2 W possède une interface Wi-Fi intégrée.

Nous allons utiliser cette interface pour choisir automatiquement un mode de fonctionnement au démarrage.

| Situation                                      | Comportement                                 |
| ---------------------------------------------- | -------------------------------------------- |
| Réseau Wi-Fi connu disponible                  | Connexion au réseau                          |
| Plusieurs réseaux connus disponibles           | Sélection selon les priorités NetworkManager |
| Aucun réseau connu disponible                  | Création du hotspot                          |
| Réseau connu réapparaissant pendant le hotspot | Conservation du hotspot                      |
| Redémarrage du Raspberry Pi                    | Nouvelle recherche des réseaux connus        |

### 🔄 1.2 — Logique de démarrage

```text
             Démarrage du Raspberry Pi
                        |
                        v
             NetworkManager démarre
                        |
                        v
          Recherche des réseaux enregistrés
                        |
                        v
           Attente maximale de 30 secondes
                        |
               Réseau connecté ?
                  /         \
                OUI         NON
                 |           |
                 v           v
          Connexion au     Création du
          réseau connu     hotspot Wi-Fi
                 |           |
                 v           v
             Accès SSH   Accès SSH direct
                 |           |
                 +-----+-----+
                       |
                       v
                    tmux
                       |
                       v
                 Proxmark3 Iceman
```

### ⚠️ 1.3 — Limitation importante

Le Raspberry Pi Zero 2 W possède une seule interface Wi-Fi intégrée.

Dans cette configuration, nous utilisons donc **soit le mode client, soit le mode hotspot**.

Nous ne cherchons pas à maintenir simultanément les deux connexions.

Le choix du mode est effectué au démarrage.

**Le retour automatique d'un hotspot actif vers un réseau connu n'est pas prévu dans cette version.**

Pour revenir à un réseau connu, il suffit normalement de redémarrer le Raspberry Pi lorsque ce réseau est disponible.

> 💡 **Pourquoi ce choix ?**
>
> Cette méthode évite les recherches périodiques de réseaux susceptibles de perturber une connexion SSH en cours.
>
> Elle privilégie la simplicité et la stabilité.

---

## 2. Configuration de référence

Le tutoriel repose sur l'installation présentée dans les chapitres précédents.

| Élément                     | Configuration                |
| --------------------------- | ---------------------------- |
| Raspberry Pi                | Zero 2 W                     |
| Système                     | Raspberry Pi OS Lite 64 bits |
| Base Debian                 | Debian 13 (Trixie)           |
| Gestionnaire réseau         | NetworkManager               |
| Interface Wi-Fi             | `wlan0`                      |
| Réseau principal d'exemple  | `Home-WiFi`                  |
| Réseau secondaire d'exemple | `Phone-Hotspot`              |
| Nom du hotspot              | `Proxmark3-Wireless`         |
| Adresse IP du hotspot       | `192.168.50.1/24`            |
| Sécurité Wi-Fi              | WPA2-Personal                |
| Délai de recherche          | 30 secondes                  |
| Accès distant               | SSH                          |
| Sessions persistantes       | tmux                         |

### 📚 Prérequis

Avant de continuer, vous devez avoir terminé :

* [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md)
* [Étape 3 — Installation de Proxmark3 Iceman](03-installation-proxmark3.md)
* [Étape 4 — Utilisation quotidienne](04-utilisation-quotidienne.md)

Le Raspberry Pi doit déjà :

* Démarrer correctement.
* Se connecter à un réseau Wi-Fi.
* Être accessible en SSH.
* Utiliser NetworkManager.
* Permettre l'exécution du client Proxmark3 Iceman.

> ⚠️ **Attention**
>
> Nous allons modifier la configuration réseau.
>
> Une erreur peut interrompre l'accès SSH.
>
> Réalisez cette installation lorsque vous pouvez redémarrer le Raspberry Pi ou accéder directement à sa carte microSD.

---

## 3. Vérifier NetworkManager

### 🔍 3.1 — Vérifier les interfaces réseau

Connectez-vous au Raspberry Pi en SSH :

```bash
ssh piuser@proxmark.local
```

Affichez les interfaces :

```bash
nmcli device status
```

Exemple :

```text
DEVICE  TYPE      STATE      CONNECTION
wlan0   wifi      connected  Home-WiFi
lo      loopback  connected  lo
```

L'interface Wi-Fi utilisée dans ce tutoriel est :

```text
wlan0
```

> 💡 **Remarque**
>
> Si votre interface possède un autre nom, adaptez les commandes suivantes.

### 📡 3.2 — Vérifier la compatibilité hotspot

Exécutez :

```bash
nmcli -f WIFI-PROPERTIES.AP device show wlan0
```

Résultat attendu :

```text
WIFI-PROPERTIES.AP: yes
```

La valeur `yes` indique que NetworkManager détecte la prise en charge du mode point d'accès.

> ⚠️ **Important**
>
> Si le résultat est `no`, ne poursuivez pas cette configuration.
>
> Vérifiez la compatibilité du matériel et du pilote Wi-Fi.

### 🔍 3.3 — Afficher les connexions enregistrées

```bash
nmcli connection show
```

Exemple :

```text
NAME           TYPE      DEVICE
Home-WiFi      wifi      wlan0
Phone-Hotspot  wifi      --
lo             loopback  lo
```

Nous allons utiliser les réseaux enregistrés pour déterminer si le hotspot doit être activé.

---

## 4. Configurer les réseaux Wi-Fi connus

### 📶 4.1 — Comprendre les priorités

NetworkManager peut enregistrer plusieurs réseaux Wi-Fi.

Lorsqu'ils sont disponibles, il utilise notamment les priorités de connexion automatique pour sélectionner un profil.

Nous allons définir :

| Réseau          | Priorité |
| --------------- | -------: |
| `Home-WiFi`     |      100 |
| `Phone-Hotspot` |       50 |

Ces noms sont des exemples.

**Remplacez-les par les noms exacts de vos profils NetworkManager.**

### 🔧 4.2 — Configurer le réseau principal

```bash
sudo nmcli connection modify "Home-WiFi" \
  connection.autoconnect yes \
  connection.autoconnect-priority 100
```

### 🔧 4.3 — Configurer le réseau secondaire

```bash
sudo nmcli connection modify "Phone-Hotspot" \
  connection.autoconnect yes \
  connection.autoconnect-priority 50
```

Si vous ne possédez qu'un seul réseau enregistré, configurez uniquement celui-ci.

### 🔍 4.4 — Vérifier les priorités

Pour le réseau principal :

```bash
nmcli -f connection.id,connection.autoconnect,connection.autoconnect-priority \
  connection show "Home-WiFi"
```

Résultat attendu :

```text
connection.id:                   Home-WiFi
connection.autoconnect:          yes
connection.autoconnect-priority: 100
```

Répétez la vérification pour les autres réseaux.

> 💡 **Conseil**
>
> La priorité la plus élevée est préférée lorsque plusieurs profils sont éligibles.
>
> Cela ne garantit pas qu'un réseau indisponible sera toujours sélectionné : NetworkManager doit également pouvoir établir la connexion.

---

## 5. Créer le hotspot Wi-Fi

Nous allons maintenant créer le profil du point d'accès.

**À cette étape, nous préparons uniquement sa configuration. Nous ne l'activons pas.**

### 📡 5.1 — Créer le profil

Exécutez :

```bash
sudo nmcli connection add \
  type wifi \
  ifname wlan0 \
  con-name "Proxmark3-Wireless" \
  ssid "Proxmark3-Wireless"
```

Cette commande crée un nouveau profil NetworkManager.

### 🔧 5.2 — Configurer le mode point d'accès

```bash
sudo nmcli connection modify "Proxmark3-Wireless" \
  802-11-wireless.mode ap \
  802-11-wireless.band bg \
  ipv4.method shared \
  ipv4.addresses 192.168.50.1/24 \
  ipv6.method disabled \
  connection.autoconnect no
```

### 📖 Explication des paramètres

| Paramètre              | Fonction                                                       |
| ---------------------- | -------------------------------------------------------------- |
| `mode ap`              | Active le mode point d'accès                                   |
| `band bg`              | Utilise la bande Wi-Fi 2,4 GHz                                 |
| `ipv4.method shared`   | Active le partage IPv4 et l'attribution d'adresses aux clients |
| `192.168.50.1/24`      | Définit l'adresse IP du Raspberry Pi                           |
| `ipv6.method disabled` | Désactive IPv6 pour ce profil                                  |
| `autoconnect no`       | Empêche l'activation automatique du hotspot par NetworkManager |

> 💡 **Pourquoi désactiver `autoconnect` ?**
>
> Le hotspot ne doit pas concurrencer les réseaux habituels au démarrage.
>
> Son activation sera contrôlée par notre script.

### 🔐 5.3 — Configurer la sécurité

Définissez le type de sécurité :

```bash
sudo nmcli connection modify "Proxmark3-Wireless" \
  wifi-sec.key-mgmt wpa-psk
```

Choisissez ensuite un mot de passe Wi-Fi robuste.

Pour éviter de le saisir directement dans une commande enregistrée dans l'historique du terminal :

```bash
read -rsp "Mot de passe du hotspot : " HOTSPOT_PASSWORD
echo
```

Enregistrez-le :

```bash
sudo nmcli connection modify "Proxmark3-Wireless" \
  wifi-sec.psk "$HOTSPOT_PASSWORD"
```

Supprimez la variable temporaire :

```bash
unset HOTSPOT_PASSWORD
```

> 🔐 **Sécurité**
>
> Utilisez un mot de passe unique et suffisamment long.
>
> Ne publiez jamais votre véritable mot de passe dans un dépôt GitHub.
>
> Le hotspot permet d'accéder au Raspberry Pi : protégez également votre compte SSH.

### 🔍 5.4 — Vérifier le profil

Exécutez :

```bash
nmcli -f 802-11-wireless.ssid,802-11-wireless.mode,ipv4.method,ipv4.addresses,connection.autoconnect \
  connection show "Proxmark3-Wireless"
```

Vous devez retrouver :

| Paramètre   | Valeur               |
| ----------- | -------------------- |
| SSID        | `Proxmark3-Wireless` |
| Mode        | `ap`                 |
| IPv4        | `shared`             |
| Adresse     | `192.168.50.1/24`    |
| Autoconnect | `no`                 |

> ⚠️ **Attention**
>
> Ne lancez pas encore :
>
> `nmcli connection up "Proxmark3-Wireless"`
>
> Cette commande pourrait interrompre immédiatement votre connexion Wi-Fi et votre session SSH.

---

## 6. Créer le script de basculement automatique

Nous allons créer un script Bash qui vérifie si le Raspberry Pi est connecté à un réseau connu.

Si aucune connexion n'est établie après 30 secondes, il active le hotspot.

### 🛠️ 6.1 — Créer le fichier

Ouvrez :

```bash
sudo nano /usr/local/sbin/proxmark-wifi-fallback.sh
```

Collez le contenu suivant :

```bash
#!/bin/bash

set -euo pipefail

INTERFACE="wlan0"
HOTSPOT="Proxmark3-Wireless"
TIMEOUT=30

logger -t proxmark-wifi "Recherche d'un réseau Wi-Fi connu..."

# Attendre au maximum 30 secondes.
for ((i=0; i<TIMEOUT; i++)); do

    ACTIVE=$(nmcli -g GENERAL.CONNECTION device show "$INTERFACE" 2>/dev/null || true)

    if [[ "$ACTIVE" == "Home-WiFi" || "$ACTIVE" == "Phone-Hotspot" ]]; then
        logger -t proxmark-wifi "Connexion établie : $ACTIVE"
        exit 0
    fi

    sleep 1
done

# Dernière vérification.
ACTIVE=$(nmcli -g GENERAL.CONNECTION device show "$INTERFACE" 2>/dev/null || true)

if [[ "$ACTIVE" == "Home-WiFi" || "$ACTIVE" == "Phone-Hotspot" ]]; then
    exit 0
fi

logger -t proxmark-wifi "Aucun réseau connu : activation du hotspot."

# Empêcher les reconnexions automatiques de l'interface
# pendant la session hotspot.
nmcli device set "$INTERFACE" autoconnect no

# Activer le hotspot.
if nmcli connection up "$HOTSPOT" ifname "$INTERFACE"; then
    logger -t proxmark-wifi "Hotspot actif : $HOTSPOT"
else
    logger -t proxmark-wifi "ERREUR : activation du hotspot impossible."
    exit 1
fi
```

### ⚠️ 6.2 — Adapter les réseaux connus

Dans le script, recherchez :

```bash
if [[ "$ACTIVE" == "Home-WiFi" || "$ACTIVE" == "Phone-Hotspot" ]]; then
```

Cette condition apparaît **deux fois**.

Remplacez les noms d'exemple par ceux de vos profils Wi-Fi.

Si vous ne possédez qu'un réseau connu, utilisez dans les deux cas :

```bash
if [[ "$ACTIVE" == "Home-WiFi" ]]; then
```

> ⚠️ **Important**
>
> Le script compare les noms des **profils NetworkManager actifs**.
>
> Ils correspondent généralement aux noms des réseaux Wi-Fi, mais peuvent être différents.
>
> Vérifiez-les avec `nmcli connection show`.

### 💾 6.3 — Enregistrer le fichier

Dans nano :

1. Appuyez sur `Ctrl + O`.
2. Validez avec `Entrée`.
3. Appuyez sur `Ctrl + X`.

### 🔐 6.4 — Autoriser l'exécution

```bash
sudo chmod 755 /usr/local/sbin/proxmark-wifi-fallback.sh
```

### 🔍 6.5 — Vérifier la syntaxe

```bash
bash -n /usr/local/sbin/proxmark-wifi-fallback.sh
```

Si aucune erreur ne s'affiche, la syntaxe Bash est correcte.

> 💡 **Remarque**
>
> Cette vérification ne garantit pas que le hotspot fonctionnera.
>
> Elle permet uniquement de détecter certaines erreurs de syntaxe.

---

## 7. Créer le service systemd

Nous allons demander à systemd d'exécuter notre script automatiquement au démarrage du Raspberry Pi.

### 🛠️ 7.1 — Créer le service

Ouvrez :

```bash
sudo nano /etc/systemd/system/proxmark-wifi-fallback.service
```

Collez :

```ini
[Unit]
Description=Proxmark3 Wi-Fi fallback hotspot
Wants=NetworkManager.service
After=NetworkManager.service

[Service]
Type=oneshot
ExecStartPre=/usr/bin/nmcli device set wlan0 autoconnect yes
ExecStart=/usr/local/sbin/proxmark-wifi-fallback.sh
TimeoutStartSec=60

[Install]
WantedBy=multi-user.target
```

### 📖 7.2 — Comprendre le service

| Paramètre                      | Fonction                                                 |
| ------------------------------ | -------------------------------------------------------- |
| `Wants=NetworkManager.service` | Demande le démarrage de NetworkManager                   |
| `After=NetworkManager.service` | Lance le service après NetworkManager                    |
| `Type=oneshot`                 | Exécute une opération ponctuelle                         |
| `ExecStartPre`                 | Réactive la connexion automatique de `wlan0`             |
| `ExecStart`                    | Lance le script de basculement                           |
| `TimeoutStartSec=60`           | Définit une limite de temps pour le démarrage du service |
| `WantedBy=multi-user.target`   | Permet l'activation au démarrage normal                  |

### 🔄 7.3 — Recharger systemd

```bash
sudo systemctl daemon-reload
```

### ⚡ 7.4 — Activer le service

```bash
sudo systemctl enable proxmark-wifi-fallback.service
```

### 🔍 7.5 — Vérifier son activation

```bash
systemctl is-enabled proxmark-wifi-fallback.service
```

Résultat attendu :

```text
enabled
```

> ⚠️ **Attention**
>
> N'exécutez pas immédiatement `systemctl start` depuis une connexion SSH distante.
>
> Si aucun réseau connu n'est détecté, le script pourrait activer le hotspot et interrompre votre connexion.

---

## 8. Tester le fonctionnement avec un réseau connu

Nous allons commencer par vérifier que le Raspberry Pi conserve son comportement habituel.

### 📡 8.1 — Vérifier la connexion actuelle

```bash
nmcli device status
```

Vérifiez que `wlan0` est connecté à votre réseau habituel.

### 🔄 8.2 — Redémarrer

```bash
sudo reboot
```

Attendez environ une minute.

Reconnectez-vous ensuite :

```bash
ssh piuser@proxmark.local
```

Si nécessaire, utilisez l'adresse IP du Raspberry Pi sur votre réseau local.

### 🔍 8.3 — Vérifier le réseau actif

```bash
nmcli device status
```

Résultat attendu :

```text
wlan0  wifi  connected  Home-WiFi
```

### 📋 8.4 — Vérifier le service

```bash
systemctl status proxmark-wifi-fallback.service --no-pager
```

Puis :

```bash
journalctl -u proxmark-wifi-fallback.service -b --no-pager
```

Exemple de journal :

```text
Recherche d'un réseau Wi-Fi connu...
Connexion établie : Home-WiFi
Finished proxmark-wifi-fallback.service
```

L'état `inactive (dead)` est normal après l'exécution réussie d'un service `oneshot` sans option `RemainAfterExit`.

---

## 9. Tester le fonctionnement sans réseau connu

Nous allons maintenant vérifier que le hotspot s'active automatiquement.

### ⚠️ Précautions

Ce test peut rendre le Raspberry Pi temporairement inaccessible en SSH.

Avant de commencer :

* Vérifiez que le premier test a réussi.
* Conservez le mot de passe du hotspot.
* Assurez-vous de pouvoir redémarrer le Raspberry Pi.
* Prévoyez, si possible, un accès local de secours.

### 🔌 9.1 — Éteindre le Raspberry Pi

Depuis la session SSH :

```bash
sudo poweroff
```

Attendez l'arrêt complet du Raspberry Pi.

### 📡 9.2 — Rendre les réseaux connus indisponibles

Désactivez temporairement les réseaux auxquels le Raspberry Pi pourrait se connecter.

Par exemple :

* Éteignez votre routeur Wi-Fi.
* Désactivez le partage de connexion de votre smartphone.
* Vérifiez qu'aucun autre réseau enregistré n'est accessible.

### ⚡ 9.3 — Démarrer le Raspberry Pi

Rallumez le Raspberry Pi.

Attendez environ 45 à 60 secondes.

Le script doit constater qu'aucun réseau connu n'est connecté, puis activer le hotspot.

### 📱 9.4 — Rechercher le hotspot

Depuis votre smartphone ou ordinateur, ouvrez les paramètres Wi-Fi.

Recherchez :

```text
Proxmark3-Wireless
```

Connectez-vous avec le mot de passe défini précédemment.

### 🔍 9.5 — Vérifier la connexion SSH

Depuis votre application SSH, utilisez :

| Paramètre        | Valeur                  |
| ---------------- | ----------------------- |
| Hôte             | `192.168.50.1`          |
| Port             | `22`                    |
| Utilisateur      | `piuser`                |
| Authentification | Mot de passe ou clé SSH |

Depuis un terminal :

```bash
ssh piuser@192.168.50.1
```

> 💡 **Remarque**
>
> Certains smartphones signalent que le hotspot ne possède pas d'accès Internet.
>
> C'est normal.
>
> Le réseau sert ici à communiquer directement avec le Raspberry Pi.

### 📡 9.6 — Vérifier le hotspot

Une fois connecté en SSH :

```bash
nmcli device status
```

Résultat attendu :

```text
wlan0  wifi  connected  Proxmark3-Wireless
```

Vérifiez également l'adresse IP :

```bash
ip -4 addr show wlan0
```

Le Raspberry Pi doit utiliser :

```text
192.168.50.1/24
```

### 📋 9.7 — Vérifier les journaux

```bash
journalctl -u proxmark-wifi-fallback.service -b --no-pager
```

Vous devez retrouver des messages indiquant l'activation du hotspot.

> ⚠️ **Important**
>
> Le test valide le comportement sur votre matériel et votre version de NetworkManager.
>
> Le fonctionnement peut varier avec d'autres pilotes, versions du système ou configurations réseau.

---

## 10. Utiliser le Proxmark3 depuis le hotspot

Le hotspot est maintenant opérationnel.

Nous pouvons utiliser le Proxmark3 Easy exactement comme sur un réseau Wi-Fi habituel.

### 📱 10.1 — Se connecter en SSH

Depuis un smartphone connecté à `Proxmark3-Wireless` :

```bash
ssh piuser@192.168.50.1
```

### 🖥️ 10.2 — Ouvrir la session tmux

```bash
pm
```

Cette commande permet de créer ou retrouver la session tmux nommée `proxmark`.

### 📡 10.3 — Lancer Iceman

Si le client n'est pas déjà actif :

```bash
pm3
```

### 🔍 10.4 — Vérifier le Proxmark3

Dans le client Iceman :

```text
hw version
```

Puis :

```text
hw status
```

Le Proxmark3 doit répondre normalement.

### 🔌 10.5 — Quitter temporairement

Pour détacher la session tmux :

```text
Ctrl + B, puis D
```

Vous pourrez retrouver la session lors d'une prochaine connexion SSH, tant que le Raspberry Pi reste allumé.

> 💡 **À retenir**
>
> Le hotspot ne modifie pas le fonctionnement du Proxmark3.
>
> Il fournit simplement un moyen supplémentaire d'accéder au Raspberry Pi.
>
> Les commandes Iceman restent identiques.

---

## 11. Revenir au réseau Wi-Fi habituel

Lorsque vous rentrez chez vous ou retrouvez un réseau enregistré, vous pouvez revenir au fonctionnement normal.

### 🔄 11.1 — Redémarrer le Raspberry Pi

Assurez-vous que votre réseau Wi-Fi habituel est disponible.

Depuis la connexion SSH au hotspot :

```bash
sudo reboot
```

Le Raspberry Pi redémarre.

NetworkManager recherche à nouveau les réseaux enregistrés.

Si l'un d'eux est disponible, le script conserve la connexion normale.

### 🔍 11.2 — Vérifier le résultat

Reconnectez-vous au Raspberry Pi depuis votre réseau local.

Puis exécutez :

```bash
nmcli device status
```

Le réseau habituel doit apparaître comme connexion active.

> ⚠️ **Attention**
>
> Rallumer votre routeur ne provoque pas automatiquement le retour du Raspberry Pi vers ce réseau lorsque le hotspot est actif.
>
> Cette version privilégie un changement de mode au redémarrage.

---

## 12. Diagnostiquer les problèmes

### ❌ 12.1 — Le hotspot n'apparaît pas

Vérifiez la compatibilité du mode point d'accès :

```bash
nmcli -f WIFI-PROPERTIES.AP device show wlan0
```

Vérifiez le profil :

```bash
nmcli connection show "Proxmark3-Wireless"
```

Consultez les journaux :

```bash
journalctl -u proxmark-wifi-fallback.service -b --no-pager
```

Vérifiez également les messages NetworkManager :

```bash
sudo journalctl -u NetworkManager -b --no-pager
```

### ❌ 12.2 — Le service a échoué

```bash
systemctl status proxmark-wifi-fallback.service --no-pager
```

Vérifiez la syntaxe du script :

```bash
bash -n /usr/local/sbin/proxmark-wifi-fallback.sh
```

Assurez-vous que le fichier est exécutable :

```bash
ls -l /usr/local/sbin/proxmark-wifi-fallback.sh
```

### ❌ 12.3 — Le smartphone se connecte, mais SSH ne fonctionne pas

Vérifiez que le service SSH est actif :

```bash
systemctl status ssh
```

Vérifiez l'adresse IP :

```bash
ip -4 addr show wlan0
```

Vérifiez que vous utilisez bien :

```text
192.168.50.1
```

Contrôlez également que votre smartphone est toujours connecté au hotspot.

### ❌ 12.4 — Le Raspberry Pi ne retrouve pas le réseau habituel

Vérifiez les profils enregistrés :

```bash
nmcli connection show
```

Vérifiez que les réseaux connus autorisent la connexion automatique :

```bash
nmcli -f connection.id,connection.autoconnect,connection.autoconnect-priority \
  connection show "Home-WiFi"
```

Vérifiez également que l'interface autorise les connexions automatiques :

```bash
nmcli -f GENERAL.AUTOCONNECT device show wlan0
```

Si nécessaire, avec un accès local ou une connexion réseau fonctionnelle :

```bash
sudo nmcli device set wlan0 autoconnect yes
```

Puis redémarrez lorsque le réseau habituel est disponible.

### ❌ 12.5 — Le hotspot s'active trop tôt

Le délai de recherche est défini dans le script :

```bash
TIMEOUT=30
```

Si votre réseau habituel met du temps à devenir disponible, vous pouvez augmenter ce délai.

Par exemple :

```bash
TIMEOUT=60
```

Après modification, enregistrez le script.

Le nouveau délai sera utilisé lors de la prochaine exécution du service.

> 💡 **Conseil**
>
> Le délai doit être suffisamment long pour laisser NetworkManager établir une connexion.
>
> Une durée plus élevée retarde également l'apparition du hotspot lorsqu'aucun réseau n'est disponible.

---

## 13. Désactiver le hotspot automatique

La fonctionnalité est optionnelle.

Vous pouvez la désactiver sans supprimer les profils Wi-Fi habituels.

### 🛑 13.1 — Désactiver le service

```bash
sudo systemctl disable proxmark-wifi-fallback.service
```

Le service ne sera plus exécuté automatiquement aux prochains démarrages.

### 🔄 13.2 — Réactiver les connexions automatiques

```bash
sudo nmcli device set wlan0 autoconnect yes
```

Vérifiez que vos profils Wi-Fi habituels possèdent :

```text
connection.autoconnect: yes
```

### 🗑️ 13.3 — Supprimer le hotspot

Si vous souhaitez également supprimer son profil :

```bash
sudo nmcli connection delete "Proxmark3-Wireless"
```

> ⚠️ **Attention**
>
> Supprimer un profil Wi-Fi actif peut interrompre immédiatement votre connexion SSH.
>
> Effectuez cette opération depuis un autre réseau ou avec un accès local de secours.

### 🔄 13.4 — Réactiver la fonctionnalité

Si le script, le service et le profil hotspot sont toujours présents :

```bash
sudo systemctl enable proxmark-wifi-fallback.service
```

La fonctionnalité sera de nouveau disponible au prochain démarrage.

---

## 14. Vérification finale

Avant de considérer cette installation comme terminée :

* [ ] NetworkManager fonctionne.
* [ ] Le mode point d'accès est compatible avec `wlan0`.
* [ ] Les réseaux Wi-Fi connus sont enregistrés.
* [ ] Les priorités de connexion sont configurées.
* [ ] Le profil `Proxmark3-Wireless` existe.
* [ ] Le hotspot utilise l'adresse `192.168.50.1`.
* [ ] Un mot de passe Wi-Fi sécurisé est configuré.
* [ ] Le script de basculement est exécutable.
* [ ] Le service systemd est activé.
* [ ] Le Raspberry Pi retrouve un réseau connu au démarrage.
* [ ] Le hotspot apparaît lorsqu'aucun réseau connu n'est disponible.
* [ ] Un smartphone peut se connecter au hotspot.
* [ ] SSH fonctionne depuis le hotspot.
* [ ] La commande `pm` fonctionne.
* [ ] Le client `pm3` démarre correctement.
* [ ] Le retour au réseau habituel fonctionne après redémarrage.

---

## ✅ Étape 7 terminée — Proxmark3 Wireless autonome

Notre installation **Proxmark3 Easy + Raspberry Pi Zero 2 W** dispose maintenant de deux modes de connexion Wi-Fi.

Nous avons appris à :

* Configurer plusieurs réseaux Wi-Fi avec NetworkManager.
* Définir leurs priorités.
* Créer un hotspot sécurisé.
* Configurer une adresse IP fixe.
* Automatiser l'activation du hotspot au démarrage.
* Utiliser systemd pour exécuter notre script.
* Tester le fonctionnement sans réseau extérieur.
* Accéder au Raspberry Pi depuis un smartphone.
* Utiliser le Proxmark3 Iceman en mode autonome.
* Diagnostiquer et désactiver la fonctionnalité.

Le système peut désormais être utilisé à domicile, au bureau ou en déplacement, sans nécessiter systématiquement un réseau Wi-Fi existant.

### 🎯 Pour aller plus loin

Cette fonctionnalité pourra évoluer avec :

* La détection périodique des réseaux connus.
* Le retour automatique du hotspot vers un réseau habituel.
* Une interface web locale.
* Un assistant de configuration du Wi-Fi.
* Une procédure de récupération réseau automatisée.

Ces évolutions ne sont pas nécessaires au fonctionnement de cette version.

---

## 📚 Références

* [NetworkManager — Documentation officielle](https://networkmanager.dev/docs/)
* [Raspberry Pi — Documentation officielle](https://www.raspberrypi.com/documentation/)
* [systemd — Documentation](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html)
* [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md)
* [Étape 4 — Utilisation quotidienne](04-utilisation-quotidienne.md)
* [Étape 5 — Connexion mobile](05-connexion-mobile.md)
* [Étape 6 — Maintenance, sauvegardes et mises à jour](06-maintenance.md)

---

## 🧭 Navigation

| ⬅️ Étape précédente                                                     | 🏠 Sommaire                              | 🌐 English                                                   |
| ----------------------------------------------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| [Étape 6 — Maintenance, sauvegardes et mises à jour](06-maintenance.md) | [Retour à l'accueil](../../README.fr.md) | [Step 7 — Automatic Wi-Fi Hotspot](../en/07-wifi-hotspot.md) |

---

📡 **Projet : Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
