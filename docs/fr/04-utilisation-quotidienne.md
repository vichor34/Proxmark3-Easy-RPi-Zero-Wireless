# 🖥️ Étape 4 — Utilisation quotidienne et accès distant au Proxmark3

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](../en/04-daily-usage.md)

[🏠 Retour au sommaire](../../README.fr.md)

---

## 📖 Présentation

Les trois premiers chapitres nous ont permis de préparer le matériel, d'installer Raspberry Pi OS et de configurer le firmware **Proxmark3 Iceman**.

Notre installation est maintenant opérationnelle.

Nous allons apprendre à utiliser le **Proxmark3 Easy à distance**, depuis un ordinateur connecté au même réseau que le Raspberry Pi Zero 2 W.

L'objectif est de pouvoir piloter le Proxmark3 sans écran, clavier ou souris directement connectés au Raspberry Pi.

Nous utiliserons principalement trois outils :

- **SSH** pour accéder au Raspberry Pi à distance.
- **tmux** pour conserver une session de terminal active.
- **Iceman** pour communiquer avec le Proxmark3 Easy.

Nous verrons également comment effectuer les premiers diagnostics, résoudre les problèmes courants et arrêter correctement le système.

> 💡 **Remarque**
>
> Ce chapitre présente l'utilisation depuis Windows, macOS et Linux.
>
> La connexion depuis un iPhone, un iPad ou un appareil Android sera détaillée dans le chapitre 5.

### 🎯 Objectifs de cette étape

À la fin de ce chapitre, vous saurez :

- Démarrer correctement le Raspberry Pi et le Proxmark3.
- Vérifier leur connexion USB.
- Vous connecter en SSH depuis un ordinateur.
- Utiliser les commandes `pm` et `pm3`.
- Retrouver une session persistante après une déconnexion.
- Utiliser les principales commandes de diagnostic Iceman.
- Effectuer un premier test de lecture RFID/NFC.
- Résoudre les problèmes de connexion courants.
- Arrêter le système sans risquer d'endommager la carte microSD.

---

## 1. Configuration de référence

Notre installation utilise les éléments suivants :

| Élément | Configuration |
|---|---|
| Raspberry Pi | Zero 2 W |
| Système | Raspberry Pi OS Lite 64 bits |
| Proxmark3 | Easy générique |
| Firmware | Iceman — RfidResearchGroup |
| Connexion Proxmark3 | USB |
| Connexion réseau | Wi-Fi 2,4 GHz |
| Accès distant | SSH |
| Session persistante | tmux |
| Nom du Raspberry Pi | `proxmark` |
| Utilisateur d'exemple | `piuser` |
| Commande de session | `pm` |
| Commande du client | `pm3` |

### 📚 Prérequis

Avant de commencer, vous devez avoir terminé :

- [Étape 1 — Matériel et branchements](01-materiel.md)
- [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md)
- [Étape 3 — Installation de Proxmark3 Iceman](03-installation-proxmark3.md)

Les commandes `pm` et `pm3` doivent avoir été configurées comme expliqué dans les chapitres précédents.

---

## 2. Démarrer le système

### 🔌 2.1 — Vérifier les branchements

Avant de mettre le Raspberry Pi sous tension, vérifiez :

1. Que la carte microSD est correctement insérée.
2. Que le Proxmark3 Easy est connecté au port USB de données.
3. Que l'adaptateur OTG est correctement branché.
4. Que l'alimentation est adaptée au Raspberry Pi et aux périphériques.
5. Que le réseau Wi-Fi configuré est disponible.

> ⚠️ **Attention**
>
> Une alimentation insuffisante peut provoquer des déconnexions USB, des redémarrages ou des erreurs de communication.
>
> Si le Proxmark3 n'est pas détecté, vérifiez l'alimentation avant de modifier la configuration logicielle.

### ⚡ 2.2 — Mettre le Raspberry Pi sous tension

Branchez l'alimentation du Raspberry Pi.

Le système démarre automatiquement.

