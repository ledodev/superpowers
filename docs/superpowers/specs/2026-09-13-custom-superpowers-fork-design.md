# Eigener superpowers-Fork für opencode – Design

Datum: 2026-09-13

## Ziel

`ledo` möchte eine eigene, anpassbare Version der `superpowers`-Skill-
Sammlung in opencode nutzen können, statt der unveränderlichen
Upstream-Plugin-Installation. Dieses Projekt liefert die Infrastruktur
dafür: Fork, Einbindung in opencode, und einen wiederholbaren Workflow
für zukünftige Anpassungen. Konkrete Änderungen an einzelnen Skills
sind explizit **nicht** Teil dieses Projekts und folgen später in
eigenen, kleineren Änderungen.

## Ausgangslage

- `superpowers` war bisher als Plugin über
  `"superpowers@git+https://github.com/obra/superpowers.git"` in
  `~/.config/opencode/opencode.jsonc` eingebunden.
- opencode installiert Git-Plugins nach
  `~/.cache/opencode/packages/<spec>/node_modules/<name>/` und legt dort
  eine eigene `package-lock.json` an, die den Checkout auf einen
  bestimmten Commit **pinnt**. Ein erneuter Start von opencode
  installiert nicht automatisch neu – der Cache-Ordner muss manuell
  gelöscht werden, damit eine neue Version gezogen wird.
- `obra/superpowers` ist ein aktives Repo mit vielen offenen Branches;
  es wird regelmäßig weiterentwickelt.

## Entscheidung

Es wird ein echter GitHub-Fork angelegt (nicht nur eine lokale Kopie),
damit Upstream-Änderungen sauber über Git-Remotes nachverfolgt und bei
Bedarf gezielt gemerged/gecherry-pickt werden können. Die Einbindung in
opencode erfolgt weiterhin über die Git-Spec-Syntax (wie beim
Original), nicht über einen lokalen Dateipfad – das entspricht am
ehesten dem bisherigen Verhalten und Update-Zyklen sind für dieses
Projekt akzeptabel per manuellem Cache-Reset.

## Komponenten

### A. Repository-Setup (erledigt)

- Fork: `ledodev/superpowers` (GitHub-Fork von `obra/superpowers`)
- Lokaler Klon: `/home/ledo/opencode/projekte/superpowers`
- Remotes:
  - `origin` → `git@github.com:ledodev/superpowers.git`
  - `upstream` → `git@github.com:obra/superpowers.git`
- Arbeitsbranch: `main` (kein separater Branch nötig für ein
  persönliches Fork-Repo)

### B. opencode-Konfiguration

- In `~/.config/opencode/opencode.jsonc` wird der Plugin-Eintrag
  geändert:
  - Vorher: `"superpowers@git+https://github.com/obra/superpowers.git"`
  - Nachher: `"superpowers@git+https://github.com/ledodev/superpowers.git"`
- Dies erzeugt beim nächsten Start einen neuen, eigenständigen
  Cache-Ordner unter
  `~/.cache/opencode/packages/superpowers@git+https:/github.com/ledodev/superpowers.git/`.
  Der alte obra-Cache-Ordner kollidiert nicht damit und kann optional
  manuell gelöscht werden.

### C. Workflow für zukünftige Anpassungen

**Eigene Änderungen vornehmen:**
1. Im lokalen Klon (`/home/ledo/opencode/projekte/superpowers`) Skills
   o.ä. anpassen.
2. Committen und nach `origin main` pushen.
3. Den zugehörigen Cache-Ordner unter
   `~/.cache/opencode/packages/superpowers@git+https:/github.com/ledodev/superpowers.git/`
   löschen, damit opencode beim nächsten Start neu installiert.
4. opencode neu starten – Änderungen sind aktiv.

**Upstream-Updates übernehmen (bei Bedarf, nicht automatisch):**
1. `git fetch upstream`
2. `git merge upstream/main` (oder gezielt `git cherry-pick <commit>`
   für einzelne Änderungen)
3. Konflikte lösen, committen.
4. Wie oben: nach `origin main` pushen, Cache-Ordner löschen, opencode
   neu starten.

### D. Verifikation

1. Nach Umstellung der Konfiguration: opencode neu starten und
   bestätigen, dass die superpowers-Skills weiterhin geladen werden
   (z. B. dass `using-superpowers` weiterhin wie gewohnt greift).
2. End-to-End-Test des Anpassungs-Workflows: eine harmlose Testzeile in
   einer `SKILL.md` im Fork ändern, committen, pushen, Cache-Ordner
   löschen, opencode neu starten, und im neuen Cache-Pfad
   (`~/.cache/opencode/packages/superpowers@git+.../node_modules/superpowers/skills/...`)
   verifizieren, dass die Änderung angekommen ist. Testzeile danach
   wieder entfernen (Commit + Push + Cache-Reset).

## Out of Scope

- Inhaltliche Anpassungen an einzelnen Skills (Themen für spätere,
  eigene Aufgaben).
- Automatisierungs-Skripte für den Push/Cache-Zyklus (bewusst manuell
  gehalten, da vom Nutzer als akzeptabel bestätigt).
- Synchronisation mit Upstream als wiederkehrender/automatisierter
  Prozess (nur bei Bedarf, manuell).
