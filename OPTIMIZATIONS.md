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

Die Offline-/Portabilitäts-Eigenschaft bleibt unverändert.

## Neue Funktionen (Incident-Management)

Auf Basis der geschärften Aufgabenstellung — Incident-Tickets verwalten, Root
Causes ermitteln, Präventionsmaßnahmen festhalten — wurde die App erweitert:

8. **Manuelle Incident-Anlage.** Neue Karte „1b. Incident manuell anlegen"
   (Schlüssel optional/auto-vergeben, Zusammenfassung, Typ, Status,
   Beschreibung). Manuelle Tickets werden in `localStorage`
   (`itasManualTickets`) gespeichert, beim Start geladen und gemeinsam mit den
   importierten Tickets (`rebuild()`) analysiert. Doppelte Schlüssel werden
   abgewiesen.

9. **Konsolidierung in Gruppen.** Umschalter „Nach Gruppe konsolidieren"
   fasst die Tabelle nach erkanntem Muster/Root-Cause-Gruppe zusammen — mit
   Gruppenkopf (Anzahl angezeigt/gesamt, RCA-Fokus vs. Einzelfall) für einen
   übersichtlichen Bearbeitungsfluss.

10. **Sortierung nach Impact, Ressourcenverbrauch, Komplexität.** Drei neue
    Bewertungsfelder je Ticket (Niedrig/Mittel/Hoch) plus ein
    „Sortieren nach"-Auswahlfeld (zusätzlich RCA-Häufigkeit). Die Sortierung
    wirkt sowohl auf die flache Liste als auch — als Reihenfolge der Gruppen
    und innerhalb der Gruppen — im konsolidierten Modus.

11. **Präventionsfokus.** Das Maßnahmenfeld ist als
    „Fix-/Präventionsmaßnahme (verhindert Wiederholung)" gekennzeichnet.
    Ältere gespeicherte Datensätze werden um die neuen Felder ergänzt
    (Backfill), ohne bestehende Eingaben zu verlieren.

12. **Kartenansicht.** Umschalter „Ansicht: Tabelle / Karten". Die
    Kartenansicht respektiert Filter, Sortierung und Gruppierung
    (Gruppentitel als Abschnittsüberschriften). Filter-, Sortier- und
    Gruppierlogik ist in gemeinsame Helfer (`collectIdx`, `sortIdx`,
    `orderedGroups`) ausgelagert, die beide Ansichten nutzen.

13. **Titelleiste je Karte farblich einstellbar.** Farbwähler in der
    Kartenkopfzeile (und in der Detailansicht); die gewählte Farbe wird pro
    Ticket gespeichert (`color`). Die Textfarbe wird automatisch nach
    Kontrast (Luminanz) gewählt.

14. **Detailansicht (Modal).** Klick auf eine Karte öffnet ein Modal mit
    Metadaten und dem vollständigen, editierbaren Feedback-Formular
    (gemeinsame `feedbackFormHTML`-Funktion für Tabelle und Modal).
    Schließen per Button, Hintergrund-Klick oder Esc.

15. **Backup & Migration – Tickets überleben neue Versionen.** Die
    localStorage-Schlüssel (`itasManualTickets`, `itasRcaFeedback`) sind
    versionsstabil, sodass eine ausgetauschte HTML-Datei am selben Ort die
    Daten behält. Zusätzlich exportiert/importiert „5. Backup & Migration"
    ein vollständiges JSON (manuelle Tickets **und** Feedback) für den Umzug
    auf neue Versionen oder andere Rechner. Import führt additiv zusammen und
    akzeptiert auch ältere reine Feedback-Exporte. (Ende-zu-Ende getestet:
    leerer Speicher → Import → alle Tickets inkl. Farbe/Bewertung wieder da.)
