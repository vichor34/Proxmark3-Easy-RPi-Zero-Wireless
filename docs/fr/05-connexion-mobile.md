# 📱 Étape 5 — Utiliser Proxmark3 depuis un iPhone, un iPad ou Android

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](../en/05-mobile-connection.md)

[🏠 Retour au sommaire](../../README.fr.md)

---

## 📖 Présentation

Dans les chapitres précédents, nous avons installé et configuré notre **Proxmark3 Easy** sur un **Raspberry Pi Zero 2 W**.

Nous savons désormais utiliser le Proxmark3 depuis un ordinateur grâce à SSH, tmux et au client Iceman.

Nous allons maintenant apprendre à piloter cette installation depuis un **smartphone ou une tablette**, sans connexion USB entre l'appareil mobile et le Proxmark3.

Cette solution fonctionne avec :

- Les iPhone sous iOS.
- Les iPad sous iPadOS.
- Les smartphones Android.
- Les tablettes Android.

Le principe reste identique sur toutes les plateformes :

**Le téléphone se connecte au Raspberry Pi par Wi-Fi, puis utilise SSH pour accéder au client Iceman.**

> 💡 **Important**
>
> Le Proxmark3 Easy reste physiquement connecté au Raspberry Pi par USB.
>
> Le smartphone ne communique pas directement avec le Proxmark3.
>
> Il contrôle à distance le logiciel Iceman exécuté sur le Raspberry Pi.

### 🎯 Objectifs de cette étape

À la fin de ce chapitre, vous saurez :

- Comprendre le fonctionnement de la connexion mobile.
- Installer un client SSH sur iPhone, iPad ou Android.
- Enregistrer une connexion au Raspberry Pi.
- Utiliser le Proxmark3 depuis un terminal mobile.
- Retrouver une session Iceman avec tmux.
- Utiliser les touches spéciales depuis un clavier tactile.
- Connecter le Raspberry Pi à un partage de connexion mobile.
- Identifier les limites des différents modes de connexion.
- Résoudre les principaux problèmes de connexion.

---

## 1. Architecture de notre installation

Notre système repose sur trois éléments :

1. Un smartphone ou une tablette.
2. Un Raspberry Pi Zero 2 W.
3. Un Proxmark3 Easy.

### 🏠 1.1 — Utilisation sur un réseau Wi-Fi domestique

Dans cette configuration, le smartphone et le Raspberry Pi sont connectés au même réseau local.

```text
       ┌──────────────────────┐
       │  iPhone / Android    │
       │                      │
       │  Application SSH     │
       └──────────┬───────────┘
                  │
                  │ Wi-Fi
                  │
       ┌──────────▼───────────┐
       │  Routeur / Box Wi-Fi │
       └──────────┬───────────┘
                  │
                  │ Wi-Fi 2,4 GHz
                  │
       ┌──────────▼───────────┐
       │  Raspberry Pi Zero 2 W│
       │                      │
       │  SSH + tmux + Iceman │
       └──────────┬───────────┘
                  │
                  │ USB
                  │
       ┌──────────▼───────────┐
       │    Proxmark3 Easy    │
       └──────────────────────┘
```

Le smartphone envoie les commandes SSH au Raspberry Pi.

Le client Iceman communique ensuite avec le Proxmark3 par USB.

### 📡 1.2 — Utilisation avec un partage de connexion

Il est également possible d'utiliser un smartphone comme point d'accès Wi-Fi.

Dans ce cas, le Raspberry Pi se connecte au réseau Wi-Fi créé par le téléphone.

```text
       ┌──────────────────────┐
       │  iPhone / Android    │
       │                      │
       │  Partage de connexion│
       │  + Application SSH   │
       └──────────┬───────────┘
                  │
                  │ Wi-Fi
                  │
       ┌──────────▼───────────┐
       │ Raspberry Pi Zero 2 W│
       │                      │
       │ SSH + tmux + Iceman  │
       └──────────┬───────────┘
                  │
                  │ USB
                  │
       ┌──────────▼───────────┐
       │    Proxmark3 Easy    │
       └──────────────────────┘
```

> ⚠️ **Attention**
>
> Le fonctionnement dépend des possibilités de partage de connexion du smartphone.
>
> Certains appareils ou réseaux peuvent limiter les communications entre les périphériques connectés.
>
> La connexion doit donc être testée avant une utilisation en déplacement.

---

## 2. Prérequis

Avant de commencer, assurez-vous que :