Attendez environ une à deux minutes avant de tenter une connexion SSH.

Le temps nécessaire peut varier selon la carte microSD et les services installés.

### 📶 2.3 — Vérifier la connexion réseau

Le Raspberry Pi doit se connecter automatiquement à un réseau Wi-Fi enregistré.

Dans notre configuration, son nom est :

```text
proxmark
```

Il peut être accessible à l'adresse :

```text
proxmark.local
```

> 💡 **Remarque**
>
> Le nom `.local` utilise généralement le protocole mDNS.
>
> Si votre ordinateur ou votre réseau ne permet pas sa résolution, utilisez directement l'adresse IP du Raspberry Pi.

---

## 3. Se connecter au Raspberry Pi depuis un ordinateur

La connexion s'effectue avec le protocole SSH.

Il permet d'ouvrir un terminal sécurisé sur le Raspberry Pi à travers le réseau.

### 💻 3.1 — Depuis Windows

Windows 10 et Windows 11 disposent généralement d'un client SSH intégré.

Ouvrez :

**PowerShell** ou **Terminal Windows**.

Exécutez :

```bash
ssh piuser@proxmark.local
```

Saisissez le mot de passe configuré lors de l'installation du Raspberry Pi.

Lors de la première connexion, SSH peut demander de confirmer l'empreinte de la clé du serveur.

Vérifiez qu'il s'agit bien de votre Raspberry Pi avant de l'accepter.

### 🍎 3.2 — Depuis macOS

Ouvrez l'application **Terminal**.

Exécutez :

```bash
ssh piuser@proxmark.local
```

Saisissez votre mot de passe.

La procédure est identique à celle utilisée sous Windows.

### 🐧 3.3 — Depuis Linux

Ouvrez un terminal.

Exécutez :

```bash
ssh piuser@proxmark.local
```

Le client OpenSSH est installé par défaut sur de nombreuses distributions Linux.

S'il est absent, installez-le avec le gestionnaire de paquets de votre distribution.

### 🌐 3.4 — Connexion avec l'adresse IP

Si `proxmark.local` ne fonctionne pas, recherchez l'adresse IP du Raspberry Pi dans l'interface d'administration de votre routeur.

Par exemple :

```bash
ssh piuser@192.168.1.100
```

> ⚠️ **Important**
>
> L'adresse `192.168.1.100` est uniquement un exemple.
>
> Remplacez-la par l'adresse réelle de votre Raspberry Pi.

### ✅ 3.5 — Vérifier la connexion

Une fois connecté, vous devez obtenir une invite de commande similaire à :

```text
piuser@proxmark:~ $
```

Vous contrôlez maintenant le Raspberry Pi à distance.

---

## 4. Vérifier la connexion du Proxmark3 Easy

Avant de lancer Iceman, nous allons vérifier que le périphérique USB est bien détecté.

### 🔍 4.1 — Afficher les périphériques USB

Dans le terminal SSH :

```bash
lsusb
```

Cette commande affiche les périphériques USB connectés.

Le Proxmark3 doit normalement apparaître dans cette liste lorsqu'il fonctionne dans son mode habituel.

### 🔍 4.2 — Vérifier le port série

Exécutez :

```bash
ls -l /dev/ttyACM*
```

Dans notre installation, le Proxmark3 apparaît sous :

```text
/dev/ttyACM0
```

Le numéro peut varier.

Si plusieurs périphériques série sont connectés, le Proxmark3 peut utiliser un autre numéro.

### 🔍 4.3 — Examiner les derniers messages du système

Si le périphérique n'apparaît pas, exécutez :

```bash
sudo dmesg | tail -n 30
```

Cette commande affiche les derniers messages du noyau Linux.

Ils peuvent aider à identifier :

- Une connexion USB.
- Une déconnexion inattendue.
- Une erreur de communication.
- Un problème d'alimentation.
- Un périphérique série nouvellement détecté.

