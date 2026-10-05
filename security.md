
# Datenschutz und Sicherheit

Ein MCP Server gibt einem LLM . Damit kann es auch Schaden anrichten.

## Lethal Trifecta

Gefährlich wird es, wenn ein System drei Eigenschaften gleichzeitig hat

| Eigenschaft | Beispiel im Incident-Server |
|-------------|-----------------------------|
| **Zugriff auf private Daten** | Incidents mit Namen, Orten, Passwort-Hinweisen |
| **Kontakt mit nicht vertrauenswürdigen Inhalten** | die Beschreibung eines Incidents stammt von irgendwem |
| **ein Weg nach draußen** | ein Tool, das E-Mails schickt oder URLs abruft |

Ein Angreifer schreibt Anweisungen in einen Incident – das LLM liest sie und schickt die Daten per E-Mail nach draußen.
**Entferne mindestens eine der drei Eigenschaften.**

### Quellen:

- [Linux Magazin](https://www.cusy.io/de/blog/securing-llm-agents.html)
- [Simon Willison](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)


## Angriffe

| Angriff | Beschreibung |
|---------|-------------|
| **direkte Prompt Injection** | die Benutzerin selbst schreibt *"Ignoriere alle Regeln und …"* |
| **indirekte Prompt Injection** | Anweisungen stecken in Daten, die das LLM liest: Incidents, Webseiten, E-Mails |
| **Tool Poisoning** | ein fremder MCP Server versteckt Anweisungen in seinen Tool-Beschreibungen |
| **Excessive Agency** | ein Tool kann mehr als nötig, z.B. `run_sql(query: str)` statt `list_incidents()` |
| **Confused Deputy** | das LLM handelt mit den Rechten des Servers, nicht mit denen der Benutzerin |

Mehr dazu: [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/)

## Datenschutz

- alles, was ein Tool zurückgibt, landet im Kontext – und damit beim Anbieter des LLM
- personenbezogene Daten nur zurückgeben, wenn sie gebraucht werden
- lokale Modelle (Ollama) für sensible Daten in Betracht ziehen
- Logs enthalten Tool-Argumente – auch sie sind personenbezogene Daten

## Least Privilege

- **Read-only-Tools**, wo immer möglich
- **eingeschränkte Zugangsdaten**: der Server bekommt nur die Datenbankrechte, die er braucht
- **Tool-Allowlists**: nur die Tools anbieten, die ein Client braucht (siehe [Mehrere MCP Server](multiple_servers.md))
- **Freigaben** für alles, was nicht rückgängig zu machen ist (siehe [Human-in-the-Loop](langgraph_workflows.md))

----

## Übung: Audit-Log

Jeder Tool-Aufruf soll nachvollziehbar sein: wer, wann, was, mit welchem Ergebnis.
Eine **Middleware** sieht jeden Aufruf, bevor er beim Tool ankommt:

```python
import json
import logging

from fastmcp.server.middleware import Middleware

audit = logging.getLogger("audit")
audit.addHandler(logging.FileHandler("audit.log"))
audit.setLevel(logging.INFO)


class AuditMiddleware(Middleware):

    async def on_call_tool(self, context, call_next):
        entry = {
            "time": context.timestamp.isoformat(timespec="seconds"),
            "tool": context.message.name,
            "arguments": context.message.arguments,
        }
        result = await call_next(context)
        audit.info(json.dumps(entry))
        return result


mcp.add_middleware(AuditMiddleware())
```

1. Füge die Middleware zu deinem Incident-Server hinzu.
2. Lasse ein LLM einige Incidents anlegen und schließen. Sieh dir `audit.log` an.
3. Protokolliere auch fehlgeschlagene Aufrufe (`try` / `except` / `finally`).
4. Welche Felder im Log sind personenbezogene Daten? Wie lange darfst du sie aufbewahren?

Lösung: [code/audit_mcp.py](code/audit_mcp.py)
