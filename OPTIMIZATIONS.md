# Analyse & Optimierung – ITAS RCA Analyzer

Ausgangspunkt war die portable Single-File-Anwendung
`ITAS_RCA_Analyzer_Portable.html` (client-seitiger CSV-Analyzer, keine
Netzwerk- oder Crawler-Funktion). Bei der Analyse wurden folgende Punkte
identifiziert und behoben.

## Korrektheit

1. **CSV-Parser zerstörte mehrzeilige Felder.**
   Der ursprüngliche `parseCSV` führte zunächst einen korrekten,
   anführungszeichen-bewussten Durchlauf aus, **verwarf dessen Ergebnis** und
   parste anschließend erneut über `text.split("\n")`. Dieser naive Split
   zerlegt jedes in Anführungszeichen stehende Feld mit Zeilenumbruch (bei
   Jira-`Beschreibung`/`Summary` sehr häufig) → Spalten verrutschen still.
   *Fix:* ein einziger Zustandsautomat, der Anführungszeichen, eingebettete
   Zeilenumbrüche und die drei Trennzeichen (`,` `;` Tab) sauber verarbeitet.
   Trennzeichen wird außerhalb von Anführungszeichen erkannt.

2. **Feedback-Datensätze wurden nur für sichtbare Zeilen erzeugt.**
   Die Anlage des Datensatzes erfolgte erst *nach* dem Filter-`return`. Bei
   aktivem Filter erhielten nur passende Tickets einen Datensatz — im
   Widerspruch zur Zusage „für jedes geladene Ticket" und mit unvollständigem
   CSV-/JSON-Export als Folge.
   *Fix:* `ensureFeedback()` legt in `analyse()` für **alle** Tickets einen
   Datensatz an; der Filter wirkt nur noch auf die Anzeige.

3. **Abgeleitete Felder blieben eingefroren.**
   `similar_count`/`rca_focus` wurden einmalig gesetzt und bei geändertem
   `minFreq` oder Datenbestand nicht aktualisiert.
   *Fix:* diese abgeleiteten Felder werden bei jeder Analyse aktualisiert,
   ohne die vom Nutzer eingegebenen Felder zu überschreiben.

## Performance

4. **Muster (`pattern`) wurden mehrfach berechnet** — einmal in `analyse`,
   erneut pro Zeile in `renderTable`. Jetzt einmalig pro Ticket in
   `S.patterns[]`/`S.keys[]` memoisiert.

5. **Tabelle wurde Zeile für Zeile angehängt** (Reflow pro Zeile) mit einem
   Event-Listener pro Feld. Jetzt ein einziger `innerHTML`-Schreibvorgang und
   **ein** delegierter `change`-Listener auf `<tbody>`.

6. **Filter renderte bei jedem Tastendruck neu.** Jetzt um 150 ms entprellt.

## Robustheit

7. **`localStorage`-Zugriff war ungeschützt** und kann bei `file://` oder im
   Privat-Modus eine Ausnahme werfen. Alle Zugriffe laufen nun über
   `persist()`/`loadStored()` mit `try/catch`; Speicherfehler werden dem
   Nutzer gemeldet (Export als Ausweichweg). `Object-URLs` werden nach dem
   Download freigegeben.

Verhalten, Oberfläche und die Offline-/Portabilitäts-Eigenschaft bleiben
unverändert.
