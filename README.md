# ha-solar-master-steuerung

**Zero-Feed-In Regelung für SunEnergy XT 500 Pro mit PI-Regler in Home Assistant**

Automatische Nulleinspeisung für einen Hausspeicher: Der Akku gibt immer genau so viel ab wie das Haus gerade braucht, ohne Strom ins öffentliche Netz einzuspeisen. Ab V8.1 werden zwei Hoymiles Microinverter gleichzeitig geregelt.

---

## Hardware

| Gerät | Funktion |
|---|---|
| SunEnergy XT 500 Pro | Hausspeicher / Wechselrichter |
| Shelly Pro 3EM | Netzleistungsmessung am Hauptzähler |
| Hoymiles HMS-2000-4T | Microinverter (Süd-Ausrichtung) |
| Hoymiles HMS-1600-4T | Microinverter (Nord/Ost/West-Ausrichtung) |
| OpenDTU | Lokale Steuerung beider Hoymiles-WR ohne Cloud |

---

## Features

### ⚡ Nulleinspeisung (Zero Feed-In)
Die Messung erfolgt über den Shelly Pro 3EM am Hauptzähler. Der PI-Regler passt den Sollwert am Netzanschluss alle 5 Sekunden an. Ein asymmetrischer Filter sorgt dafür, dass die Regelung bei echtem Mehrbedarf schnell reagiert, bei kurzen Lastspitzen (z.B. Wasserkocher) aber ruhig bleibt.

### 🌞 Dualer Microinverter-Support (ab V8.1)
Beide Hoymiles-WR werden gemeinsam geregelt. Wenn der Akku voll ist und Einspeisung droht, werden beide WR proportional zur aktuellen Ist-Leistung gedrosselt — automatisch, ohne feste Aufteilung.

### 🌙 Nachtmodus
Ohne Solarproduktion schaltet die Automation in den Nachtmodus: Der Sollwert orientiert sich direkt am Hausverbrauch, ohne PI-Regelung.

### 🔋 SOC-Schutz (Tiefentladeschutz)
Entladung stoppt automatisch unterhalb eines konfigurierbaren Mindest-SOC.

### 🔄 15-Tage-Zwangsladung
Nach länger als 15 Tagen ohne Vollladung (z.B. Dauerbewölkung) erzwingt die Automation eine Vollladung zum Schutz der Zellchemie.

### 🛡️ Watchdog & Datengültigkeit
Prüft ob alle Sensoren aktuelle und gültige Werte liefern. Bei Fehler: sauberer Stopp mit Filter-Reset. `continue_on_error: true` verhindert Abbruch bei Einzelfehlern.

### ✅ Manueller Modus
Über `input_boolean.nulleinspeisung_automatik` jederzeit deaktivierbar.

---

## Benötigte Helfer

```yaml
input_boolean:
  nulleinspeisung_automatik:
  akku_volladen:

input_number:
  pid_regler_gefilterter_error_speicher:
    min: -5000
    max: 5000
    step: 0.1
  solar_p_letzter_wert:
    min: 0
    max: 5000
    step: 0.1
  kopfspeicher_pi_integral:
    min: -5000
    max: 5000
    step: 0.01
  xt_max_ausgangsleistung:
    min: 0
    max: 5000
    step: 1
    initial: 2400
  xt_min_soc:
    min: 0
    max: 100
    step: 1
    initial: 10
  solar_schwelle:
    min: 0
    max: 500
    step: 1
    initial: 50

input_select:
  deadband_status:
    options:
      - neutral
      - import
      - export

input_datetime:
  letzte_akku_volladung:
    has_date: true
    has_time: true
```

---

## Verwendete Entitäten

| Entität | Beschreibung |
|---|---|
| `sensor.shellypro3em_leistung` | Netzleistung in W (+ = Bezug, − = Einspeisung) |
| `sensor.hausverbrauch_aktuell` | Aktueller Hausverbrauch gesamt in W |
| `sensor.sunenergyxt_500_pro_system_speicherlevel` | SOC des Akkus in % |
| `sensor.sunenergyxt_500_pro_systemleistung_am_netzanschluss` | Ist-Leistung am Netzanschluss in W |
| `number.sunenergyxt_500_pro_sollwert_leistung_netzanschluss` | Sollwert-Eingang SunEnergyXT |
| `sensor.hoymiles_hms_2000_4t_power` | Ist-Leistung HMS-2000 in W |
| `sensor.hoymiles_hms_1600_4t_power` | Ist-Leistung HMS-1600 in W |
| `binary_sensor.hoymiles_hms_2000_4t_reachable` | Erreichbarkeit HMS-2000 |
| `binary_sensor.hoymiles_hms_1600_4t_reachable` | Erreichbarkeit HMS-1600 |
| `number.hoymiles_hms_2000_4t_limit_nonpersistent_absolute` | Sollwert HMS-2000 in W |
| `number.hoymiles_hms_1600_4t_limit_nonpersistent_absolute` | Sollwert HMS-1600 in W |

---

## Installation

1. Helfer aus dem Abschnitt oben in Home Assistant anlegen
2. YAML-Datei in den Automations-Editor kopieren (YAML-Modus)
3. Alle Entitäts-IDs an das eigene Setup anpassen
4. `input_boolean.nulleinspeisung_automatik` einschalten

---

## Hinweise

> ⚠️ Die Entitäts-IDs sind auf mein Setup zugeschnitten. Vor dem Einspielen bitte alle `sensor.*`, `number.*`, `binary_sensor.*` und `input_*` Entitäten an die eigene Installation anpassen.

> ℹ️ Wer nur einen Hoymiles-WR hat: HMS-1600 Zeilen einfach weglassen — die Automation funktioniert mit nur dem HMS-2000 genauso.