- Le Raspberry Pi démarre correctement.
- Le Proxmark3 Easy est connecté au Raspberry Pi.
- Le service SSH est activé.
- Le Raspberry Pi est connecté à un réseau Wi-Fi.
- Le client Iceman fonctionne.
- Les commandes `pm` et `pm3` sont configurées.

Ces opérations sont décrites dans les chapitres précédents.

### 📚 Chapitres nécessaires

- [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md)
- [Étape 3 — Installation de Proxmark3 Iceman](03-installation-proxmark3.md)
- [Étape 4 — Utilisation quotidienne](04-utilisation-quotidienne.md)

### 🔍 2.1 — Vérifier l'adresse réseau

Depuis un terminal SSH déjà fonctionnel sur un ordinateur, exécutez :

```bash
hostname -I
```

Cette commande affiche les adresses IP attribuées au Raspberry Pi.

Pour connaître l'adresse IPv4 utilisée par l'interface Wi-Fi :

```bash
ip -4 addr show wlan0
```

Exemple :

```text
inet 192.168.1.100/24
```

Dans cet exemple, l'adresse IP du Raspberry Pi est :

```text
192.168.1.100
```

> 💡 **Remarque**
>
> Cette adresse est uniquement un exemple.
>
> L'adresse réelle dépend de votre réseau et peut changer lorsque le Raspberry Pi se connecte à un autre point d'accès.

---

## 3. Se connecter depuis un iPhone ou un iPad

Sur iOS et iPadOS, il est nécessaire d'utiliser une application permettant d'établir une connexion SSH.

Nous utiliserons **Termius** comme exemple.

### 🍎 3.1 — Installer un client SSH

Ouvrez l'App Store.

Recherchez :

```text
Termius
```

Installez l'application.

Termius permet notamment :

- D'établir des connexions SSH.
- D'enregistrer les informations de connexion.
- D'ouvrir un terminal distant.
- D'utiliser des touches supplémentaires adaptées aux commandes Linux.

> 💡 **Remarque**
>
> Certaines fonctionnalités de Termius sont payantes.
>
> Une connexion SSH classique peut être réalisée sans abonnement, sous réserve des fonctionnalités proposées par la version installée.
>
> D'autres applications SSH compatibles peuvent également être utilisées.

### 🔧 3.2 — Ajouter le Raspberry Pi

Ouvrez Termius.

Créez un nouvel hôte SSH.

Renseignez les informations suivantes :

| Paramètre | Valeur |
|---|---|
| Nom | Proxmark3 |
| Adresse | `proxmark.local` |
| Port | `22` |
| Utilisateur | `piuser` |
| Authentification | Mot de passe ou clé SSH |

Remplacez `piuser` par le nom d'utilisateur configuré sur votre Raspberry Pi.

Si `proxmark.local` ne fonctionne pas, utilisez l'adresse IP réelle du Raspberry Pi.

### 🔐 3.3 — Établir la connexion

Sélectionnez l'hôte enregistré.

L'application établit une connexion SSH avec le Raspberry Pi.

Lors de la première connexion, vérifiez l'empreinte de la clé SSH du serveur avant de l'accepter.

Saisissez ensuite votre mot de passe si vous utilisez cette méthode d'authentification.

Une fois connecté, vous devez obtenir une invite similaire à :

```text
piuser@proxmark:~ $
```

### 📡 3.4 — Lancer le Proxmark3

Dans le terminal :

```bash
pm
```

Puis, si Iceman n'est pas déjà lancé :

```bash
pm3
```

Le client Iceman doit démarrer.

Vous pouvez maintenant exécuter une commande de diagnostic :

```text
hw version
```

### 📱 3.5 — Utiliser les touches spéciales

Certaines commandes nécessitent des touches qui ne sont pas directement présentes sur le clavier tactile.

C'est notamment le cas de :

- `Ctrl`
- `Esc`
- `Tab`
- Les touches directionnelles

Dans Termius, utilisez la barre de touches supplémentaires du terminal lorsqu'elle est disponible.

Pour détacher une session tmux :

1. Activez la touche `Ctrl`.
2. Appuyez sur `B`.
3. Relâchez la combinaison.
4. Appuyez sur `D`.

La session tmux reste active sur le Raspberry Pi.

> 💡 **Conseil**
>
> Si vous utilisez régulièrement le Proxmark3 depuis un iPhone, familiarisez-vous avec les touches spéciales de votre application SSH.
>
> Leur disposition peut varier selon la version de l'application.

