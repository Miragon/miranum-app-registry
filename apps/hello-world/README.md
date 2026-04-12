# Hello World

Demo-App zum Verifizieren des Deploy- und Gateway-Flows.

## Was macht diese App?

Ein minimaler HTTP-Server, der auf `GET /health` antwortet und damit zeigt,
dass die komplette Deploy-Kette funktioniert:

1. Portal startet ein neues Fly.io Machine aus `registry.fly.io/miranum-app-template:latest`
2. Das Gateway leitet Requests an `https://<org-slug>.gateway/<app-key>/...` an die Machine weiter
3. Die App antwortet auf Port 3000

## Wann nutzen?

- Als erste App beim Einrichten einer neuen Organisation, um zu prüfen dass
  Fly.io-Credentials, Gateway-Routing und Status-Anzeige im Portal alle
  korrekt konfiguriert sind.
- Als Vorlage für neue Apps — der Dockerfile ist minimal und zeigt die
  erwartete Port- und Health-Check-Konfiguration.

## Konfiguration

Keine. Die App hat keine Secrets und kein externes Setup.
