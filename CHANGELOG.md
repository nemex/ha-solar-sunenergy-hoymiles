# Changelog

Alle wichtigen Änderungen an der Solar Master-Steuerung werden hier dokumentiert.

---

## [V8.5.1] – 2026-05

### Geändert
- HMS-Drosselung greift ab SOC 96 % statt 98 % – verhindert zuverlässig Einspeisung, bevor der SunEnergy XT intern bei ~97 % aufhört zu laden

---

## [V8.5.0] – 2026-05

### Hinzugefügt
- Hoymiles HMS-1600-4T als zweite Solarquelle integriert
- `hms_1600_online` Erreichbarkeitsprüfung – Anlage läuft auch weiter wenn HMS-1600 offline ist
- Proportionale Leistungsverteilung zwischen HMS-2000 und HMS-1600 basierend auf aktuellem Anteil (`anteil_2000`)
- Automatischer Reset beider Wechselrichter auf Vollleistung wenn SOC wieder unter 96 %

---

## [V8.4.x] – 2026-04

### Hinzugefügt
- Hoymiles HMS-2000-4T Drosselung ab SOC 98 % (später auf 96 % korrigiert)
- Aktive Leistungsbegrenzung via `number.hoymiles_hms_2000_4t_limit_nonpersistent_absolute`
- Schreibschwelle für HMS-Drosselung: nur bei Änderung ≥ 50 W wird geschrieben

---

## [V8.3.x] – 2026-04

### Hinzugefügt
- Feedforward-Steuerung: bei Solarsprüngen > 150 W wird ein Vorsteuerungsterm berechnet, der Regelabweichungen antizipiert
- Dynamisches Rate-Limiting: bei Solarsprüngen und starker Einspeisung bis 2400 W/Schritt, normal 100–200 W
- Konfigurierbare Parameter über `input_number`-Helfer (min_soc, max_p, solar_schwelle)

---

## [V8.2.x] – 2026-03

### Geändert
- Dreistufiger Deadband (import / export / neutral) ersetzt einfache Schwellwerte
- Deadband-Low nachts auf −80 W erweitert (tagsüber −50 W) für ruhigeres Nachtverhalten
- Integralspeicher-Abbau differenziert: 0.90/Zyklus nachts, 0.97/Zyklus bei neutralem Deadband

### Hinzugefügt
- Dynamischer kP: 0.08 / 0.2 / 0.5 abhängig von der Abweichungsgröße
- Anti-Windup verbessert: berücksichtigt Vorzeichen des Fehlers

---

## [V8.1.x] – 2026-03

### Hinzugefügt
- Tag-/Nachtmodus mit konfigurierbarer Solar-Schwelle
- `akku_volladen` schaltet sich nach Vollladung automatisch aus und schreibt Zeitstempel

---

## [V7.8] – 2026-02

### Geändert
- kP-Aktivierungsschwelle von 100 W auf 50 W gesenkt – schnellere Reaktion bei kleinen Defiziten
- Nachtmodus setzt Sollwert direkt auf `haus_p`
- Asymmetrischer Tiefpassfilter eingeführt (70/30 bei Bezug, 15/85 sonst)

### Hinzugefügt
- `continue_on_error: true` bei allen Sollwert-Schreibvorgängen
- Filter-Reset im Sicherheits-Stopp
- SOC-Schutz: unterhalb `min_soc` wird nicht mehr entladen

---

## [V7.x] – 2026-01 bis 2026-02

### Hinzugefügt
- PI-Regler mit Anti-Windup
- Rate-Limiting und Schreibschwelle
- Watchdog und Datengültigkeitsprüfung
- Sauberer Stopp mit definiertem Fallback

---

## [V6.x] – 2025-12 bis 2026-01

### Hinzugefügt
- Erster gefilterter Messwert (symmetrischer Tiefpassfilter)
- 15-Tage-Zwangsladung
- Manueller Modus und Akku-Volladen Modus
- Logging des letzten Vollladezeitpunkts

### Frühere Versionen
- Einfache Schwellwert-Steuerung ohne Filterung und ohne PI-Regler