---

## 4. Se connecter depuis Android

La procédure est similaire à celle utilisée sur iPhone.

Nous pouvons également utiliser Termius sur Android.

### 🤖 4.1 — Installer un client SSH

Ouvrez le Google Play Store.

Recherchez :

```text
Termius
```

Installez l'application.

Vous pouvez également utiliser un autre client SSH compatible avec Android.

### 🔧 4.2 — Ajouter le Raspberry Pi

Créez un nouvel hôte SSH.

Renseignez :

| Paramètre | Valeur |
|---|---|
| Nom | Proxmark3 |
| Adresse | `proxmark.local` |
| Port | `22` |
| Utilisateur | `piuser` |
| Authentification | Mot de passe ou clé SSH |

Si le nom `.local` ne fonctionne pas, utilisez l'adresse IP du Raspberry Pi.

### 🔐 4.3 — Se connecter

Ouvrez la connexion SSH enregistrée.

Vérifiez l'identité du serveur lors de la première connexion.

Saisissez le mot de passe si nécessaire.

Une fois connecté, le terminal du Raspberry Pi apparaît.

### 📡 4.4 — Lancer Iceman

Exécutez :

```bash
pm
```

Puis :

```bash
pm3
```

Vous pouvez maintenant utiliser les commandes Iceman.

Par exemple :

```text
hw status
```

### ⌨️ 4.5 — Utiliser le clavier Android

Les applications SSH Android disposent généralement d'outils permettant d'accéder aux touches spéciales.

Pour détacher une session tmux :

```text
Ctrl + B
```

Puis :

```text
D
```

Le fonctionnement exact dépend du client SSH utilisé.

---

## 5. Retrouver une session depuis un autre appareil

L'un des principaux avantages de notre configuration est l'utilisation de tmux.

Une session Iceman peut rester active même lorsque le smartphone se déconnecte.

### 🔄 5.1 — Exemple d'utilisation

Imaginons que vous utilisiez le Proxmark3 depuis votre ordinateur.

Vous avez lancé :

```bash
pm
```

Puis :

```bash
pm3
```

Vous souhaitez maintenant reprendre votre travail depuis votre smartphone.

### 🔌 5.2 — Détacher la session sur l'ordinateur

Dans tmux :

```text
Ctrl + B
```

Puis :

```text
D
```

Vous pouvez fermer la connexion SSH.

### 📱 5.3 — Reprendre la session sur le smartphone

Ouvrez votre application SSH.

Connectez-vous au Raspberry Pi.

Exécutez :

```bash
pm
```

Vous retrouvez la session existante.

Si Iceman est toujours actif, son interface réapparaît.

> 💡 **Important**
>
> Le Raspberry Pi doit rester alimenté.
>
> Le Proxmark3 doit rester connecté.
>
> tmux ne conserve pas les sessions après un redémarrage du Raspberry Pi.

### ⚠️ 5.4 — Éviter les connexions concurrentes

Évitez de contrôler simultanément la même session depuis plusieurs appareils.

Pour passer d'un appareil à un autre, détachez la session sur le premier appareil avant de la reprendre sur le second.

---

## 6. Utiliser le partage de connexion d'un smartphone

Jusqu'à présent, nous avons utilisé un réseau Wi-Fi domestique.

Mais notre objectif est également de pouvoir utiliser le Proxmark3 en déplacement.

Pour cela, nous pouvons connecter le Raspberry Pi au partage de connexion Wi-Fi d'un smartphone.

### 📶 6.1 — Principe

Le smartphone crée un réseau Wi-Fi.

Le Raspberry Pi se connecte à ce réseau.

L'application SSH du smartphone utilise ensuite l'adresse du Raspberry Pi sur ce réseau.

Cette solution ne nécessite pas de routeur externe.

> ⚠️ **Limitation**
>
> Le Raspberry Pi Zero 2 W fonctionne en Wi-Fi 2,4 GHz.
>
> Le point d'accès du smartphone doit donc proposer une connexion compatible avec cette bande.

---

## 7. Configurer un partage de connexion iPhone

### 🍎 7.1 — Activer le partage de connexion

Sur l'iPhone :

1. Ouvrez **Réglages**.
2. Accédez à **Partage de connexion**.
3. Activez **Autoriser d'autres utilisateurs**.
4. Vérifiez le nom du réseau et son mot de passe.

Si l'option est disponible, activez :

**Maximiser la compatibilité**

