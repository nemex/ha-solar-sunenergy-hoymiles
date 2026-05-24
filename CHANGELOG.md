# Changelog

Alle wichtigen Änderungen an der Solar Master-Steuerung werden hier dokumentiert.

---

## [V7.8] – 2026-05

### Geändert
- kP-Aktivierungsschwelle von 100 W auf 50 W gesenkt – schnellere Reaktion bei kleinen Defiziten
- Nachtmodus setzt Sollwert jetzt direkt auf `haus_p` statt auf einen festen Wert
- Asymmetrischer Tiefpassfilter: bei Netzbezug > 30 W schnelle Reaktion (70/30), sonst gedämpft (15/85)

### Hinzugefügt
- `continue_on_error: true` bei allen Sollwert-Schreibvorgängen – Automation bricht bei Verbindungsproblemen nicht ab
- Filter-Reset (`pid_regler_gefilterter_error_speicher` und `kopfspeicher_pi_integral` auf 0) im Sicherheits-Stopp
- SOC-Schutz: unterhalb `min_soc` (Standard 18 %) wird nicht mehr entladen

---

## [V7.x] – 2026-03 bis 2026-04

### Hinzugefügt
- PI-Regler mit konfigurierbaren Werten für kP und kI
- Anti-Windup für den Integralanteil
- Rate-Limiting: max. 100 W Sollwertänderung pro Regelschritt
- Schreibschwelle: Sollwert wird nur geschrieben wenn Änderung > 50 W
- Watchdog: prüft ob Shelly Pro 3EM in den letzten 60 Sekunden aktualisiert hat
- Datengültigkeitsprüfung: alle relevanten Sensoren auf `unknown`/`unavailable` prüfen
- Sauberer Stopp mit definiertem Fallback bei Fehler

---

## [V6.x] – 2026-01 bis 2026-02

### Hinzugefügt
- Erster gefilterter Messwert (symmetrischer Tiefpassfilter)
- 15-Tage-Zwangsladung zum Schutz der Zellchemie
- Manueller Modus über `input_boolean.nulleinspeisung_automatik`
- Akku-Volladen Modus über `input_boolean.akku_volladen`
- Logging des letzten Vollladezeitpunkts in `input_datetime.letzte_akku_volladung`

### Frühere Versionen
- Einfache Schwellwert-Steuerung ohne Filterung und ohne PI-Regler
