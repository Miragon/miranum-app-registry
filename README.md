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
  "env": {
    "NODE_ENV": "production"
  },
  "volumes": [{ "name": "data", "mountPath": "/data", "sizeGb": 1 }],
  "resources": { "memoryMb": 512, "cpus": 1, "cpuKind": "shared" },
  "secrets": []
}
```

Felder:

| Feld              | Pflicht | Beschreibung                                                                                  |
| ----------------- | ------- | --------------------------------------------------------------------------------------------- |
| `image`           | ja      | Docker-Image-URI, typischerweise `registry.fly.io/<name>:latest`.                             |
| `internalPort`    | nein    | Port im Container (Default `3000`).                                                           |
| `healthCheckPath` | nein    | Optionaler HTTP-Health-Check-Pfad.                                                            |
| `repoUrl`         | nein    | Link zum Quellcode-Repo — wird im Details-Dialog als Link angezeigt.                          |
| `env`             | nein    | Nicht-sensible Default-Env-Vars, die das Portal beim Deploy setzt: `Record<string, string>`.  |
| `volumes`         | nein    | Persistente Volumes, die das Portal anlegen + mounten muss (siehe unten).                     |
| `resources`       | nein    | Sizing-Hint für die Machine. Default = Fly-Default (256 MB, shared-1x).                       |
| `secrets`         | nein    | Liste der Secrets, die der Admin pro Org konfigurieren muss: `{ key, label, required? }[]`.   |

#### `env` vs. `secrets`

`secrets` sind sensibel (API-Tokens, Passwörter) und werden pro Org vom Admin
befüllt — das Portal speichert sie als Fly-Secret. `env` ist für **statische
nicht-sensible** Defaults aus dem Registry (z. B. `SETTINGS_PATH=/data/settings.json`)
und wird beim Deploy als normale Env-Var gesetzt.

#### `volumes`

Persistente Storage-Anforderungen der App. Das Portal legt für jeden Eintrag
beim ersten Deploy ein Fly-Volume an und referenziert es im Machine-Config als
Mount. Volumes überleben Redeploys.

```json
"volumes": [
  { "name": "data", "mountPath": "/data", "sizeGb": 1, "region": "fra" }
]
```

| Feld        | Pflicht | Beschreibung                                                                  |
| ----------- | ------- | ----------------------------------------------------------------------------- |
| `name`      | ja      | Stabiler Bezeichner, lowercase + Underscore. Wird zum Fly-Volume-Namen.       |
| `mountPath` | ja      | Pfad im Container, z. B. `/data`.                                             |
| `sizeGb`    | ja      | Größe in GB. Fly Minimum ist 1.                                               |
| `region`    | nein    | Region des Volumes. Default = Region der App. Wichtig: Volumes sind region-pinned. |

#### `resources`

| Feld       | Pflicht | Beschreibung                                                  |
| ---------- | ------- | ------------------------------------------------------------- |
| `memoryMb` | nein    | RAM in MB.                                                    |
| `cpus`     | nein    | Anzahl vCPUs.                                                 |
| `cpuKind`  | nein    | `shared` oder `performance`. Default `shared`.                |

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
