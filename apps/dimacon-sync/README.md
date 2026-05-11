# Dimacon · Clockin Sync

Synchronisiert die Tagesplanung aus **Dimacon** in **Clockin** — Projekte
upserten, Mitarbeiter zuweisen, nicht eingeplante Projekte archivieren. Optional
mit Anreicherung aus **Lexoffice** für Kunden-Stammdaten.

## Was macht diese App?

Die App lädt zu einem Stichtag die geplanten Dimacon-Termine, reichert die
Kunden- und Projektdaten gegen Lexoffice an und überträgt das Ergebnis nach
Clockin:

- **Projekte** werden per "Search-before-create" upgesertet (kein
  Duplikat-Spam).
- **Mitarbeiter** werden über Nachname → Vorname → E-Mail gematched und am
  Projekt attached/detached.
- **Nicht mehr eingeplante** Projekte werden archiviert.

Die App bietet drei Trigger für denselben Sync-Lauf:

- UI unter `/sync` (mit Datum-Picker und Dry-Run-Modus)
- HTTP-Webhook auf `POST /api/sync/run` (optional via Shared-Secret geschützt)
- Eingebauter Cron-Scheduler (über die UI unter `/settings` konfigurierbar —
  Zeitplan, Zeitzone, Enabled-Flag)

## Persistenz

Der Sync-Scheduler hält seinen Zeitplan persistent in einer JSON-Datei unter
`/data/settings.json`. Damit die Konfiguration Redeploys überlebt, mountet die
App ein Volume an `/data`. Beim ersten Boot wird die Datei aus den optionalen
Seed-Env-Vars (`SYNC_CRON`, `SYNC_TZ`) angelegt; danach gewinnt die UI.

## Konfiguration

| Secret                    | Pflicht | Beschreibung                                        |
| ------------------------- | ------- | --------------------------------------------------- |
| `CLOCKIN_API_TOKEN`       | ja      | API-Token für die Clockin-Cloud                     |
| `DIMACON_BASE_URL`        | ja      | Basis-URL der Dimacon-Instanz                       |
| `DIMACON_TENANT`          | ja      | Tenant in Dimacon                                   |
| `DIMACON_API_TOKEN`       | ja      | API-Token für Dimacon                               |
| `LEXWARE_OFFICE_API_KEY`  | ja      | API-Key für Lexoffice                               |
| `CLOCKIN_BASE_URL`        | nein    | Override für Clockin (sonst Default)                |
| `LEXWARE_OFFICE_BASE_URL` | nein    | Override für Lexoffice (sonst Default)              |
| `SYNC_WEBHOOK_SECRET`     | nein    | Shared-Secret für `POST /api/sync/run` (Webhook)    |

## Quellen

- Repo: [Miragon/miranum-dimacon-sync](https://github.com/Miragon/miranum-dimacon-sync)
