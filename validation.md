
# Validierung mit Pydantic

Ein Parameter wie `data: str` sagt dem LLM fast nichts. Mit **Pydantic** beschreibst du genau, welche Werte erlaubt sind.
FastMCP übersetzt das Modell in ein JSON-Schema für das LLM und **prüft jeden Aufruf**, bevor deine Funktion läuft.

----

## Beispiel: Ein Pydantic-Modell als Tool-Parameter

```python
from typing import Literal

from pydantic import BaseModel, Field, field_validator


class Incident(BaseModel):
    """An IT incident reported by a colleague."""

    title: str = Field(min_length=5, max_length=80, description="short summary")
    description: str = Field(description="what happened, in the words of the reporter")
    category: Literal["hardware", "software", "network", "security", "other"]
    severity: int = Field(ge=1, le=5, description="1 = cosmetic, 5 = business stopped")
    affected_users: int = Field(default=1, ge=1, le=10_000)

    @field_validator("location")
    @classmethod
    def check_location(cls, value: str) -> str:
        """Locations have the format <building>-<floor>, e.g. B2-3."""
        building, _, floor = value.partition("-")
        if not building or not floor.isdigit():
            raise ValueError("location must look like 'B2-3' (building-floor)")
        return value.upper()

@mcp.tool
def create_incident(incident: Incident) -> dict:
    ...
```

| Werkzeug | Wirkung |
|----------|---------|
| `Field(description=...)` | Text für das LLM |
| `Field(ge=1, le=5)` | Wertebereich für Zahlen |
| `Field(min_length=5, max_length=80)` | Länge von Strings |
| `Literal["a", "b"]` | erlaubte Werte (wird zu `enum` im JSON-RPC Schema) |
| `@field_validator` | eigene Funktion zur Validierung |

----

## Übung 1: Das Schema ansehen

Starte [code/incident_mcp.py](code/incident_mcp.py).
Sieh dir das Schema der tools von `create_incident` in [code/client.yp](code/client.py) an.
Füge dort folgende Zeilen hinzu:

```python
for t in tools:
    pprint(t.input_schema)
```

Finde den niedrigsten und höchsten erlaubten Wert für `enum`, `minimum` und `maximum` in der JSON-RPC Ausgabe.

## Übung 2: Ungültige Aufrufe

Rufe das Tool aus `client.py` mit ungültigen Werten auf, z.B. `severity=9` oder `location="Küche"`.
Wie sieht die Fehlermeldung aus? Wer bekommt sie zu sehen?

## Übung 3: Incidents erzeugen

Verbinde ein LLM mit dem MCP-Dienst und lasse es drei verschiedene Incidents anlegen. Beobachte, was in `data` ankommt.

Versuche, ungültige Werte einzuschicken. Wie reagiert das LLM auf eine Fehlermeldung des Validators? Korrigiert es sich selbst?

**Tipp:** Kleine Modelle wie `qwen3:0.6b` machen mehr Fehler.
