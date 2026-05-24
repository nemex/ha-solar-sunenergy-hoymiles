# Solar: Master-Steuerung SunEnergy XT 500 Pro
### Zero-Feed-In Regelung mit PI-Regler für Home Assistant — V7.8

---

## Was macht diese Automation?

Diese Automation regelt einen SunEnergy XT 500 Pro Hybrid-Wechselrichter in Home Assistant vollautomatisch auf Nulleinspeisung. Das bedeutet: Der Akku gibt immer genau so viel ab wie das Haus gerade braucht, ohne dabei Strom ins öffentliche Netz zu drücken.

Die Messung erfolgt über einen Shelly Pro 3EM am Hauptzähler. Der Messwert wird mit einem asymmetrischen Filter geglättet – bei echtem Mehrbedarf reagiert die Regelung schnell, bei kurzen Lastspitzen wie einem Wasserkocher bleibt sie ruhig. Die eigentliche Regelung übernimmt ein PI-Regler, der den Sollwert am Netzanschlusspunkt alle 5 Sekunden anpasst. Damit sich der Sollwert nicht zu schnell ändert, gibt es ein Rate-Limiting und eine Schreibschwelle – kleine Schwankungen werden ignoriert.

Geladen wird der Akku ausschließlich mit Solarüberschuss. Produziert die Anlage mehr als das Haus gerade verbraucht, fließt die überschüssige Energie automatisch in den Akku. Es wird also nie gezielt Netzstrom zum Laden verwendet – außer in einem Ausnahmefall: Wurde der Akku länger als 15 Tage nicht vollgeladen, erzwingt die Automation eine Zwangsladung zum Schutz der Zellchemie.

Ein konkretes Beispiel: Die Solaranlage produziert gerade 1200 W, das Haus verbraucht aber 2000 W. Die fehlenden 800 W müssten eigentlich aus dem Netz kommen. Liegt der Akkustand dabei unter dem konfigurierbaren Mindest-SOC (Standard: 18 %), greift der SOC-Schutz und die Automation holt die fehlende Leistung aus dem Netz, anstatt den Akku weiter zu entladen. Der Akku bleibt dann in Reserve, bis er wieder durch Solarüberschuss geladen wird. Ist der Akkustand ausreichend, gleicht er die Differenz zwischen Solar und Verbrauch automatisch aus – das Netz bleibt außen vor.

Nachts, wenn keine Solar-Produktion vorhanden ist, wechselt die Automation automatisch in einen einfacheren Modus und orientiert sich direkt am Hausverbrauch – der Akku entlädt sich dann kontrolliert, bis entweder der Mindest-SOC erreicht ist oder der Morgen kommt.

Zusätzlich überwacht ein Watchdog ob die Sensoren noch aktuelle Werte liefern. Bei ungültigen Daten stoppt die Regelung sauber und setzt alle internen Speicher zurück. Über einen einfachen Schalter lässt sich die Automatik jederzeit deaktivieren.

> ⚠️ Die Entitäts-IDs sind auf mein Setup zugeschnitten und müssen vor dem Einspielen an die eigene Installation angepasst werden.

---

## Hardware-Voraussetzungen

| Komponente | Funktion |
|---|---|
| SunEnergy XT 500 Pro | Hybrid-Wechselrichter / Batteriespeicher |
| Shelly Pro 3EM | Netzleistungsmessung am Hauptzähler |
| Shelly Pro 1pm | Messung der Microinverter-Leistung (Hoymiles) |
| Hoymiles Microinverter | Optionale zweite Solarquelle |

---

## Features V7.8

**Asymmetrischer Tiefpassfilter**
Bei Netzbezug über 30 W reagiert der Regler schnell (70 % Neuwert). Sonst gedämpft (15 % Neuwert) – kurze Lastspitzen werden ignoriert.

**PI-Regler mit Rate-Limiting**
Proportionalanteil ab 50 W Abweichung, Integralanteil für langsame Drifts, Anti-Windup, max. 100 W Änderung pro Schritt, Schreibschwelle 50 W.

**Nachtmodus**
Ohne Solarproduktion direkter Bezug auf Hausverbrauch, ohne PI-Regelung.

**SOC-Schutz**
Entladung stoppt unterhalb des konfigurierbaren Mindest-SOC.

**15-Tage-Zwangsladung**
Automatische Vollladung nach längerer Schlechtwetterperiode zum Zellschutz.

**Watchdog & Datengültigkeit**
Prüft ob alle Sensoren aktuelle und gültige Werte liefern. Bei Fehler: sauberer Stopp mit Filter-Reset.

**Manueller Modus**
Regelung per `input_boolean` jederzeit deaktivierbar.

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

input_select:
  deadband_status:
    options:
      - im Deadband
      - Bezug
      - Einspeisung

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
| `sensor.shellypro1pm_hoymiles_leistung` | Leistung Hoymiles Microinverter |
| `sensor.hausverbrauch_aktuell` | Aktueller Hausverbrauch gesamt |
| `sensor.sunenergyxt_500_pro_system_speicherlevel` | SOC des Akkus in % |
| `sensor.sunenergyxt_500_pro_systemleistung_am_netzanschluss` | Ist-Leistung am Netzanschluss |
| `number.sunenergyxt_500_pro_sollwert_leistung_netzanschluss` | Sollwert-Eingang Wechselrichter |
| `sensor.energy_production_today_2` | Tagesertrag Solar in kWh |

---

## Installation

1. Helfer aus dem Abschnitt oben in Home Assistant anlegen
2. YAML-Datei in den Automations-Editor kopieren (YAML-Modus)
3. Alle Entitäts-IDs an das eigene Setup anpassen
4. `input_boolean.nulleinspeisung_automatik` einschalten
5. PI-Integral-Speicher auf 0 setzen und Automation neu starten

---

## Konfigurierbare Parameter

| Parameter | Standardwert | Beschreibung |
|---|---|---|
| `min_soc` | 18 % | Untergrenze SOC-Schutz |
| `kp` | 0.6 | Proportionalbeiwert |
| `ki` | 0.04 | Integralbeiwert |
| `max_step` | 100 W | Maximale Sollwertänderung pro Schritt |
| Schreibschwelle | 50 W | Minimale Abweichung zum Schreiben |
| kP-Aktivierungsschwelle | 50 W | Ab wann der P-Anteil aktiv wird |
| Zwangsladung | 15 Tage | Intervall ohne Vollladung |

---

## Versionsverlauf

| Version | Änderung |
|---|---|
| **V7.8** | kP-Schwelle 100 W → 50 W, `continue_on_error`, Filter-Reset im Sicherheits-Stopp, asymmetrischer Filter, Nachtmodus auf `haus_p`, SOC-Schutz |
| V7.x | PI-Regler, Rate-Limiting, Anti-Windup, Watchdog, Datengültigkeitsprüfung |
| V6.x | Einfache Schwellwert-Steuerung mit gefiltertem Messwert |

---

## Lizenz

MIT – frei verwendbar, Verbesserungen und Pull Requests willkommen!
