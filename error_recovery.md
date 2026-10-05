
# Fehlerbehandlung

Ein LLM sieht nur, was ein Tool zurückgibt. Meldet ein Tool keinen Fehler, nimmt das LLM an, dass alles geklappt hat –
und erzählt das auch der Benutzerin.

## Wo setzt man an?

| Muster | Problem | Umsetzung |
|--------|---------|-----------|
| **aussagekräftige Fehler** | das LLM rät, was schiefging | `raise ToolError("...")` mit Hinweis, wie es richtig geht |
| **Retry** | ein externer Dienst ist kurz nicht erreichbar | erneut versuchen, im Client oder im Tool |
| **Timeout** | ein Tool hängt und blockiert den Agent Loop | `@mcp.tool(timeout=5.0)` |
| **Idempotenz** | ein Retry legt einen Incident doppelt an | ein zweiter gleicher Aufruf ändert nichts |
| **Fallback** | ein Dienst ist dauerhaft ausgefallen | Ersatzweg, z.B. Ticket per E-Mail statt per API |

```python
from fastmcp.exceptions import ToolError

if incident_id not in incidents:
    raise ToolError(
        f"Incident {incident_id} does not exist. "
        "Call list_incidents to find valid incident numbers."
    )
```

Mit `FastMCP("Incidents", mask_error_details=True)` sieht das LLM nur noch Meldungen aus `ToolError`.
Andere Exceptions (mit Stacktraces, Pfaden, Passwörtern …) werden versteckt.

----

## Übung: Ein Tool, das still versagt

Der Server [code/silent_failure_mcp.py](code/silent_failure_mcp.py) enthält zwei Incidents.

1. Verbinde ihn mit einem LLM.
2. Bitte das LLM: *"Schließe Incident 7, das Problem ist gelöst."*
3. Was antwortet das LLM? Was steht wirklich in `list_incidents`?
4. Finde die Ursache im Code.
5. Ersetze das stille Versagen durch einen `ToolError` mit einer hilfreichen Nachricht.
6. Wiederhole den Prompt. Ruft das LLM jetzt von selbst `list_incidents` auf?
7. Mache `close_incident` idempotent: ein bereits geschlossener Incident wird nicht noch einmal verändert.

Lösung: [code/error_recovery_mcp.py](code/error_recovery_mcp.py)
