# 🎄 Der Große Preis – Weihnachtsfeier-Quiz
 
Ein Quiz im Stil von „Der Große Preis“ als Web-App: Der Game Master (GM) steuert das Spiel auf dem Laptop, alle anderen schauen live auf dem Handy zu (Board, aktuelle Frage, Punktestand, Reihenfolge). Gebaut für die Abteilungs-Weihnachtsfeier 2026, weil vor Ort kein Beamer/Bildschirm verfügbar war.
 
- **Zuschauer-URL:** `https://<github-user>.github.io/<repo>/`
- **Game-Master-URL:** `https://<github-user>.github.io/<repo>/#gm` (Passwort, siehe unten)
---
 
## Architektur (bewusst minimal)
 
| Teil | Zweck |
|---|---|
| `index.html` | **Die komplette App** (HTML + CSS + JS in einer Datei, kein Build, keine Dependencies) |
| GitHub Pages | Hostet `index.html` statisch |
| Firebase Realtime Database (Spark/kostenlos) | Hält den gemeinsamen Spielstand; alle Clients hören live mit (`onValue`) |
 
- Firebase wird per CDN (`gstatic.com/firebasejs/10.12.2/...`) als ES-Modul geladen – nichts zu installieren.
- **Kein Login/kein Backend-Code.** Der GM-Bereich ist nur durch ein Passwort im JS geschützt (`GM_PASSWORD`). Das ist absichtlich simpel: Wer den Quelltext liest, kann Passwort und Fragen sehen. Für eine interne Feier akzeptiert.
- **Lokaler Testmodus:** Ist `FIREBASE_CONFIG.databaseURL` leer, speichert die App in `localStorage` und synchronisiert nur zwischen Tabs desselben Browsers. Praktisch zum Entwickeln ohne die echte DB zu verändern.
### Konfiguration (oben im `<script>` in `index.html`)
 
```js
const FIREBASE_CONFIG = { ... };   // Web-App-Config aus der Firebase-Konsole
const GM_PASSWORD = "weihnachten"; // Passwort für #gm
const DB_PATH = "quiz";            // Pfad in der Realtime Database
```
 
Aktuelles Firebase-Projekt: `der-grosse-preis-27e6a` (Region `europe-west1`).
 
---
 
## Setup von null (ca. 15 min)
 
1. **Firebase**
   1. https://console.firebase.google.com → Projekt hinzufügen (Analytics nicht nötig).
   2. *Build → Realtime Database → Datenbank erstellen*, Standort Belgien (`europe-west1`), „Testmodus“.
   3. Tab **Regeln** ersetzen und veröffentlichen (sonst läuft der Testmodus nach 30 Tagen ab):
```json
{
  "rules": {
    "quiz": { ".read": true, ".write": true },
    ".read": false,
    ".write": false
  }
}
```
   4. Projektübersicht → Web-App hinzufügen (`</>`), Hosting **nicht** anhaken. Die angezeigte `firebaseConfig` in `index.html` bei `FIREBASE_CONFIG` einfügen. Das npm-/Script-Snippet wird nicht gebraucht.
2. **GitHub Pages**
   1. Repo mit `index.html` im Root.
   2. *Settings → Pages → Deploy from a branch → `main` / `(root)`*.
   3. Nach 1–2 min ist die Seite unter `https://<user>.github.io/<repo>/` erreichbar.
   4. Hinweis: Pages aus einem **privaten** Repo braucht GitHub Pro. Ein öffentliches Repo ist unkritisch, die Fragen liegen in Firebase, nicht im Repo (sofern man sie nicht exportiert und eincheckt).
3. **Prüfen:** Seite öffnen → oben rechts muss grün **„live“** stehen (statt „Lokaler Testmodus“ / „verbinde…“).
---
 
## Spielablauf & Bedienung
 
### Konzept
- Nur der GM klickt. Teams sagen ihr Feld an („Sport für 30“), der GM tippt es an.
- Handys sind reine Anzeige: aktuelle Frage, Antwort nach Auflösung, Board, Punktestand, Reihenfolge.
- **Rotation:** Teams sind reihum dran. Jedes Team hat eine **aktive Person**; nach jedem eigenen Zug rückt im Team die nächste Person nach (Team 1·Anna → Team 2·Emil → … → Team 1·Ben …). Die aktive Person jedes Teams wird unter der Frage als „Aktive Spieler“ angezeigt – damit ist bei 1-gegen-1-Minispielen/Duellen klar, wer gegeneinander spielt.
### GM-Ansicht (`#gm`)
- **Spiel-Tab**
  - Feld antippen → Frage erscheint live auf allen Handys (Antwort sieht nur der GM, gestrichelt).
  - Pro Team **✓ richtig** (Punkte + Feld in Teamfarbe) oder **✗ falsch** (für Klau-Runden; optional Minuspunkte).
  - **Antwort aufdecken**, **Keiner** (Feld verfällt), **Abbrechen** (Frage schließen, Feld bleibt frei).
  - Nach Wertung: **Weiter → nächste Runde** (nächstes Team + nächste Person).
  - Duelle: einfach beim Gewinner-Team „richtig“ drücken.
  - Rangliste: **↻ Person** (Person im Team überspringen), **±10** (Punkte korrigieren), **ist dran** (Zug manuell setzen).
  - Bereits vergebenes Feld antippen → kann wieder freigegeben werden (Punkte werden abgezogen).
  - **↶ Rückgängig** (lokaler Verlauf des GM-Browsers, bis 60 Schritte).
