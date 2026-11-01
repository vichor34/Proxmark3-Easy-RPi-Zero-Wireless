# 🔧 Matériel nécessaire et branchements

🌐 **Langue :** 🇫🇷 Français | [🇬🇧 English](../en/01-hardware.md)

[🏠 Retour à l'accueil](../../README.fr.md)

---

## 1. Présentation du matériel

Ce projet utilise un **Raspberry Pi Zero 2 W** pour piloter un **Proxmark3 Easy** à distance via Wi-Fi.

Le montage repose sur un adaptateur USB OTG avec alimentation externe, permettant de connecter le Proxmark3 au Raspberry Pi tout en fournissant une alimentation électrique.

### Liste des composants

| Composant                   | Quantité | Description                               |
| --------------------------- | -------- | ----------------------------------------- |
| Raspberry Pi Zero 2 W       | 1        | Ordinateur miniature avec Wi-Fi intégré   |
| Proxmark3 Easy              | 1        | Outil d'analyse RFID/NFC                  |
| Adaptateur USB OTG alimenté | 1        | Adaptateur avec alimentation externe      |
| Câble USB-A vers micro-USB  | 1        | Connexion du Proxmark3 à l'adaptateur OTG |
| Alimentation USB 5 V        | 1        | Alimentation électrique du montage        |
| Carte microSD               | 1        | 16 Go minimum recommandés                 |
| Smartphone ou ordinateur    | 1        | Accès à distance via SSH                  |

---

## 2. Adaptateur USB OTG avec alimentation

L'adaptateur utilisé possède trois connecteurs :

* **Micro-USB mâle** : connexion au port USB du Raspberry Pi.
* **USB-A femelle** : connexion du Proxmark3 Easy.
* **Micro-USB femelle** : connexion de l'alimentation externe 5 V.

Ce montage évite d'utiliser un hub USB classique et permet de conserver un ensemble compact.

**Attention :** tous les adaptateurs OTG alimentés ne fonctionnent pas de la même manière. Certains modèles ne fournissent pas d'alimentation au Raspberry Pi ou ne sont pas compatibles avec son fonctionnement USB OTG.

Il est donc important de choisir un adaptateur adapté et de vérifier son fonctionnement.

---

## 3. Schéma de branchement

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

### Procédure de connexion

1. Insérer la carte microSD dans le Raspberry Pi Zero 2 W.
2. Brancher le connecteur micro-USB mâle de l'adaptateur OTG sur le port **USB** du Raspberry Pi.
3. Brancher le câble USB du Proxmark3 Easy sur la prise USB-A femelle de l'adaptateur.
4. Connecter l'autre extrémité du câble au Proxmark3 Easy.
5. Brancher l'alimentation 5 V sur la prise micro-USB femelle de l'adaptateur OTG.
6. Vérifier que le Raspberry Pi démarre et que le Proxmark3 est alimenté.

---

## 4. Points importants

### Port USB du Raspberry Pi

Le Raspberry Pi Zero 2 W dispose de deux connecteurs micro-USB :

* **USB** : port de données compatible USB OTG.
* **PWR IN** : entrée d'alimentation.

L'adaptateur OTG doit impérativement être connecté au port **USB** pour permettre la communication avec le Proxmark3.

### Alimentation électrique

Utiliser une alimentation 5 V régulée, capable de fournir suffisamment de courant pour le Raspberry Pi et le Proxmark3.

Une alimentation 5 V / 2,5 A constitue un bon point de départ, sous réserve de la compatibilité et des caractéristiques de l'adaptateur OTG.

Éviter les alimentations de mauvaise qualité, susceptibles de provoquer des déconnexions USB ou des redémarrages.

### Compatibilité

Le montage décrit correspond à la configuration utilisée pour ce projet.

Le fonctionnement d'autres modèles d'adaptateurs OTG alimentés n'est pas garanti.

---

## 5. Vérification du matériel

Une fois Raspberry Pi OS installé et le système démarré, il sera possible de vérifier la détection du Proxmark3 avec la commande :

```bash
lsusb
```

Pour rechercher le port série ass