> 💡 **Conseil**
>
> Si le Proxmark3 fonctionnait auparavant, vérifiez d'abord le câble, l'adaptateur OTG et l'alimentation avant de réinstaller le firmware.

---

## 5. Comprendre les commandes `pm` et `pm3`

Nous avons configuré deux commandes dans les chapitres précédents.

Elles ont des fonctions différentes.

| Commande | Fonction |
|---|---|
| `pm` | Créer ou retrouver une session tmux |
| `pm3` | Lancer le client Proxmark3 Iceman |

### 🖥️ 5.1 — La commande `pm`

Exécutez :

```bash
pm
```

Cette commande utilise l'alias créé au chapitre 2 :

```bash
alias pm='tmux new-session -A -s proxmark'
```

Elle permet :

- De créer une session nommée `proxmark`.
- De retrouver cette session si elle existe déjà.
- De conserver les programmes exécutés dans cette session lors d'une déconnexion SSH.

### 📡 5.2 — La commande `pm3`

Une fois dans la session tmux :

```bash
pm3
```

Cette commande lance le client Iceman installé au chapitre 3.

Si le Proxmark3 est connecté et reconnu, le client doit établir la communication avec le périphérique.

### 🔄 5.3 — Procédure complète

Depuis votre ordinateur :

**1. Connexion SSH :**

```bash
ssh piuser@proxmark.local
```

**2. Ouverture de la session persistante :**

```bash
pm
```

**3. Lancement du client Iceman :**

```bash
pm3
```

Vous pouvez maintenant utiliser les commandes Proxmark3.

### 💡 Pourquoi deux commandes ?

`pm` ne lance pas directement le Proxmark3.

Il ouvre un environnement de travail persistant.

`pm3` lance ensuite le logiciel permettant de communiquer avec le Proxmark3.

Cette séparation permet également d'utiliser la session tmux pour d'autres opérations, comme la maintenance ou la compilation.

---

## 6. Utiliser une session persistante avec tmux

L'un des avantages de notre installation est la possibilité de conserver une session active après une déconnexion SSH.

### 🔌 6.1 — Détacher une session

Lorsque vous êtes dans tmux, appuyez successivement sur :

```text
Ctrl + B
```

Relâchez les touches, puis appuyez sur :

```text
D
```

Vous revenez au terminal SSH classique.

Les programmes exécutés dans la session tmux continuent de fonctionner.

### 🔄 6.2 — Retrouver une session

Après une nouvelle connexion SSH :

```bash
pm
```

Vous retrouvez la session précédente.

Si le client Iceman était toujours actif, son terminal réapparaît.

### 🔍 6.3 — Afficher les sessions existantes

Depuis un terminal situé en dehors de tmux :

```bash
tmux ls
```

Cette commande affiche les sessions disponibles.

### 🛑 6.4 — Fermer définitivement une session

Pour fermer correctement la session :

1. Quittez le client Iceman.
2. Revenez à l'invite de commande Linux.
3. Exécutez :

```bash
exit
```

Si aucun autre programme ou fenêtre ne reste actif dans la session tmux, celle-ci se termine.

> ⚠️ **Important**
>
> Détacher une session n'est pas la même chose que la fermer.
>
> Une session détachée peut être retrouvée.
>
> Une session terminée ne peut pas être restaurée.

### ⚠️ 6.5 — Limites de tmux

tmux ne protège pas contre :

- Une coupure d'alimentation.
- Un redémarrage du Raspberry Pi.
- Une déconnexion physique du Proxmark3.
- L'arrêt du programme Iceman.
- Une panne matérielle.

Le Raspberry Pi doit rester allumé pour que la session continue de fonctionner.

---

## 7. Découvrir les commandes Iceman

Le client Iceman possède de nombreuses commandes.

Nous allons commencer par les commandes d'aide et de diagnostic.

> ⚠️ **Utilisation responsable**
>
> Utilisez le Proxmark3 uniquement sur des badges, cartes et équipements vous appartenant ou pour lesquels vous disposez d'une autorisation explicite.

### 📖 7.1 — Afficher l'aide générale