Cette option favorise notamment la compatibilité avec les appareils utilisant le Wi-Fi 2,4 GHz.

### 🔧 7.2 — Enregistrer le réseau sur le Raspberry Pi

Pour cette première configuration, utilisez un ordinateur déjà connecté au Raspberry Pi par SSH.

Sur le Raspberry Pi, exécutez :

```bash
sudo nmcli device wifi connect "NOM_DU_RESEAU_IPHONE" --ask
```

Remplacez `NOM_DU_RESEAU_IPHONE` par le nom du réseau créé par votre iPhone.

NetworkManager vous demandera le mot de passe si nécessaire.

> ⚠️ **Attention**
>
> La connexion SSH actuelle peut être interrompue lorsque le Raspberry Pi change de réseau.
>
> Effectuez cette opération lorsque vous pouvez retrouver l'adresse du Raspberry Pi sur le nouveau réseau.

### 🔍 7.3 — Vérifier la connexion

Une fois le Raspberry Pi connecté au partage de connexion, recherchez son adresse IP.

Vous pouvez utiliser :

```bash
hostname -I
```

si vous disposez encore d'un accès au terminal.

Sinon, consultez les informations disponibles sur le téléphone ou utilisez un autre moyen de découverte réseau autorisé.

### 📱 7.4 — Se connecter depuis l'iPhone

Dans Termius :

1. Ouvrez la connexion enregistrée.
2. Utilisez `proxmark.local` si ce nom fonctionne.
3. Sinon, renseignez l'adresse IP du Raspberry Pi sur le réseau du partage de connexion.
4. Établissez la connexion SSH.

Puis :

```bash
pm
```

Et, si nécessaire :

```bash
pm3
```

> 💡 **Remarque**
>
> L'accès SSH depuis l'iPhone qui fournit lui-même le partage de connexion dépend du comportement réseau d'iOS.
>
> Il est recommandé de tester cette configuration avant de l'utiliser hors de chez vous.
>
> Si la communication locale est impossible, utilisez un point d'accès Wi-Fi externe compatible ou un autre mode de réseau.

---

## 8. Configurer un partage de connexion Android

### 🤖 8.1 — Activer le point d'accès Wi-Fi

Sur Android, ouvrez les paramètres réseau.

Selon le fabricant, l'option peut s'appeler :

- Point d'accès mobile.
- Partage de connexion.
- Hotspot Wi-Fi.

Activez le point d'accès.

### 📶 8.2 — Vérifier la bande Wi-Fi

Si votre téléphone permet de choisir la bande Wi-Fi, sélectionnez :

```text
2,4 GHz
```

Cette bande est compatible avec le Raspberry Pi Zero 2 W.

### 🔧 8.3 — Enregistrer le réseau

Depuis le Raspberry Pi :

```bash
sudo nmcli device wifi connect "NOM_DU_RESEAU_ANDROID" --ask
```

Remplacez le nom du réseau par celui de votre point d'accès Android.

Saisissez le mot de passe demandé.

### 🔍 8.4 — Retrouver l'adresse IP

Une fois le Raspberry Pi connecté, recherchez son adresse IP.

Certains smartphones Android affichent la liste des appareils connectés et leurs adresses.

Vous pouvez également vérifier l'adresse depuis le Raspberry Pi lorsque vous disposez d'un accès au terminal.

### 📡 8.5 — Utiliser le Proxmark3

Depuis votre application SSH Android :

1. Connectez-vous à l'adresse du Raspberry Pi.
2. Ouvrez la session tmux.
3. Lancez Iceman si nécessaire.

Commandes :

```bash
pm
```

Puis :

```bash
pm3
```

> ⚠️ **Remarque**
>
> Le fonctionnement d'une connexion SSH depuis le téléphone qui héberge lui-même le point d'accès peut varier selon les versions d'Android et les fabricants.
>
> Testez cette possibilité sur votre appareil.

---

## 9. Passer du Wi-Fi domestique au partage de connexion

NetworkManager peut enregistrer plusieurs réseaux Wi-Fi.

Nous avons déjà abordé cette fonctionnalité au chapitre 2.

### 📋 9.1 — Afficher les connexions enregistrées

Sur le Raspberry Pi :

```bash
nmcli connection show
```

Cette commande affiche les connexions réseau connues.

### 📶 9.2 — Afficher le réseau actif

```bash
nmcli device status
```

L'interface Wi-Fi est généralement :

```text
wlan0
```

