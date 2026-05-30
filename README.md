# Solar: Master-Steuerung SunEnergy XT 500 Pro
### Zero-Feed-In Regelung mit PI-Regler und Hoymiles-Drosselung für Home Assistant — V8.6

---

## Was macht diese Automation?

Diese Automation regelt einen SunEnergy XT 500 Pro Hybrid-Wechselrichter in Home Assistant vollautomatisch auf Nulleinspeisung. Das bedeutet: Der Akku gibt immer genau so viel ab wie das Haus gerade braucht, ohne dabei Strom ins öffentliche Netz zu drücken.

Die Messung erfolgt über einen Shelly Pro 3EM am Hauptzähler. Der Messwert wird mit einem asymmetrischen Filter geglättet – bei echtem Mehrbedarf reagiert die Regelung schnell, bei kurzen Lastspitzen wie einem Wasserkocher bleibt sie ruhig. Die eigentliche Regelung übernimmt ein PI-Regler, der den Sollwert am Netzanschlusspunkt alle 5 Sekunden anpasst. Dabei wird zwischen drei Zuständen unterschieden: Netzbezug (import), Einspeisung (export) und einem neutralen Bereich dazwischen, in dem die Regelung ruhig bleibt.

Geladen wird der Akku ausschließlich mit Solarüberschuss. Produziert die Anlage mehr als das Haus gerade verbraucht, fließt die überschüssige Energie automatisch in den Akku. Es wird nie gezielt Netzstrom zum Laden verwendet – außer wenn der Akku länger als 15 Tage nicht vollgeladen wurde. In diesem Fall erzwingt die Automation eine Zwangsladung zum Schutz der Zellchemie.

Ein konkretes Beispiel: Die Solaranlage produziert gerade 1200 W, das Haus verbraucht aber 2000 W. Die fehlenden 800 W müssten eigentlich aus dem Netz kommen. Liegt der Akkustand dabei unter dem konfigurierbaren Mindest-SOC, greift der SOC-Schutz und die Automation holt die fehlende Leistung aus dem Netz, anstatt den Akku weiter zu entladen. Der Akku bleibt dann in Reserve, bis er wieder durch Solarüberschuss geladen wird. Ist der Akkustand ausreichend, gleicht er die Differenz automatisch aus – das Netz bleibt außen vor.

Neu in V8.6: Neben dem Solar-Feedforward gibt es jetzt auch einen Haus-Feedforward. Schaltet sich ein großer Verbraucher ein oder aus – zum Beispiel eine Waschmaschine oder ein E-Auto-Ladegerät – erkennt die Automation den Lastsprung sofort und passt den Sollwert ohne Verzögerung an, noch bevor der PI-Regler reagiert hat. Das verhindert kurze Netzspitzen bei plötzlich steigendem Verbrauch.

Ab einem Akkustand von 96 % drosselt die Automation zusätzlich die Hoymiles-Wechselrichter aktiv, um Einspeisung ins Netz zu verhindern. Die Drosselung verteilt sich proportional auf beide Hoymiles-Geräte. Sobald der SOC wieder unter 96 % fällt, werden beide auf Vollleistung zurückgesetzt.

Nachts wechselt die Automation automatisch in einen einfacheren Modus und orientiert sich direkt am Hausverbrauch – der Akku entlädt sich kontrolliert bis zum Mindest-SOC.

Zusätzlich überwacht ein Watchdog ob die Sensoren noch aktuelle Werte liefern. Bei ungültigen Daten oder deaktivierter Automatik stoppt die Regelung sauber. Über einen einfachen Schalter lässt sich die Automatik jederzeit deaktivieren.

> ⚠️ Die Entitäts-IDs sind auf mein Setup zugeschnitten und müssen vor dem Einspielen an die eigene Installation angepasst werden.

---

## Hardware-Voraussetzungen

| Komponente | Funktion |
|---|---|
| SunEnergy XT 500 Pro | Hybrid-Wechselrichter / Batteriespeicher |
| Shelly Pro 3EM | Netzleistungsmessung am Hauptzähler |
| Hoymiles HMS-2000-4T | Microinverter Solaranlage 1 |
| Hoymiles HMS-1600-4T | Microinverter Solaranlage 2 (optional) |
| OpenDTU | Lokale Hoymiles-Integration via MQTT ohne Cloud |

---

## Features V8.6

**Asymmetrischer Tiefpassfilter**
Bei Netzbezug über 30 W reagiert der Regler schnell (70 % Neuwert). Sonst gedämpft (15 % Neuwert) – kurze Lastspitzen werden ignoriert.

**PI-Regler mit dynamischem kP**
Der Proportionalbeiwert passt sich der Abweichung an: 0.5 bei starkem Defizit (> 200 W), 0.2 bei mittlerem (> 50 W), 0.08 bei kleinem. Integralanteil baut langsame Drifts ab.

**Solar-Feedforward**
Bei plötzlichen Solarsprüngen über 150 W wird ein Vorsteuerungsterm berechnet, der die Regelabweichung antizipiert und die Reaktionszeit deutlich verkürzt.