Dans le client Iceman :

```text
help
```

Cette commande affiche les principales catégories de commandes disponibles.

### 🔍 7.2 — Afficher les informations du matériel

```text
hw version
```

Cette commande permet notamment de consulter les versions du client, du firmware et du bootloader.

### 🩺 7.3 — Vérifier l'état du matériel

```text
hw status
```

Cette commande fournit des informations sur l'état du Proxmark3.

Elle peut être utile pour identifier certains problèmes de communication ou de configuration.

### 🧪 7.4 — Effectuer un diagnostic matériel

```text
hw tune
```

Cette commande effectue des mesures liées aux antennes du Proxmark3.

Elle peut aider à vérifier leur comportement.

> 💡 **Remarque**
>
> Les résultats dépendent notamment du modèle de Proxmark3, des antennes et de l'environnement.
>
> Une valeur isolée ne suffit pas nécessairement à diagnostiquer une panne.

### 📡 7.5 — Tester la lecture NFC

Pour effectuer un premier test, utilisez un badge NFC ISO 14443-A vous appartenant.

Exécutez :

```text
hf 14a reader
```

Approchez le badge de l'antenne haute fréquence du Proxmark3.

Le logiciel doit tenter de détecter le badge et afficher les informations disponibles.

Cette commande est adaptée à un premier test non destructif.

### 💡 7.6 — Différence entre HF et LF

Le Proxmark3 prend en charge différentes familles de technologies RFID.

| Catégorie | Fréquence courante | Exemples |
|---|---|---|
| LF — Low Frequency | 125 kHz / 134,2 kHz | Certains badges d'identification |
| HF — High Frequency | 13,56 MHz | NFC, ISO 14443, ISO 15693 |

Les commandes et les antennes utilisées dépendent de la technologie du badge.

Pour un premier essai, utilisez un badge dont vous connaissez le type.

---

## 8. Utiliser le Proxmark3 depuis plusieurs ordinateurs

Le Raspberry Pi peut être accessible depuis différents ordinateurs connectés à son réseau.

Par exemple :

- Un ordinateur Windows.
- Un Mac.
- Un ordinateur Linux.

Chaque appareil utilise une connexion SSH.

### 🔄 8.1 — Passer d'un ordinateur à un autre

Depuis le premier ordinateur :

1. Ouvrez la session SSH.
2. Lancez `pm`.
3. Lancez `pm3`.
4. Détachez la session avec `Ctrl + B`, puis `D`.
5. Fermez la connexion SSH.

Depuis le second ordinateur :

```bash
ssh piuser@proxmark.local
```

Puis :

```bash
pm
```

Vous retrouvez la session tmux existante.

### ⚠️ 8.2 — Éviter les accès simultanés

Il est préférable qu'une seule personne contrôle le Proxmark3 à la fois.

Plusieurs connexions SSH peuvent coexister, mais cela ne signifie pas que plusieurs clients Iceman doivent accéder simultanément au même périphérique USB.

Évitez également d'envoyer des commandes concurrentes depuis plusieurs terminaux attachés à la même session tmux.

> 💡 **Conseil**
>
> Pour passer d'un appareil à un autre, détachez d'abord la session sur le premier appareil avant de la reprendre sur le second.

---

## 9. Résoudre les problèmes courants

Cette section regroupe les problèmes les plus fréquents lors de l'utilisation quotidienne.

### ❌ 9.1 — Impossible de se connecter en SSH

Exemple :

```text
ssh: Could not resolve hostname proxmark.local
```

Vérifiez :

1. Que le Raspberry Pi est allumé.
2. Qu'il est connecté au Wi-Fi.
3. Que votre ordinateur peut accéder au même réseau.
4. Que le nom `proxmark.local` est correctement écrit.

Essayez ensuite de vous connecter avec son adresse IP.

### ❌ 9.2 — Connexion SSH refusée

Exemple :

```text
Connection refused
```

Cela peut indiquer que le service SSH n'est pas accessible sur l'adresse utilisée.

