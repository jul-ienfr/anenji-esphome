# Anenji ESPHome - Onduleur Van Master

Firmware ESPHome pour monitoring et contrôle d'un onduleur solaire Anenji via Modbus RTU depuis Home Assistant.

## Quick Start

```bash
# 1. Cloner le repo
git clone https://github.com/VOTRE_USER/anenji-esphome.git
cd anenji-esphome

# 2. Configurer les credentials
cp secrets.yaml.example secrets.yaml
# Éditer secrets.yaml avec vos vraies credentials WiFi/OTA

# 3. Flasher l'ESP32
esphome run Anenji.YAML
```

## Matériel

| Composant | Description | Prix estimé |
|-----------|-------------|-------------|
| ESP32dev | Carte de développement ESP32 | ~5-8€ |
| Module RS485 | MAX485 ou SP3485 (TTL ↔ RS485) | ~1-3€ |
| Câble torsadé | 2 fils, 0.5mm² minimum | ~2-5€ |
| Alimentation 5V | USB ou buck converter 12V→5V | ~3-5€ |

**Total : ~12-23€**

Voir [WIRING.md](WIRING.md) pour le câblage détaillé.

## Fonctionnalités

### Capteurs (11 Modbus + 4 calculés)
| Capteur | Unité | Description |
|---------|-------|-------------|
| Battery Voltage | V | Tension batterie (×0.1) |
| Battery Current | A | Courant batterie (×0.1, signé) |
| Battery SOC | % | État de charge |
| Battery Power | W | Puissance (V × A) |
| Battery Autonomy | min | Temps restant estimé |
| PV Voltage | V | Tension panneaux solaires |
| PV Power | W | Puissance solaire |
| PV Current | A | Courant solaire (calculé) |
| Grid Voltage | V | Tension réseau |
| Grid Frequency | Hz | Fréquence réseau |
| AC Output Voltage | V | Tension sortie |
| AC Output Power | W | Puissance sortie |
| AC Output Load | % | Charge sortie |
| Inverter Temperature | °C | Température onduleur |

### Contrôles
- **Select** : Priorité source, priorité charge, type batterie
- **Sliders** : Courants max charge, voltages de charge, capacité batterie
- **Switches** : Buzzer, Power Saving Mode, Status LED, Web Server
- **Debug Mode** : Toggle production/debug via slider HA

### Fonctionnalités avancées
- **Mode Debug/Production** : Ajuste automatiquement les intervals et filtres
- **Filtres intelligents** : Delta filters, sliding window, skip_updates
- **Alertes automatiques** : Batterie basse (<20%, <10%), température élevée (>60°C)
- **Diagnostics** : Uptime, erreurs Modbus, status device

## Configuration

### Paramètres modifiables dans le YAML
```yaml
substitutions:
  cpu_freq: "80MHz"        # Fréquence CPU (80/160/240MHz)
  logger_level: "WARN"     # Niveau log (DEBUG/INFO/WARN/ERROR)
```

### Paramètres modifiables depuis Home Assistant
- Slider "Debug Mode" : 0=Production, 1=Debug
- Slider "Battery Capacity" : 1000-10000 Wh
- Tous les contrôles Modbus (courants, voltages, priorités)

## Optimisations Énergie

Ce firmware est optimisé pour les systèmes autonomes (van/solaire) :
- CPU à 80 MHz (vs 240 par défaut)
- WiFi à 8.5 dB (vs 20 dB max)
- Logger à WARN (réduit I/O)
- WiFi power_save à LIGHT
- Skip updates sur capteurs stables
- Delta filters pour réduire le trafic
- LED éteinte en mode production

## Protocole Modbus

### Capteurs (lecture seule)
| Registre | Type | Nom | Unité | Facteur |
|----------|------|-----|-------|---------|
| 200 | U_WORD | Grid Voltage | V | ×0.1 |
| 201 | U_WORD | Grid Frequency | Hz | ×0.01 |
| 202 | U_WORD | AC Output Voltage | V | ×0.1 |
| 213 | U_WORD | AC Output Power | W | - |
| 214 | U_WORD | AC Output Load | % | - |
| 215 | U_WORD | Battery Voltage | V | ×0.1 |
| 216 | S_WORD | Battery Current | A | ×0.1 |
| 227 | S_WORD | Inverter Temperature | °C | ×0.1 |
| 229 | U_WORD | Battery SOC | % | - |
| 263 | U_WORD | PV Voltage | V | ×0.1 |
| 264 | U_WORD | PV Power | W | - |

### Contrôles (écriture)
| Registre | Type | Nom | Valeurs |
|----------|------|-----|---------|
| 250 | U_WORD | Max Grid Charge Current | 2-80A (pas de 2) |
| 252 | U_WORD | Max Charge Current | 10-120A (pas de 10) |
| 253 | U_WORD | Battery Type | 0=AGM, 1=Flooded, 2=User, 3=Pylontech |
| 254 | U_WORD | Bulk Charge Voltage | 24-30V (×0.1) |
| 255 | U_WORD | Float Charge Voltage | 24-30V (×0.1) |
| 256 | U_WORD | Output Source Priority | 0=Utility, 1=Solar, 2=SBU |
| 257 | U_WORD | Charger Source Priority | 0=Solar, 1=Solar+Utility, 2=Solar, 3=Utility |
| 258 | U_WORD | Low DC Cut-off Voltage | 20-24V (×0.1) |
| 259 | U_WORD | Switches (bitmask) | bit0=PowerSaving, bit2=Buzzer |

## Sécurité & Fiabilité

- **OTA protégé** par mot de passe
- **Safe mode** : retour auto au firmware précédent après 5 échecs de boot
- **Reboot auto** : 10 min sans connexion API = redémarrage
- **WiFi fiable** : power_save LIGHT pour stabilité Modbus
- **Valeurs filtrées** : bornes min/max sur tous les capteurs
- **Alertes ESPHome** : logs automatiques si batterie < 20%/< 10%, tension > 30V, temp > 60°C

## Licence

Ce projet est sous licence [GPL-3.0](LICENSE).

## Contribuer

1. Forker le projet
2. Créer une branche (`git checkout -b feature/amelioration`)
3. Committer (`git commit -m 'feat: ajout de...'`)
4. Pusher (`git push origin feature/amelioration`)
5. Ouvrir une Pull Request