### 🔄 9.3 — Connexion automatique

Lorsque plusieurs réseaux enregistrés sont disponibles, NetworkManager peut sélectionner une connexion en fonction de sa disponibilité et de sa priorité.

Il est donc possible de préparer le Raspberry Pi pour différents environnements :

- Réseau Wi-Fi domestique.
- Réseau d'un atelier.
- Point d'accès d'un smartphone.

> 💡 **Conseil**
>
> Enregistrez et testez les réseaux avant de partir en déplacement.
>
> Cela évite de devoir modifier la configuration réseau lorsque vous ne disposez plus d'un accès SSH fonctionnel.

### ⚠️ 9.4 — Attention aux changements d'adresse IP

L'adresse IP du Raspberry Pi peut changer lorsqu'il rejoint un autre réseau.

Par exemple :

```text
Réseau domestique : 192.168.1.100
```

```text
Partage de connexion : adresse différente
```

Ces valeurs sont uniquement des exemples.

Si `proxmark.local` ne fonctionne pas, utilisez l'adresse IP attribuée sur le réseau actuellement utilisé.

---

## 10. Sécurité des connexions mobiles

Le Raspberry Pi donne accès au système Linux et au Proxmark3.

Il est donc important de sécuriser les connexions.

### 🔐 10.1 — Utiliser un mot de passe robuste

Le compte utilisateur du Raspberry Pi doit être protégé par un mot de passe suffisamment long et difficile à deviner.

Évitez de réutiliser un mot de passe déjà utilisé sur d'autres services.

### 🔑 10.2 — Préférer les clés SSH

Pour une utilisation régulière, l'authentification par clé SSH est recommandée.

Elle permet d'éviter l'utilisation systématique d'un mot de passe.

La configuration exacte dépend de l'application SSH utilisée.

> 💡 **Conseil**
>
> Ne désactivez pas l'authentification par mot de passe avant d'avoir vérifié qu'une connexion par clé SSH fonctionne correctement.

### 🌐 10.3 — Ne pas exposer SSH directement sur Internet

Notre tutoriel utilise des connexions sur un réseau local.

Il n'est pas nécessaire d'ouvrir le port SSH du Raspberry Pi sur Internet.

Évitez donc de configurer une redirection du port `22` sur votre routeur.

Pour un accès à travers Internet, utilisez une solution sécurisée adaptée, comme un VPN correctement configuré.

### 📱 10.4 — Protéger le smartphone

Protégez votre téléphone avec :

- Un code de verrouillage.
- Une authentification biométrique si disponible.
- Les mises à jour de sécurité.
- Un accès contrôlé aux identifiants SSH.

Ne partagez pas vos clés privées SSH.

---

## 11. Résoudre les problèmes courants

### ❌ 11.1 — Le smartphone ne trouve pas le Raspberry Pi

Vérifiez :

1. Que le Raspberry Pi est allumé.
2. Qu'il est connecté au bon réseau Wi-Fi.
3. Que le smartphone peut communiquer avec les autres appareils de ce réseau.
4. Que l'adresse IP utilisée est correcte.

Essayez l'adresse IP à la place de :

```text
proxmark.local
```

### ❌ 11.2 — Le partage de connexion ne fonctionne pas

Vérifiez :

- Que le partage de connexion est activé.
- Que le réseau utilise une bande Wi-Fi compatible.
- Que le mot de passe est correct.
- Que le Raspberry Pi est réellement connecté.
- Que les communications locales entre appareils sont autorisées.

### ❌ 11.3 — La connexion SSH est refusée

Vérifiez que le service SSH fonctionne sur le Raspberry Pi.

Depuis un accès local ou une autre connexion fonctionnelle :

```bash
systemctl status ssh
```

### ❌ 11.4 — La commande `pm` ne fonctionne pas

Vérifiez l'alias :

```bash
alias pm
```

Si nécessaire, utilisez directement :

```bash
tmux new-session -A -s proxmark
```

### ❌ 11.5 — La commande `pm3` ne fonctionne pas

Vérifiez :

```bash
command -v pm3
```

Si nécessaire, consultez le chapitre 3 pour vérifier l'installation du client Iceman.

### ❌ 11.6 — Le clavier tactile ne permet pas d'utiliser tmux

Vérifiez que votre application SSH dispose d'une touche `Ctrl`.

Pour détacher la session :

```text
Ctrl + B
```

Puis :

```text
D
```

