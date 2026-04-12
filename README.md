# miranum-app-registry

Öffentlicher App-Katalog für die Miranum-Plattform. Das Miranum Portal liest
diesen Katalog und zeigt jede App als Card an, die pro Organisation aktiviert
und auf Fly.io deployt werden kann.

## Struktur

```
miranum-app-registry/
├── registry.json            ← leichtgewichtiges Manifest, listet alle Apps
└── apps/
    └── <app-key>/
        ├── app.json         ← vollständige Deploy-Definition
        └── README.md        ← ausführliche Beschreibung (Markdown)
```

### `registry.json`

Das Manifest enthält nur die Metadaten, die fürs Listing gebraucht werden —
also Name, kurze Beschreibung, Kategorie, Tags. Das Portal lädt diese Datei
beim Öffnen des Apps-Tabs.

```json
{
  "name": "miranum-app-registry",
  "version": "1.0.0",
  "apps": [
    {
      "key": "hello-world",
      "label": "Hello World",
      "description": "Demo-App zum Verifizieren des Deploy- und Gateway-Flows",
      "category": "demo",
      "tags": ["demo"]
    }
  ]
}
```

Felder pro App:

| Feld | Pflicht | Beschreibung |
|---|---|---|
| `key` | ja | URL-safe Identifier. Muss zum Ordnernamen unter `apps/` passen. |
| `label` | ja | Anzeigename in der Portal-UI. |
| `description` | ja | Einzeilige Kurzbeschreibung für die Card. |
| `category` | nein | Kategorie für optionale Filterung (`demo`, `buchhaltung`, ...). |
| `tags` | nein | Stichwörter als Badges. |
| `icon` | nein | Optional: Name eines Lucide-Icons. |

### `apps/<key>/app.json`

Die volle Deploy-Definition. Wird geladen, sobald ein Nutzer die App-Details
öffnet oder das Portal die App deployen bzw. den Status abfragen will.

```json
{
  "image": "registry.fly.io/miranum-hello-world:latest",
  "internalPort": 3000,
  "healthCheckPath": "/health",
  "repoUrl": "https://github.com/dominikhorn93/miranum-hello-world",
  "secrets": []
}
```

Felder:

| Feld | Pflicht | Beschreibung |
|---|---|---|
| `image` | ja | Docker-Image-URI, typischerweise `registry.fly.io/<name>:latest`. |
| `internalPort` | nein | Port im Container (Default `3000`). |
| `healthCheckPath` | nein | Optionaler HTTP-Health-Check-Pfad. |
| `repoUrl` | nein | Link zum Quellcode-Repo — wird im Details-Dialog als Link angezeigt. |
| `secrets` | nein | Liste der Secrets, die der Admin pro Org konfigurieren muss: `{ key, label, required? }[]`. |

### `apps/<key>/README.md`

Ausführliche Beschreibung als Markdown. Wird beim Öffnen des Details-Dialogs
im Portal angezeigt. Hier können Features, Setup-Hinweise, Screenshots
oder alles andere stehen, das dem Nutzer hilft.

## Neue App hinzufügen

1. Neuen Unterordner `apps/<neuer-key>/` anlegen.
2. `app.json` mit Image + Deploy-Konfiguration schreiben.
3. `README.md` mit ausführlicher Beschreibung schreiben.
4. Eintrag in `registry.json` unter `apps` hinzufügen.
5. Commit + Push.

Das Portal aktualisiert seinen Cache alle 5 Minuten — danach taucht die App
automatisch im Apps-Tab auf, ohne dass das Portal neu deployt werden muss.

## Konfiguration im Portal

Das Portal liest den Katalog standardmäßig von diesem Repo. Über Env-Vars
lässt sich das umstellen:

- `APP_REGISTRY_REPO` — GitHub-Repo als `owner/name` (Default: `dominikhorn93/miranum-app-registry`)
- `APP_REGISTRY_BRANCH` — Branch (Default: `main`)