**Haus-Feedforward (neu in V8.6)**
Bei Lastsprüngen des Hausverbrauchs über 200 W – zum Beispiel wenn eine Waschmaschine oder ein Ladegerät einschaltet – wird der Vorsteuerungsterm sofort auf den Lastsprung angepasst. Das Rate-Limiting wird dabei ebenfalls aufgehoben, sodass der Sollwert in einem Schritt auf den neuen Bedarf springen kann.

**Dreistufiger Deadband**
Zwischen −50 W und +50 W (tagsüber) bzw. −80 W und +50 W (nachts) ist die Regelung neutral.

**Dynamisches Rate-Limiting**
Maximale Sollwertänderung pro Schritt: bei Solar- oder Lastsprüngen und starker Einspeisung bis zu 2400 W, normal 100–200 W je nach Situation.

**Hoymiles-Drosselung ab SOC 96 %**
Beide Wechselrichter werden aktiv gedrosselt wenn der Akku fast voll ist. Proportionale Verteilung auf HMS-2000 und HMS-1600. Automatischer Reset auf Vollleistung bei SOC < 96 %.

**Tag-/Nachtmodus**
Automatischer Wechsel abhängig von der Solarproduktion. Schwellwert konfigurierbar.

**SOC-Schutz**
Entladung stoppt unterhalb des konfigurierbaren Mindest-SOC.

**15-Tage-Zwangsladung**
Automatische Vollladung nach längerer Schlechtwetterperiode zum Zellschutz.

**Watchdog & Datengültigkeit**
Prüft ob alle Sensoren aktuelle und gültige Werte liefern. Bei Fehler sauberer Stopp.

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
  haus_p_letzter_wert:        # NEU in V8.6
    min: 0
    max: 5000
    step: 0.1
  kopfspeicher_pi_integral:
    min: -150
    max: 150
    step: 0.01
  xt_min_soc:
    min: 0
    max: 50
    step: 1
    initial: 10
  xt_max_ausgangsleistung:
    min: 0
    max: 2400
    step: 100
    initial: 2400
  solar_schwelle:
    min: 0
    max: 200
    step: 10
    initial: 50

input_select:
  deadband_status:
    options:
      - import
      - export
      - neutral

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
| `sensor.hoymiles_hms_2000_4t_power` | Leistung HMS-2000-4T in W |
| `sensor.hoymiles_hms_1600_4t_power` | Leistung HMS-1600-4T in W |
| `binary_sensor.hoymiles_hms_1600_4t_reachable` | Online-Status HMS-1600-4T |
| `sensor.hausverbrauch_aktuell` | Aktueller Hausverbrauch gesamt in W |
| `sensor.sunenergyxt_500_pro_system_speicherlevel` | SOC des Akkus in % |
| `sensor.sunenergyxt_500_pro_systemleistung_am_netzanschluss` | Ist-Leistung am Netzanschluss in W |
| `number.sunenergyxt_500_pro_sollwert_leistung_netzanschluss` | Sollwert-Eingang Wechselrichter |
| `number.hoymiles_hms_2000_4t_limit_nonpersistent_absolute` | Leistungslimit HMS-2000 in W |
| `number.hoymiles_hms_1600_4t_limit_nonpersistent_absolute` | Leistungslimit HMS-1600 in W |

---

## Installation

1. Helfer aus dem Abschnitt oben in Home Assistant anlegen – neu in V8.6: `input_number.haus_p_letzter_wert`
2. YAML-Datei in den Automations-Editor kopieren (YAML-Modus)
3. Alle Entitäts-IDs an das eigene Setup anpassen
4. `input_boolean.nulleinspeisung_automatik` einschalten
5. PI-Integral-Speicher (`input_number.kopfspeicher_pi_integral`) auf 0 setzen
6. Automation neu starten

> 💡 Wer nur einen Hoymiles-Wechselrichter hat, kann alle `hms_1600`-Variablen und Aktionen entfernen. Der `anteil_2000` wird dann automatisch auf `1.0` gesetzt.

---

## Konfigurierbare Parameter

| Parameter | Entität / Wert | Beschreibung |
|---|---|---|
| Mindest-SOC | `input_number.xt_min_soc` | Untergrenze Entladeschutz (Standard: 10 %) |
| Max. Ausgangsleistung | `input_number.xt_max_ausgangsleistung` | Obergrenze Wechselrichter (Standard: 2400 W) |
| Solar-Schwelle | `input_number.solar_schwelle` | Ab wann Tagmodus aktiv (Standard: 50 W) |
| HMS-Drosselung ab | SOC 96 % | Fest kodiert – bei Bedarf im YAML anpassen |
| Haus-Feedforward ab | 200 W Lastsprung | Fest kodiert – bei Bedarf im YAML anpassen |
| Zwangsladung | 15 Tage | Intervall ohne Vollladung |
| kP dynamisch | 0.08 / 0.2 / 0.5 | Je nach Abweichungsgröße |
| kI | 0.005 | Integralbeiwert |
| I-Limit | ±150 | Anti-Windup Begrenzung |

---

## Versionsverlauf

Siehe [CHANGELOG.md](CHANGELOG.md)

---

## Lizenz

MIT – frei verwendbar, Verbesserungen und Pull Requests willkommen!
