# Câblage Anenji - ESP32 ↔ Onduleur

## Vue d'ensemble

```
┌─────────────┐      RS485       ┌─────────────────┐
│   ESP32     │◄════════════════►│  Onduleur Anenji │
│  (ESP32dev) │    (2 fils)      │  (Modbus RTU)    │
└──────┬──────┘                  └─────────────────┘
       │
       │  Alimentation 5V
       │
┌──────┴──────┐
│  Alimentation│
│  5V USB ou  │
│  buck 12V→5V│
└─────────────┘
```

## Matériel nécessaire

| Composant | Description | Prix estimé |
|-----------|-------------|-------------|
| ESP32dev | Carte de développement ESP32 | ~5-8€ |
| Module RS485 | MAX485 ou SP3485 (TTL ↔ RS485) | ~1-3€ |
| Câble torsadé | 2 fils, 0.5mm² minimum, longueur selon besoin | ~2-5€ |
| Alimentation 5V | USB ou buck converter 12V→5V (si 12V dispo) | ~3-5€ |
| Connecteur | Bornier à vis ou connecteur selon onduleur | ~1-2€ |

**Total : ~12-23€**

## Détail du câblage

### 1. ESP32 → Module RS485 (TTL)

Le module RS485 convertit le signal TTL (3.3V) du ESP32 en signal RS485 (différentiel) pour le bus Modbus.

```
ESP32 (ESP32dev)          Module MAX485/SP3485
┌──────────────┐          ┌──────────────────┐
│         GPIO17 (TX) ────│ DI   (Data In)   │
│         GPIO16 (RX) ────│ RO   (Data Out)  │
│              GND ───────│ GND              │
│              3.3V ──────│ VCC              │  ← Si le module le supporte
│                        │                  │
│         GPIO4  ─────────│ DE   (TX Enable) │  ← Optionnel (voir ci-dessous)
│         GPIO4  ─────────│ /RE  (RX Enable) │  ← Optionnel (voir ci-dessous)
└──────────────┘          └──────────────────┘
```

### 2. Module RS485 → Onduleur (bus Modbus)

```
Module MAX485/SP3485      Onduleur Anenji
┌──────────────────┐      ┌──────────────────┐
│           A+  ──────────│ A+  (ou Data+)   │
│           B-  ──────────│ B-  (ou Data-)   │
│           GND ──────────│ GND (optionnel)  │
└──────────────────┘      └──────────────────┘
```

### 3. Alimentation ESP32

```
Option A : USB (recommandé pour le debug)
  ESP32 ← Câble USB ← PC ou adaptateur murale 5V/1A

Option B : Buck converter 12V→5V (installé van)
  Batterie 12V → Buck converter → 5V → ESP32 (broche VIN ou USB)
  ⚠️ Vérifier que le buck fournit au moins 500mA stable
```

## Broches utilisées

| Broche ESP32 | Fonction | Direction | Notes |
|-------------|----------|-----------|-------|
| GPIO17 | UART TX | ESP32 → RS485 | Données Modbus sortantes |
| GPIO16 | UART RX | RS485 → ESP32 | Données Modbus entrantes |
| GND | Masse | Commun | **Obligatoire** avec le module RS485 |
| VIN ou 5V | Alimentation | Entrée | 5V via USB ou buck converter |

## Configuration Modbus

| Paramètre | Valeur | Notes |
|-----------|--------|-------|
| Baud rate | 9600 | Doit correspondre à l'adresse 0x01 de l'onduleur |
| Data bits | 8 | Standard |
| Parity | NONE | Pas de parité |
| Stop bits | 1 | Standard |
| Address | 0x01 | Adresse Modbus de l'onduleur (vérifier sur l'écran) |
| Protocole | Modbus RTU | Sur bus RS485 |

## Câblage détaillé du module RS485

### Module MAX485 (le plus courant)

