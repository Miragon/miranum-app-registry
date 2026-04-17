# Office 365 MCP

Ein Model Context Protocol (MCP) Server, der Microsoft 365 und Microsoft
Office Services über die Microsoft Graph API für KI-Assistenten zugänglich
macht.

## Was macht diese App?

Die App stellt über 200 Tools bereit, die 1-zu-1 auf Microsoft-Graph-Endpoints
gemappt sind — von Outlook-Mails und Kalender über Teams-Chats und
SharePoint-Dateien bis hin zu OneDrive und To-Do-Listen. Authentifizierung
läuft über die Microsoft Authentication Library (MSAL).

Damit können angebundene LLMs (z. B. Claude) im Namen des Nutzers auf
Office-365-Daten zugreifen, ohne dass für jeden Service ein eigener Connector
gebaut werden muss.

## Konfiguration

Vor dem Deployment muss eine App-Registrierung im Microsoft Entra ID
(Azure AD) angelegt werden. Aus dieser Registrierung werden zwei Werte
benötigt:

| Secret | Beschreibung |
|---|---|
| `MS365_MCP_CLIENT_ID` | Application (client) ID der Entra-App-Registrierung. |
| `MS365_MCP_TENANT_ID` | Directory (tenant) ID der Microsoft-365-Organisation. |

Beide Werte finden sich im Azure Portal unter *Entra ID → App registrations →
&lt;eure App&gt; → Overview*.

## Quellen

- Fork: [Miragon/miranum-365-mcp-server](https://github.com/Miragon/miranum-365-mcp-server)
- Upstream: [softeria/ms-365-mcp-server](https://github.com/softeria/ms-365-mcp-server)