Vérifiez la configuration SSH du Raspberry Pi et l'adresse réseau.

Si vous disposez d'un accès local au Raspberry Pi, vous pouvez contrôler le service avec :

```bash
systemctl status ssh
```

### ❌ 9.3 — Le Proxmark3 n'est pas détecté

Commencez par :

```bash
lsusb
```

Puis :

```bash
ls -l /dev/ttyACM*
```

Et :

```bash
sudo dmesg | tail -n 30
```

Vérifiez les branchements et l'alimentation.

Si le Proxmark3 apparaît sous un autre port série, adaptez la configuration du client si nécessaire.

### ❌ 9.4 — La commande `pm` ne fonctionne pas

Exemple :

```text
pm: command not found
```

Vérifiez la présence de l'alias :

```bash
alias pm
```

Si l'alias est absent, consultez le chapitre 2.

Vous pouvez également lancer directement :

```bash
tmux new-session -A -s proxmark
```

### ❌ 9.5 — La commande `pm3` ne fonctionne pas

Exemple :

```text
pm3: command not found
```

Vérifiez le chemin du client :

```bash
command -v pm3
```

Si aucun chemin n'est affiché, vérifiez l'installation et la configuration du `PATH` décrites au chapitre 3.

Vous pouvez essayer de lancer le client directement :

```bash
~/proxmark3-new/pm3
```

### ❌ 9.6 — Une session tmux est déjà ouverte

Si la session est déjà attachée à un autre terminal, détachez-la depuis cet autre terminal avec :

```text
Ctrl + B
```

Puis :

```text
D
```

Reconnectez-vous ensuite avec :

```bash
pm
```

### ❌ 9.7 — Le client Iceman ne répond plus

Si le client ne répond plus, vérifiez d'abord que la connexion SSH est toujours active.

Si nécessaire, essayez d'interrompre la commande en cours avec :

```text
Ctrl + C
```

> ⚠️ **Attention**
>
> N'interrompez pas une opération de flashage en cours.
>
> Une interruption pendant l'écriture du firmware peut rendre le périphérique inutilisable.

Si le client est réellement bloqué, il peut être nécessaire de le fermer puis de le relancer.

Évitez de débrancher le Proxmark3 sans avoir identifié la cause du problème.

---

## 10. Arrêter correctement le système

Le Raspberry Pi utilise une carte microSD pour son système d'exploitation.

Une coupure brutale de l'alimentation pendant une écriture peut provoquer une corruption des fichiers.

Il est donc recommandé d'arrêter correctement Linux.

### 🛑 10.1 — Quitter le client Iceman

Dans le client Proxmark3 :

```text
quit
```

Vous revenez au terminal Linux.

### 🔚 10.2 — Quitter la session tmux

Si vous souhaitez terminer la session :

```bash
exit
```

Vous revenez au terminal SSH classique.

### 🔌 10.3 — Arrêter le Raspberry Pi

Exécutez :

```bash
sudo shutdown -h now
```

Le Raspberry Pi commence sa procédure d'arrêt.

La connexion SSH sera automatiquement interrompue.

Attendez la fin de l'activité du système avant de débrancher l'alimentation.

> ⚠️ **Important**
>
> Le Raspberry Pi Zero 2 W ne possède pas de bouton d'arrêt intégré permettant de couper automatiquement son alimentation.
>
> La commande `shutdown` arrête le système d'exploitation, mais l'alimentation électrique reste physiquement connectée.

### 🔄 10.4 — Redémarrer le Raspberry Pi

Pour redémarrer sans débrancher l'alimentation :

```bash
sudo reboot
```

Attendez que le système soit de nouveau accessible avant de vous reconnecter en SSH.

---

## 11. Procédure quotidienne simplifiée

Voici la procédure à retenir pour utiliser le Proxmark3 au quotidien.

### 🚀 Démarrage

**1. Alimenter le Raspberry Pi et le Proxmark3.**

Attendre que le système démarre.