```
        ┌─────────────────┐
        │    MAX485       │
        │                 │
  RO  ──┤1            8├── VCC (5V ou 3.3V)
  /RE ──┤2            7├── B- ────── vers onduleur
   DE ──┤3            6├── A+ ────── vers onduleur
  DI  ──┤4            5├── GND ───── GND commun
        └─────────────────┘

Câblage :
- RO (pin 1) → GPIO16 (RX ESP32)
- DI (pin 4) → GPIO17 (TX ESP32)
- /RE (pin 2) → GND (mode réception permanent)
  OU → GPIO4 (contrôle optionnel : HIGH = transmet, LOW = reçoit)
- DE (pin 3) → GND (mode réception permanent)
  OU → GPIO4 (même broche que /RE, câblage en opposition)
- VCC (pin 8) → 5V (ou 3.3V selon le module)
- GND (pin 5) → GND ESP32
- A+ (pin 6) → A+ onduleur
- B- (pin 7) → B- onduleur
```

### Module SP3485 (plus moderne, 3.3V natif)

Même câblage que MAX485, mais fonctionne nativement en 3.3V → compatible directement avec l'ESP32 sans adaptateur de niveau.

## Résistance de terminaison

Sur les longs câbles (>10m), ajouter une résistance de **120Ω** entre A+ et B- **aux deux extrémités** du bus :

```
Module RS485          Onduleur
     A+ ───────┬──────── A+
                │
              [120Ω]  ← Résistance de terminaison
                │
     B- ───────┴──────── B-
```

- **Câble court (<5m)** : pas nécessaire
- **Câble moyen (5-20m)** : recommandé
- **Câble long (>20m)** : obligatoire

## Connecteur de l'onduleur

La plupart des onduleurs Anenji/similaires ont un connecteur RS485 à l'arrière. Vérifier le manuel pour identifier :

| Broche onduleur | Fonction |
|----------------|----------|
| A+ ou D+ | Données+ (RS485) |
| B- ou D- | Données- (RS485) |
| GND | Masse (optionnel) |

**⚠️ Important** : ne pas confondre A+ et B- ! Si les données ne passent pas, essayer d'inverser A et B.

## Schéma de câblage complet

```
┌─────────────────────────────────────────────────────────────┐
│                        VAN                                   │
│                                                              │
│  ┌──────────┐    USB    ┌──────────┐                        │
│  │ Batterie │───5V────►│  ESP32   │                        │
│  │  12V     │           │          │                        │
│  └──────────┘           │  GPIO17 ─┼──┐                     │
│                         │  GPIO16 ─┼──┤                     │
│                         │  GND   ─┼──┤                     │
│                         └──────────┘  │                     │
│                                       │                     │
│                         ┌──────────┐  │                     │
│                         │ MAX485   │◄─┘                     │
│                         │          │                        │
│                         │ A+  ─────┼──── Câble torsadé ──┐ │
│                         │ B-  ─────┼─────────────────────┤ │
│                         └──────────┘                     │ │
│                                                          │ │
│                         ┌─────────────────┐              │ │
│                         │   ONDULEUR      │◄─────────────┘ │
│                         │   ANENJI        │                │
│                         │   RS485 A+/B-   │                │
│                         └─────────────────┘                │
└─────────────────────────────────────────────────────────────┘
```

## Checklist de test

1. **Vérifier l'alimentation** : ESP32 allume (LED verte)
2. **Vérifier le câblage TTL** : GX17→DI, GPIO16→RO, GND→GND
3. **Vérifier le câblage RS485** : A+→A+, B-→B-, GND commun
4. **Flasher le firmware** : `esphome run Anenji.YAML`
5. **Vérifier les logs** : `esphome logs Anenji.YAML`
   - Si "Modbus Status: Connected" → tout fonctionne
   - Si "Modbus Status: Error" → vérifier câblage et addresses
6. **Tester une lecture** : vérifier que "Battery Voltage" affiche une valeur réaliste (20-30V)

## Dépannage

| Symptôme | Cause probable | Solution |
|----------|---------------|----------|
| Aucune donnée | Câble A+/B- inversé | Inverser A et B |
| Aucune donnée | Masse non connectée | Connecter GND entre ESP32 et module |
| Données erratiques | Pas de terminaison | Ajouter résistance 120Ω |
| Timeout Modbus | Mauvaise adresse | Vérifier adresse Modbus de l'onduleur |
| Timeout Modbus | Mauvais baud rate | Vérifier 9600 baud sur l'onduleur |
| ESP32 redémarre | Alimentation insuffisante | Utiliser USB ou buck 5V/1A |
| Module RS485 ne chauffe pas | VCC non connecté | Vérifier broche VCC |
