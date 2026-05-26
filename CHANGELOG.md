# Changelog

## [V8.1] – 2026-05

### Geändert
- Proportionaler Split vereinfacht: kein uhrzeit-basierter Fallback mehr, rein aus Ist-Leistungs-Verhältnis
- `anteil_2000` Dummy-Wert bei < 50W Gesamtleistung auf 0.5 (irrelevant, da nie gedrosselt wird)

## [V8.0] – 2026-05

### Hinzugefügt
- Dualer Microinverter-Support: HMS-2000-4T + HMS-1600-4T
- `solar_p` jetzt als Summensensor aus beiden WR (statt Shelly-Einzelmessung)
- Proportionaler Sollwert-Split basierend auf aktuellen Ist-Leistungen
- Fallback auf HMS-2000 allein wenn HMS-1600 offline (`reachable = off`)
- Hoymiles-Drosselung wenn Akku voll und Einspeisung droht

### Geändert
- `data_valid` prüft nur noch HMS-2000 als Pflicht (HMS-1600 optional)
- Normalbetrieb: WR laufen immer mit voller Leistung, Drosselung nur bei SOC = 100% + Einspeisung

## [V7.8] – 2026-03

### Geändert
- kP-Schwelle von 100 W auf 50 W gesenkt für schnellere Reaktion
- `continue_on_error: true` an allen WR-Befehlen
- Filter-Reset im Sicherheits-Stopp
- Asymmetrischer Filter (0.85/0.30 statt symmetrisch)
- Nachtmodus direkt auf `haus_p` ohne PI-Regelung
- SOC-Schutz (Tiefentladeschutz) hinzugefügt

## [V7.x] – 2026-01 bis 2026-02

### Hinzugefügt
- PI-Regler (Proportional + Integral)
- Anti-Windup Logik
- Rate-Limiting (max. 100 W Änderung pro Regelschritt)
- Schreibschwelle (Sollwert nur schreiben bei Änderung > 50 W)
- Watchdog: prüft ob Shelly Pro 3EM in den letzten 60 Sekunden aktualisiert hat
- Datengültigkeitsprüfung: alle relevanten Sensoren auf `unknown`/`unavailable`
- Wolken-Feedforward via `solar_delta`
- Dynamisches kP (3 Stufen je nach Fehlergröße)
- Sauberer Stopp mit Filter-Reset bei Fehler

## [V6.x] – 2025-12 bis 2026-01

### Hinzugefügt
- Erster gefilterter Messwert (symmetrischer Tiefpassfilter)
- 15-Tage-Zwangsladung zum Schutz der Zellchemie
- Manueller Modus über `input_boolean.nulleinspeisung_automatik`
- Akku-Volladen Modus über `input_boolean.akku_volladen`
- Logging des letzten Vollladezeitpunkts in `input_datetime.letzte_akku_volladung`

## Frühere Versionen

Einfache Schwellwert-Steuerung ohne Filterung und ohne PI-Regler.
