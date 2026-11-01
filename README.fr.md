📡 Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless

🌐 Langue : 🇫🇷 Français | 🇬🇧 English

📖 Présentation

Transformez votre Proxmark3 Easy en outil RFID portable et sans fil grâce à un Raspberry Pi Zero 2 W !

Ce projet explique étape par étape comment connecter un Proxmark3 Easy à un Raspberry Pi Zero 2 W afin de le piloter à distance depuis un smartphone ou un ordinateur.

L'objectif est de créer un système RFID compact, portable et autonome, permettant d'exploiter les fonctionnalités du Proxmark3 sans avoir besoin de le connecter directement à un ordinateur.

Le Raspberry Pi assure la communication avec le Proxmark3, tandis que le smartphone ou l'ordinateur permet d'accéder à son interface de commande à travers une connexion Wi-Fi.

✨ Fonctionnalités
🔌 Connexion du Proxmark3 Easy au Raspberry Pi via USB.
💻 Installation du firmware et du client Proxmark3 Iceman.
📶 Communication sans fil via Wi-Fi.
📱 Pilotage depuis un iPhone grâce à une connexion SSH.
🤖 Compatibilité avec les smartphones Android et les ordinateurs.
🛠️ Exécution des commandes Proxmark3 à distance.
🎒 Possibilité de créer un système portable et autonome.
🔧 Matériel nécessaire
Matériel	Description
Raspberry Pi Zero 2 W	Ordinateur assurant le pilotage du Proxmark3
Proxmark3 Easy	Lecteur et outil d'analyse RFID/NFC
Carte microSD	Stockage du système Raspberry Pi OS
Adaptateur USB OTG	Connexion du Proxmark3 au Raspberry Pi
Alimentation 5 V	Alimentation du Raspberry Pi et du Proxmark3
Smartphone ou ordinateur	Interface de contrôle à distance
🔗 Architecture du système
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

Le Raspberry Pi Zero 2 W communique directement avec le Proxmark3 Easy via USB.

L'utilisateur se connecte au Raspberry Pi en Wi-Fi et utilise une session SSH pour accéder au client Proxmark3 Iceman.

Les commandes sont exécutées sur le Raspberry Pi, puis transmises au Proxmark3.

📚 Documentation

Le tutoriel sera organisé en plusieurs chapitres :

Présentation du projet — Objectifs et fonctionnement.
Matériel nécessaire — Liste des composants et branchements.
Installation du Raspberry Pi — Préparation de la carte microSD et configuration initiale.
Installation du Proxmark3 Iceman — Installation des dépendances et compilation.
Configuration du Wi-Fi — Connexion et utilisation sans fil.
Connexion depuis un iPhone — Accès SSH et pilotage du Proxmark3.
Utilisation du Proxmark3 — Premières commandes et exemples.
Dépannage — Solutions aux problèmes les plus courants.
🚧 État du projet

🟡 Documentation en cours de rédaction.

Le projet repose sur un montage personnel utilisant un Raspberry Pi Zero 2 W et un Proxmark3 Easy.

Les procédures d'installation et de configuration seront progressivement documentées et vérifiées.

🤝 Contributions

Les contributions sont les bienvenues !

Vous pouvez participer au projet en :

Signalant des erreurs ou des problèmes.
Proposant des améliorations.
Partageant vos expériences.
Contribuant à la traduction de la documentation.

N'hésitez pas à ouvrir une Issue ou à proposer une Pull Request.

⚠️ Avertissement

Ce projet est destiné à l'apprentissage, à la recherche en sécurité RFID et aux expérimentations sur des équipements personnels ou pour lesquels vous disposez d'une autorisation explicite.

L'utilisation de ce matériel doit respecter les lois et réglementations applicables.

Les auteurs ne sauraient être tenus responsables d'une utilisation illégale ou non autorisée du matériel et des informations présentées.

📜 Licence

Ce projet est distribué sous licence MIT. Consultez le fichier LICENSE pour plus d'informations.