**2. Se connecter en SSH :**

```bash
ssh piuser@proxmark.local
```

**3. Ouvrir la session persistante :**

```bash
pm
```

**4. Lancer Iceman si le client n'est pas déjà actif :**

```bash
pm3
```

**5. Utiliser le Proxmark3.**

Par exemple :

```text
hw version
```

### 🔌 Déconnexion temporaire

Pour conserver la session :

```text
Ctrl + B
```

Puis :

```text
D
```

Vous pouvez ensuite fermer la connexion SSH.

### 🔄 Reconnexion

Depuis le même ordinateur ou un autre :

```bash
ssh piuser@proxmark.local
```

Puis :

```bash
pm
```

Vous retrouvez votre session.

### 🛑 Arrêt complet

Quittez le client Iceman :

```text
quit
```

Quittez la session tmux :

```bash
exit
```

Puis arrêtez le Raspberry Pi :

```bash
sudo shutdown -h now
```

---

## 12. Vérification avant de continuer

Avant de passer au chapitre suivant, assurez-vous que :

- [ ] Le Raspberry Pi démarre correctement.
- [ ] Le Proxmark3 Easy est connecté et alimenté.
- [ ] Le Raspberry Pi rejoint automatiquement le Wi-Fi.
- [ ] La connexion SSH fonctionne depuis un ordinateur.
- [ ] Le Proxmark3 est détecté sur le port USB.
- [ ] La commande `pm` ouvre une session tmux.
- [ ] La commande `pm3` lance Iceman.
- [ ] Les commandes `hw version` et `hw status` fonctionnent.
- [ ] Un test de lecture NFC autorisé fonctionne.
- [ ] Vous savez détacher une session tmux.
- [ ] Vous savez retrouver une session existante.
- [ ] Vous savez arrêter correctement le Raspberry Pi.

---

## ✅ Étape 4 terminée — Proxmark3 prêt pour une utilisation quotidienne

Notre installation est maintenant utilisable à distance depuis un ordinateur Windows, macOS ou Linux.

Nous savons :

- Démarrer le Raspberry Pi et le Proxmark3.
- Accéder au système avec SSH.
- Utiliser les commandes `pm` et `pm3`.
- Conserver une session Iceman active avec tmux.
- Effectuer des diagnostics et des tests RFID/NFC.
- Résoudre les problèmes de connexion les plus courants.
- Arrêter correctement le Raspberry Pi.

### 🚀 Dans la prochaine étape

Nous verrons comment utiliser exactement la même installation depuis un appareil mobile :

- **iPhone et iPad (Apple / iOS / iPadOS).**
- **Smartphones et tablettes Android.**

Nous expliquerons comment installer un client SSH adapté, enregistrer une connexion au Raspberry Pi et utiliser le Proxmark3 depuis un terminal mobile.

L'objectif sera de disposer d'un **Proxmark3 Easy sans fil, pilotable depuis un smartphone**, sans modifier le matériel.

---

## 📚 Références

- [Dépôt officiel Proxmark3 Iceman — RfidResearchGroup](https://github.com/RfidResearchGroup/proxmark3)
- [Étape 1 — Matériel et branchements](01-materiel.md)
- [Étape 2 — Installation du Raspberry Pi](02-installation-raspberry.md)
- [Étape 3 — Installation de Proxmark3 Iceman](03-installation-proxmark3.md)

---

## 🧭 Navigation

| ⬅️ Étape précédente | 🏠 Sommaire | 🌐 English |
|---|---|---|
| [Étape 3 — Installation de Proxmark3 Iceman](03-installation-proxmark3.md) | [Retour à l'accueil](../../README.fr.md) | [Step 4 — Daily Usage](../en/04-daily-usage.md) |

> 💡 **Navigation provisoire**
>
> Les liens de navigation vers les chapitres suivants seront harmonisés lorsque toutes les pages du tutoriel auront été créées.

---

📡 **Projet : Proxmark3 Easy — Raspberry Pi Zero 2 W Wireless**
