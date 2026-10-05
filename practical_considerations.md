
# Praktische Überlegungen

Zum Abschluss überlegen betrachten wir noch einmal das Software Engineering: Was braucht ein MCP-Dienst, damit er im Alltag funktioniert?

## Übung 1: Automatische Tests

Ein FastMCP Server lässt sich ohne Netzwerk und ohne LLM testen, indem man den `Client` direkt mit dem Server-Objekt verbindet:

```bash
uv add --dev pytest pytest-asyncio
```

```python
import pytest
import pytest_asyncio
from fastmcp import Client

from incident_mcp import incidents, mcp

pytestmark = pytest.mark.asyncio


@pytest_asyncio.fixture
async def client():
    incidents.clear()
    async with Client(mcp) as c:
        yield c


async def test_create_incident(client):
    result = await client.call_tool(
        "create_incident",
        {"title": "printer on fire", "description": "smoke", "location": "B2-3"},
    )
    assert result.data["status"] == "open"
```

Führe den Test aus.

```bash
uv run pytest
```

----

## Diskussion

1. Woran erkenne ich, dass meine LLM-Anwendung **funktioniert**?
2. Woran erkenne ich, dass sie **das Richtige tut**?
3. Wie wird sich mein Dienst **in Zukunft verändern** – neue Modelle, neue Clients, neue Tools?
