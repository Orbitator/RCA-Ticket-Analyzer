# RCA-Ticket-Analyzer

Portable, vollständig offline laufende Single-File-Webanwendung zur Auswertung von
Jira-/CRM-Ticket-Exporten (CSV). Die App clustert Tickets zu wiederkehrenden
Root-Cause-Mustern, priorisiert RCA-Fokusthemen nach Häufigkeit und erfasst je
Ticket ein Feedback (Status, Root-Cause-Familie, Evidenz, Priorität, Maßnahme,
Notizen).

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
