# RCA-Ticket-Analyzer

Portable, vollständig offline laufende Single-File-Webanwendung zum Verwalten von
Incident-Tickets: importierte oder manuell angelegte Tickets werden ausgewertet,
um Root Causes zu ermitteln und Maßnahmen festzuhalten, die eine Wiederholung des
Incidents verhindern. Die App konsolidiert Tickets automatisch zu wiederkehrenden
Root-Cause-Gruppen, priorisiert RCA-Fokusthemen nach Häufigkeit und erfasst je
Ticket ein Feedback (Status, Root-Cause-Familie, Evidenz, Impact,
Ressourcenverbrauch, Komplexität, Priorität, Präventionsmaßnahme, Notizen).

## Funktionen

- **Import & manuelle Anlage**: CSV-Import *und* manuelles Anlegen einzelner
  Incidents; beide Quellen werden gemeinsam analysiert.
- **Konsolidierung**: automatische Gruppierung nach Root-Cause-Muster, optional
  als konsolidierte Tabellenansicht mit Gruppenköpfen.
- **Priorisierung & Sortierung**: nach Impact, Ressourcenverbrauch, Komplexität
  oder RCA-Häufigkeit.

## Nutzung

`ITAS_RCA_Analyzer_Portable.html` im Browser öffnen — keine Installation, kein
Server, keine Netzwerkverbindung nötig. Details siehe [`README.txt`](README.txt).

- **CSV-Import**: Komma-, Semikolon- und Tabulator-Trennung, UTF-8/Windows-1252,
  BOM sowie mehrzeilige (in Anführungszeichen eingeschlossene) Felder.
- **Feedback**: wird im `localStorage` des Browsers gespeichert und kann als
  CSV oder JSON exportiert bzw. wieder importiert werden.

## Optimierungen

Die Änderungen dieser Version sind in [`OPTIMIZATIONS.md`](OPTIMIZATIONS.md)
dokumentiert (u. a. korrektes Einlesen mehrzeiliger CSV-Felder, vollständige
Feedback-Erzeugung unabhängig vom Filter, schnelleres Rendering).