Si votre application ne permet pas d'envoyer facilement ces touches, essayez un autre client SSH.

### ❌ 11.7 — La connexion se coupe lorsque le téléphone se verrouille

Certains systèmes mobiles suspendent les applications réseau lorsqu'elles passent en arrière-plan.

Cela peut interrompre la connexion SSH.

Si Iceman a été lancé dans une session tmux, le programme peut continuer de fonctionner sur le Raspberry Pi.

Reconnectez-vous puis exécutez :

```bash
pm
```

Vous retrouverez la session si elle est toujours active.

---

## 12. Procédure mobile simplifiée

Voici la procédure à retenir.

### 📱 Démarrage

**1. Alimenter le Raspberry Pi et le Proxmark3.**

**2. Connecter le smartphone au réseau permettant d'accéder au Raspberry Pi.**

**3. Ouvrir l'application SSH.**

**4. Se connecter au Raspberry Pi.**

**5. Ouvrir la session persistante :**

```bash
pm
```

**6. Lancer Iceman si nécessaire :**

```bash
pm3
```

### 🔍 Premier diagnostic

Dans Iceman :

```text
hw version
```

Puis :

```text
hw status
```

### 🔌 Déconnexion temporaire

Détachez la session tmux :

```text
Ctrl + B
```

Puis :

```text
D
```

Vous pouvez ensuite fermer l'application SSH.

### 🔄 Reconnexion

Ouvrez de nouveau votre application SSH.

Connectez-vous au Raspberry Pi.

Exécutez :

```bash
pm
```

Vous retrouvez votre session.

---

## 13. Vérification avant de continuer

Avant de considérer la configuration mobile comme terminée :

- [ ] Le Raspberry Pi fonctionne correctement.
- [ ] Le Proxmark3 Easy est connecté et reconnu.
- [ ] Une application SSH est installée sur le smartphone.
- [ ] Une connexion SSH au Raspberry Pi fonctionne.
- [ ] La commande `pm` ouvre une session tmux.
- [ ] La commande `pm3` lance Iceman.
- [ ] Les commandes de diagnostic fonctionnent.
- [ ] Les touches spéciales du clavier mobile sont accessibles.
- [ ] Une session tmux peut être détachée.
- [ ] La session peut être retrouvée après reconnexion.
- [ ] Le partage de connexion a été testé si vous souhaitez l'utiliser.
- [ ] Vous savez retrouver l'adresse IP du Raspberry Pi.
- [ ] L'accès SSH est protégé.

---

## ✅ Étape 5 terminée — Proxmark3 utilisable depuis un smartphone

Notre installation peut maintenant être utilisée depuis :

- Un ordinateur Windows.
- Un ordinateur macOS.
- Un ordinateur Linux.
- Un iPhone.
- Un iPad.
- Un smartphone Android.
- Une tablette Android.

Grâce au Raspberry Pi Zero 2 W, le Proxmark3 Easy peut être piloté à distance par Wi-Fi.

Les sessions tmux permettent de conserver un environnement de travail persistant et de passer d'un appareil à un autre.

### 🚀 Dans la prochaine étape

Nous aborderons la maintenance de notre installation :

- Sauvegarder les fichiers importants.
- Mettre à jour Raspberry Pi OS.
- Mettre à jour le client Iceman.
- Préparer une mise à jour du firmware.
- Vérifier les versions et la compatibilité.
- Diagnostiquer les problèmes avancés.
- Restaurer une installation précédente si nécessaire.

---

## 📚 Références

- [Raspberry Pi — Documentation officielle](https://www.raspberrypi.com/documentation/)
- [Termius — Site officiel](https://termius.com/)
- [Proxmark3 Iceman — RfidResearchGroup](https://github.com/RfidResearchGroup/proxmark3)
- [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md)
- [Étape 3 — Installation de Proxmark3 Iceman](03-installation-proxmark3.md)
- [Étape 4 — Utilisation quotidienne](04-utilisation-quotidienne.md)

---

## 🧭 Navigation

| ⬅️ Étape précédente | 🏠 Sommaire | 🌐 English |
|---|---|---|
| [Étape 4 — Utilisation quotidienne](04-utilisation-quotidienne.md) | [Retour à l'accueil](../../README.fr.md) | [Step 5 — Mobile Connection](../en/05-mobile-connection.md) |

> 💡 **Navigation provisoire**
>
> Les liens de navigation seront harmonisés lorsque toutes les pages du tutoriel auront été créées.

---

📡 **Projet : Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
