# Anenji - Onduleur Van Master

Monitoring et contrôle d'un onduleur solaire Anenji (ou similaire) via Modbus RTU depuis Home Assistant.

## Matériel
- ESP32dev (Arduino framework)
- Onduleur solaire Anenji via Modbus RTU (UART: TX GPIO17, RX GPIO16, 9600 baud)
- Adresse Modbus: 0x01

## Fonctionnalités
- **11 capteurs** : tension, courant, SOC, PV, grid, AC out, température
- **3 contrôles select** : priorité source, priorité charge, type batterie
- **5 sliders** : courants max, voltages de charge
- **2 switches** : buzzer, power saving
- **2 capteurs calculés** : Battery Power (W), Autonomie (min)
- **Alertes automatiques** : batterie basse, température élevée
- **Diagnostics** : uptime, erreurs Modbus, status device

## Protocole Modbus - Registres

### Capteurs (lecture seule)
| Registre | Type | Nom | Unité | Facteur |
|----------|------|-----|-------|---------|
| 200 | U_WORD | Grid Voltage | V | x0.1 |
| 201 | U_WORD | Grid Frequency | Hz | x0.01 |
| 202 | U_WORD | AC Output Voltage | V | x0.1 |
| 213 | U_WORD | AC Output Power | W | - |
| 214 | U_WORD | AC Output Load | % | - |
| 215 | U_WORD | Battery Voltage | V | x0.1 |
| 216 | S_WORD | Battery Current | A | x0.1 |
| 227 | S_WORD | Inverter Temperature | °C | x0.1 |
| 229 | U_WORD | Battery SOC | % | - |
| 263 | U_WORD | PV Voltage | V | x0.1 |
| 264 | U_WORD | PV Power | W | - |

### Contrôles (écriture)
| Registre | Type | Nom | Valeurs |
|----------|------|-----|---------|
| 250 | U_WORD | Max Grid Charge Current | 2-80A (pas de 2) |
| 252 | U_WORD | Max Charge Current | 10-120A (pas de 10) |
| 253 | U_WORD | Battery Type | 0=AGM, 1=Flooded, 2=User, 3=Pylontech |
| 254 | U_WORD | Bulk Charge Voltage | 24-30V (x0.1) |
| 255 | U_WORD | Float Charge Voltage | 24-30V (x0.1) |
| 256 | U_WORD | Output Source Priority | 0=Utility, 1=Solar, 2=SBU |
| 257 | U_WORD | Charger Source Priority | 0=Solar, 1=Solar+Utility, 2=Solar, 3=Utility |
| 258 | U_WORD | Low DC Cut-off Voltage | 20-24V (x0.1) |
| 259 | U_WORD | Switches (bitmask) | bit0=PowerSaving, bit2=Buzzer |

## Sécurité & Fiabilité
- **OTA protégé** par mot de passe
- **Safe mode** : retour auto au firmware précédent après 5 échecs de boot
- **Reboot auto** : 10 min sans connexion API = redémarrage
- **WiFi fiable** : power_save OFF pour stabilité Modbus
- **Valeurs filtrées** : bornes min/max sur tous les capteurs (détection erreurs 0xFFFF)
- **Alertes ESPHome** : logs automatiques si batterie < 20%/< 10%, tension > 30V, temp > 60°C

## Capteurs calculés
| Capteur | Formule | Description |
|---------|---------|-------------|
| Battery Power | V × A | Puissance batterie (positif=decharge, negatif=charge) |
| Battery Autonomy | (SOC% × 4800Wh) / Power | Temps restant estimé en minutes |

## Configuration
1. Remplir `secrets.yaml` avec les vraies credentials WiFi et OTA
2. Ajuster les pins UART si nécessaire
3. Ajuster la capacité batterie dans `battery_autonomy` (défaut: 200Ah × 24V = 4800Wh)
4. Flasher via ESPHome : `esphome run Anenji.YAML`