- **Setup-Tab** (Änderungen erst mit **Speichern** live)
  - Titel, Punktestufen (z. B. `10, 20, 30, 40, 50`), Minuspunkte an/aus.
  - Teams: Name, Farbe, Mitglieder (Komma oder Zeilenumbruch, in Spielreihenfolge).
  - Kategorien: Name + Frage/Antwort je Punktestufe; Kategorien hinzufügen/löschen.
  - **Export/Import (JSON)** als Backup. **Spielstand zurücksetzen** löscht nur den Spielverlauf, nicht Fragen/Teams.
### Am Abend
- QR-Code zur Zuschauer-URL auf die Tische.
- Vorher Empfang/WLAN vor Ort prüfen; Firebase verbindet sich nach Abbrüchen selbst neu.
- Vor dem Spiel einmal exportieren (Backup) und „Spielstand zurücksetzen“.
---
 
## Datenmodell (Firebase-Pfad `/quiz`)
 
```jsonc
{
  "settings": {
    "title": "Der Große Preis",
    "points": [10, 20, 30, 40, 50],          // Zeilen des Boards
    "allowMinus": false,                      // Minuspunkte bei "falsch"
    "teams": [{ "id": "t1", "name": "Team 1", "color": "#d1495b", "members": ["Anna", "Ben"] }],
    "categories": [{ "name": "Sport", "questions": [{ "q": "Frage", "a": "Antwort" }] }]  // questions[i] gehört zu points[i]
  },
  "game": {
    "taken":  { "<catIndex>_<rowIndex>": { "team": "t1", "pts": 30 } },  // team "" = keiner wusste es
    "adjust": { "t1": -10 },                  // manuelle Korrekturen/Minuspunkte
    "ptr":    { "t1": 2 },                    // Rotations-Zeiger: aktive Person = members[ptr % members.length]
    "turn":   0,                              // Index des Teams am Zug in settings.teams
    "log":    [{ "team": "t1", "member": "Anna", "c": 0, "r": 2, "winner": "t1" }],  // gespielte Runden
    "current": { "c": 0, "r": 2, "reveal": false, "wrong": ["t2"], "winner": "t1" } // offene Frage oder null
  }
}
```
 
- **Punktestand wird berechnet**, nicht gespeichert: Summe `taken[*].pts` pro Team + `adjust`.
- Firebase entfernt leere Arrays/`null` und kann Arrays als Objekte liefern → `normalize()` repariert das beim Lesen. Neue Felder dort mit Default ergänzen.
- Der GM schreibt bei jeder Aktion den **kompletten** State (`set()` auf `/quiz`). Bei einem GM-Gerät gibt es keine Konflikte.
## Code-Aufbau (`index.html`, `<script type="module">`)
 
| Abschnitt | Inhalt |
|---|---|
| Konfiguration | `FIREBASE_CONFIG`, `GM_PASSWORD`, `DB_PATH` |
| `defaultState()` / `normalize()` | Standard-Daten (6 Kategorien, 4 Teams) und Reparatur eingehender Daten |
| `scores()`, `turnTeam()`, `activeMember()`, `upcoming()` | Abgeleitete Werte (Punkte, Zug, Rotation, Vorschau-Reihenfolge) |
| `initStore()` / `onRemote()` | Firebase- oder localStorage-Anbindung, Live-Updates |
| `commit()` / `undo()` | Jede Änderung: Verlauf sichern → State mutieren → speichern → rendern |
| `A = {...}` | Alle GM-Aktionen (open, reveal, award, wrong, nobody, close, cancel, setTurn, skip, adj) |
| `*HTML()`-Funktionen + `render()` | Rendering per Template-Strings (`headerHTML`, `boardHTML`, `currentHTML`, `leaderHTML`, `scheduleHTML`, `setupHTML`) |
| Events | Event-Delegation über `data-act`-Attribute; Setup-Eingaben schreiben in `draft` |
| Schneefall | Kleines Canvas-Script, deaktiviert bei `prefers-reduced-motion` |
 
Design: Winter-/Weihnachtsthema (Nachthimmel, Schneefall, Lichterkette, Felder als Geschenke – weiß, erspielt in Teamfarbe), Akzent hansgrohe-Türkis `#26a792`, Gold `#f2c14e`. Fonts: Jost + Berkshire Swash (Google Fonts). Farben als CSS-Variablen in `:root`.
 
## Lokal entwickeln
 
- `index.html` einfach im Browser öffnen, oder in VS Code mit der Extension „Live Server“.
- Ohne die echte DB zu berühren: `databaseURL` temporär auf `""` setzen → lokaler Testmodus. Zwei Tabs öffnen: `index.html` und `index.html#gm`.
- Änderungen committen + pushen → GitHub Pages aktualisiert sich nach ~1 min (ggf. Handy-Cache neu laden).
## Historie
 
- 2026-11: Erstellt mit Claude (Konzept, App, Design). Vorbild war eine frühere Freundesrunde mit Beamer + Excel-Tabelle.
 