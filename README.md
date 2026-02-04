# Bacheloarbeit_Bingoel_Ilkaan_S0584398.-Wirtschaftsinformatik
KI-basierter Kaltakquise-Prototyp, entwickelt im Rahmen einer Bachelorarbeit.

Datensatz / Softwarepaket (Was?)

Softwarepaket:
Prototypische Python-Anwendung zur automatisierten Generierung personalisierter Kaltakquise-E-Mails mittels eines Large Language Models (LLM).

Art der Daten:
Synthetisch erzeugte strukturierte Kundendaten
Generierte Textdaten (E-Mail-Betreff und -Textkörper)
Metadaten (Prompts, Modellantworten, Zeitstempel)

Sprache:
Deutsch

Genutzte wissenschaftliche Methode:
Artefaktbasierte Forschung
Prototypische Implementierung
Quantitativ-qualitative Evaluation mittels standardisierter Befragung (Likert-Skalen)

Datenursprung (Wer?)

Autor:
Ilkaan Bingöl

Datenerhebung:
Alle verwendeten Kundendaten wurden synthetisch erzeugt und repräsentieren keine realen Personen.
Die E-Mail-Texte wurden durch ein Large Language Model auf Basis dieser synthetischen Daten generiert.

Lizenz (Software):
MIT License

Identifier:
Kein DOI vergeben (lokaler Forschungsprototyp im Rahmen einer Bachelorarbeit).

Zeitraum (Wann?)

Softwareentwicklung: Dezember 2025 – Januar 2026
Datengenerierung & Tests: fortlaufend während der Entwicklungsphase
Evaluation der E-Mails: Dezember 2025 – Januar 2026
Abgabe der Arbeit: 06.02.2026

Datenformate und -größe (Welche? Wie viel?)

Formate:
SQLite-Datenbank (.db)
Python-Quellcode (.py)
Textdaten (JSON-Strukturen innerhalb der Modellantworten)

Datenumfang (ungefähr):
Synthetische Kundendaten: < 1 MB
Generierte E-Mails inkl. Metadaten: < 1 MB
Gesamtdatenumfang: < 5 MB

Werkzeuge (inkl. Versionen)

Programmiersprache:
Python 3.x

Datenhaltung:
SQLite (über sqlite3)

KI / LLM:
OpenAI GPT-4-basierte API (z. B. gpt-4o)

Weitere Bibliotheken:
json
random
datetime
Entwicklungsumgebung:
Lokale Entwicklungsumgebung (z. B. Spyder)

Qualitätssicherung
- Verwendung eines festen Random-Seeds zur Reproduzierbarkeit synthetischer Daten
- Speicherung von Prompt und Roh-Modellantwort zur vollständigen Nachvollziehbarkeit
- JSON-Parsing mit Fehlerbehandlung (Fallback-Mechanismen)
- Trennung von Datenhaltung, Prompt-Erstellung und Modellaufruf
- Dokumentation aller Verarbeitungsschritte im Quellcode

Datenschutz
- Es werden keine personenbezogenen Daten realer Personen verarbeitet
- Alle Kundendaten sind vollständig synthetisch
- Keine DSGVO-relevanten Inhalte
- Keine Speicherung von API-Keys im Repository

