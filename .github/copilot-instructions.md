# Anweisungen für KI-Assistenten in diesem Repo
 
Lies zuerst `README.md` – dort stehen Architektur, Datenmodell und Spielablauf.
 
## Projekt in Kürze
- „Der Große Preis“-Quiz für eine Weihnachtsfeier. **Die komplette App ist `index.html`** (HTML + CSS + JS, ein `<script type="module">`).
- Hosting: GitHub Pages (statisch). Live-Sync: Firebase Realtime Database, Pfad `/quiz`, per CDN-Import geladen.
- Zuschauer-Ansicht unter `/`, Game-Master-Ansicht unter `/#gm` (Passwort `GM_PASSWORD`).
## Regeln für Änderungen
- **Eine Datei bleibt eine Datei.** Kein Build-Step, kein npm, keine Frameworks, keine neuen Dateien für Code. Nur Vanilla JS + CSS.
- Antworten, Kommentare und UI-Texte auf **Deutsch**.
- Den bestehenden Stil beibehalten: State-Änderungen nur über `commit(s => { ... })` bzw. neue Methoden im Objekt `A`; Klicks über `data-act`-Attribute und den zentralen Click-Handler; Rendering über die `*HTML()`-Funktionen.
- Neue Felder im State immer in `defaultState()` **und** `normalize()` ergänzen (Firebase entfernt leere Arrays/`null` und liefert Arrays teils als Objekte).
- Punktestand wird berechnet (`scores()`), nicht gespeichert – nichts doppelt speichern.
- Alles muss **mobil (ca. 380 px Breite)** gut aussehen; das Board hat 6 Spalten.
- Farben über die CSS-Variablen in `:root` (Gold `--gold`, hansgrohe-Türkis `--teal`). Design: winterlich-weihnachtlich, Felder als Geschenke (weiß, erspielt in Teamfarbe).
- Nutzerdaten immer mit `esc()` ausgeben (innerHTML).
- `FIREBASE_CONFIG` und `GM_PASSWORD` nicht ändern, außer ausdrücklich verlangt.
- Keine Testdaten in die echte Firebase-DB schreiben. Zum Testen `databaseURL` lokal temporär auf `""` setzen (lokaler Testmodus) – und diese Änderung nicht committen.
## Nach Änderungen
- Kurz beschreiben, was geändert wurde und wie man es testet (zwei Tabs: `index.html` und `index.html#gm`).
- README aktualisieren, wenn sich Bedienung, Datenmodell oder Setup ändern.
 