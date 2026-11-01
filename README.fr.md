# 📡 Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](README.md)

---

## 📖 Présentation

**Transformez votre Proxmark3 Easy en outil RFID portable et sans fil grâce à un Raspberry Pi Zero 2 W !**

Ce projet explique étape par étape comment connecter un **Proxmark3 Easy** à un **Raspberry Pi Zero 2 W** afin de le piloter à distance depuis un smartphone ou un ordinateur.

L'objectif est de créer un système RFID compact, portable et autonome, permettant d'exploiter les fonctionnalités du Proxmark3 sans avoir besoin de le connecter directement à un ordinateur.

Le Raspberry Pi assure la communication avec le Proxmark3, tandis que le smartphone ou l'ordinateur permet d'accéder à son interface de commande à travers une connexion Wi-Fi.

### ✨ Fonctionnalités

* 🔌 Connexion du Proxmark3 Easy au Raspberry Pi via USB.
* 💻 Installation du firmware et du client Proxmark3 Iceman.
* 📶 Communication sans fil via Wi-Fi.
* 📱 Pilotage depuis un iPhone grâce à une connexion SSH.
* 🤖 Compatibilité avec les smartphones Android et les ordinateurs.
* 🛠️ Exécution des commandes Proxmark3 à distance.
* 🎒 Possibilité de créer un système portable et autonome.

---

## 🔧 Matériel nécessaire

| Matériel                 | Description                                  |
| ------------------------ | -------------------------------------------- |
| Raspberry Pi Zero 2 W    | Ordinateur assurant le pilotage du Proxmark3 |
| Proxmark3 Easy           | Lecteur et outil d'analyse RFID/NFC          |
| Carte microSD            | Stockage du système Raspberry Pi OS          |
| Adaptateur USB OTG       | Connexion du Proxmark3 au Raspberry Pi       |
| Alimentation 5 V         | Alimentation du Raspberry Pi et du Proxmark3 |
| Smartphone ou ordinateur | Interface de contrôle à distance             |

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

L'utilisateur se connecte au Ra
