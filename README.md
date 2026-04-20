# ETRS_403
# 🛤️ Rail Linéaire — Télécommande Web

Projet de contrôle d'un rail linéaire sur tige filetée, piloté via une interface web hébergée sur un **Raspberry Pi Pico W**, qui communique avec un **Arduino Uno** équipé d'un shield LCD.

> Réalisé par **Elsa Sohn, Fanny Santi, Alban Stora, Léon Hewitt**

---

## 📐 Architecture du système

```
Navigateur Web
      │
      │  HTTP (Wi-Fi)
      ▼
Raspberry Pi Pico W        ←── Serveur web embarqué (MicroPython)
      │
      │  I2C (adresse 0x08)
      ▼
Arduino Uno + Shield LCD   ←── Contrôle moteur + affichage menu
      │
      │  PWM
      ▼
Moteurs du rail linéaire
```

---

## 🧰 Matériel requis

| Composant | Rôle |
|---|---|
| Raspberry Pi Pico W | Serveur web Wi-Fi + maître I2C |
| Arduino Uno | Contrôle moteurs + interface LCD |
| Shield LCD 16×2 avec boutons | Affichage et navigation locale |
| Moteurs DC | Déplacement du rail (avancer / reculer) |
| Câblage I2C (SDA/SCL) | Communication Pico → Arduino |

---

## ⚙️ Configuration

### Réseau Wi-Fi (Pico W)

Dans `pico/main.py`, modifier les identifiants réseau :

```python
ssid     = 'votre_ssid'
password = 'votre_mot_de_passe'
```

### Broches Arduino

| Broche | Fonction |
|---|---|
| `D3` | PWM moteur — Avancer (`->`) |
| `D11` | PWM moteur — Reculer (`<-`) |
| `A0` | Lecture des boutons du shield LCD |
| `A4 / A5` | SDA / SCL (I2C esclave, adresse `8`) |

### Broches Pico W

| Broche | Fonction |
|---|---|
| `GP4` | SDA — bus I2C vers Arduino |
| `GP5` | SCL — bus I2C vers Arduino |

---

## 🚀 Installation et démarrage

### 1. Flasher le Pico W

1. Installer [MicroPython](https://micropython.org/download/rp2-pico-w/) sur le Pico W
2. Copier `pico/main.py` à la racine du Pico W (via Thonny ou `mpremote`)
3. Installer les bibliothèques nécessaires :
   ```
   picozero
   ```

### 2. Flasher l'Arduino

1. Ouvrir `arduino/rail.ino` dans l'IDE Arduino
2. Installer les bibliothèques :
   - `LiquidCrystal` (incluse par défaut)
   - `Wire` (incluse par défaut)
3. Téléverser sur l'Arduino Uno

### 3. Câblage I2C

Connecter le Pico W à l'Arduino :

```
Pico GP4  →  Arduino A4  (SDA)
Pico GP5  →  Arduino A5  (SCL)
Pico GND  →  Arduino GND
```

### 4. Démarrage

1. Alimenter l'Arduino puis le Pico W
2. L'adresse IP du Pico s'affiche dans la console série
3. Ouvrir un navigateur et accéder à `http://<ip_du_pico>`

---

## 🖥️ Interface web

### Page Accueil (`/`)

- Présentation du projet
- Affichage de la température interne du Pico W

### Page Télécommande (`/telecom`)

- **Boutons de commande** : Avancer, Reculer, Arrêt total
- **Contrôle LED** : allumer / éteindre la LED embarquée du Pico
- **Potentiomètre virtuel** : slider de vitesse de -100 % à +100 % avec retour automatique à 0 après 2 secondes
- **Barre de statut** : affichage en temps réel de la température, de l'état LED et des moteurs

### Routes HTTP disponibles

| Route | Action |
|---|---|
| `/` | Page d'accueil |
| `/telecom` | Page de télécommande |
| `/avancer` | Démarre le moteur dans le sens horaire |
| `/reculer` | Démarre le moteur dans le sens anti-horaire (arrête le moteur 1) |
| `/stop` | Arrête le moteur |
| `/lighton` | Allume la LED du Pico |
| `/lightoff` | Éteint la LED du Pico |
| `/speed?v=XX` | Règle la vitesse via le slider AJAX (-100 à +100) |

---

## 🎮 Interface LCD locale (Arduino)

Le shield LCD permet de contrôler le rail directement sans passer par le site web.

### Navigation

| Bouton | Action |
|---|---|
| `UP` / `DOWN` | Naviguer dans le menu principal |
| `SELECT` / `RIGHT` | Entrer dans un sous-menu |
| `LEFT` | Retour au menu principal |

### Entrées du menu

| Entrée | Description |
|---|---|
| `1. Avancer (->)` | Activer / désactiver le moteur dans le sens horaire |
| `2. Reculer (<-)` | Activer / désactiver le moteur dans le sens anti-horaire |
| `3. Vitesse` | Régler la vitesse PWM (0–255) via UP/DOWN (±10) et LEFT/RIGHT (±1) |
| `4. Infos` | Afficher l'état des moteurs et la vitesse courante |

---

## 📡 Protocole I2C

Le Pico W envoie à chaque action une chaîne de caractères à l'Arduino (adresse `8`) :

```
T:24.3/L:ON/M1:ON/M2:OFF
```

| Champ | Description |
|---|---|
| `T:xx.x` | Température en °C |
| `L:ON\|OFF` | État de la LED |
| `M1:ON\|OFF` | État du moteur 1 (Avancer) |
| `M2:ON\|OFF` | État du moteur 2 (Reculer) |

---

## ⚠️ Points d'attention

- Les deux moteurs ne peuvent pas être actifs simultanément : activer l'un arrête l'autre automatiquement.
- La vitesse PWM est partagée entre les deux moteurs (réglable de 0 à 255).
- Le `delay(150)` dans la boucle Arduino sert d'anti-rebond basique pour les boutons.
- Le buffer I2C est limité à 31 caractères : ne pas dépasser cette taille dans les messages envoyés.
