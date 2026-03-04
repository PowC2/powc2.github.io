# Import der Sportstruktur über CSV-Vorlage

## Ziel
Massenimport von **Kategorien**, **Ligen** und **Teams** in die aktive Saison des Clubs mithilfe einer CSV-Vorlage ermöglichen.

Diese Anleitung beschreibt das **derzeit implementierte Verhalten** im Bildschirm für Sportstruktur.

## Verwendungsort
- Bildschirm: `SportStructureScreen`
- Button: **CSV-Vorlage importieren**
- Voraussetzung: Benutzer mit Club-Management-Rechten und aktiviertem Bearbeitungsmodus.

## Referenzdateien
- Basisvorlage: `docs/templates/sport_structure_import_template.csv`
- Gültiges Beispiel: `docs/templates/sport_structure_import_example_valid_en.csv`
- Ungültiges Beispiel: `docs/templates/sport_structure_import_example_invalid_en.csv`

## Erforderliches CSV-Format
Erforderliche Header (empfohlene Reihenfolge):

```csv
entity,name,category,league
```

### Spalten
- `entity`: Zeilentyp. Zulässige Werte:
  - `category`
  - `league`
  - `team`
- `name`: Hauptname der Entität.
- `category`: nur erforderlich bei `entity=team`.
- `league`: nur erforderlich bei `entity=team`.

## Validierungsregeln
Die Validierung läuft vor dem Einfügen:

1. Datei muss Header + mindestens eine Datenzeile enthalten.
2. Spalten `entity` und `name` müssen vorhanden sein.
3. Wenn `entity=team`, sind `category` und `league` erforderlich.
4. Jeder `entity`-Wert außerhalb von `category|league|team` ist ungültig.
5. Für Teams muss referenzierte Kategorie/Liga existieren:
   - entweder bereits in der aktiven Saison-DB, oder
   - im selben CSV über `category`-/`league`-Zeilen erstellt.

Bei Validierungsfehlern wird der Import **nicht ausgeführt** und eine Fehlerzusammenfassung angezeigt.

## Einfügeverhalten
- Neue Kategorien: werden eingefügt, wenn in aktiver Saison nicht vorhanden (normalisierter Namensvergleich).
- Neue Ligen: gleiches Verhalten.
- Neue Teams: werden eingefügt, wenn Tupel `name+category+league` nicht existiert.
- Vorhandene Daten werden nicht gelöscht.
- Vorhandene Namen werden nicht aktualisiert (append-orientiertes Verhalten).

## Textnormalisierung
Vor Vergleich/Einfügen:
- `trim` wird angewendet
- mehrere Leerzeichen werden auf eines reduziert

Beispiel:
- `"  Senior   A  "` → `"Senior A"`

## Gültiges Beispiel
```csv
entity,name,category,league
category,U18,,
category,U16,,
league,Premier,,
league,Regional,,
team,U18 A,U18,Premier
team,U18 B,U18,Regional
team,U16 A,U16,Regional
```

Erwartetes Ergebnis:
- Erstellte Kategorien: 2 (falls fehlend)
- Erstellte Ligen: 2 (falls fehlend)
- Erstellte Teams: 3 (falls fehlend)

## Ungültiges Beispiel
```csv
entity,name,category,league
team,Team ohne Liga,U18,
foo,Unbekannter Zeilentyp,,
team,,U18,Premier
team,Team mit fehlender Referenz,U14,Premier
```

Erwartete Fehler:
- `team`-Zeile ohne `league`;
- ungültiges `entity` (`foo`);
- leeres `name`;
- Referenz auf nicht vorhandene Kategorie (`U14`).

## Empfohlener Ablauf
1. Basisvorlage herunterladen.
2. Zuerst Kategorien und Ligen ausfüllen.
3. Teams mit exakten Kategorie-/Ligennamen ergänzen.
4. In Staging importieren.
5. Erstellungszusammenfassung prüfen.
6. In Produktion wiederholen.

## Best Practices
- Einheitliche Benennung verwenden (z. B. `U18 A`, `U18 B`).
- Typografische Varianten für dieselbe Entität vermeiden.
- In Blöcken importieren (erst Grundstruktur, dann Erweiterungen).

## Aktueller Umfang
Dieser Import unterstützt aktuell:
- Kategorien
- Ligen
- Teams

Noch nicht enthalten:
- Massenzuweisung von Mitgliedern
- Massenimport von `player_profile`
- Massen-Update/-Löschen

## Fehlerbehebung
### "CSV requires columns: entity,name,category,league"
Header ist falsch. Erste Zeile prüfen.

### "Template has no data"
Datei enthält nur den Header oder ist leer.

### "team requires category and league"
Team-Zeile ist unvollständig.

### "category/league does not exist"
Erstellungszeilen im selben CSV hinzufügen oder Datensätze zuerst in der App anlegen.
