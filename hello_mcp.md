
# Dein erster MCP Server

## Übung: Fibonacci-MCP

In dieser Übung baust du einen einfachen MCP Server, der Fibonacci-Zahlen berechnen kann.

### Schritt 1: Ein Projekt anlegen

```bash
python -m pip install uv
mkdir mcp
cd mcp
uv init
uv add fastmcp
```

Der letzte Befehl sollte fehlschlagen. Lies die Fehlermeldung, um herauszufinden, warum.
Finde eine passende Lösung.

Überprüfe das Ergebnis mit:

```bash
uv run fastmcp version
```

### Schritt 2: Einen Server schreiben

Erstelle eine Datei `fibonacci_mcp.py`:

```python
from fastmcp import FastMCP

mcp = FastMCP("Fibonacci")


@mcp.tool
def hello(name: str) -> str:
    return f"hello {name}"


if __name__ == "__main__":
    mcp.run(transport="http", host="127.0.0.1", port=8000)
```

### Schritt 3: Ein Tool hinzufügen

Füge deine Funktion `fibonacci()` aus der vorherigen Übung hinzu.
Dekoriere sie mit `@mcp.tool` und ergänze Type Hints.

### Schritt 4: Den Server starten

```bash
uv run fibonacci_mcp.py
```

Öffne [http://localhost:8000/mcp](http://localhost:8000/mcp) im Browser.
Anders als bei FastAPI solltest du eine Fehlermeldung sehen – der Server spricht kein HTML, sondern nur JSON-RPC.

Beende den Server mit `Strg+C`. Starte ihn erneut, diesmal mit dem Terminal-Befehl `fastmcp` und Auto-Reload:

```bash
uv run fastmcp run fibonacci_mcp.py:mcp --transport http --port 8000 --reload
```

### Schritt 5: Den Server aus Python aufrufen

Erstelle `client.py`:

```python
import asyncio
from pprint import pprint

from fastmcp import Client

client = Client("http://localhost:8000/mcp")


async def main():
    async with client:
        tools = await client.list_tools()
        pprint([t.name for t in tools])

        result = await client.call_tool("fibonacci", {"n": 10})
        pprint(result.data)


asyncio.run(main())
```

Führe es in einem zweiten Terminal aus:

```bash
uv run client.py
```

Alternativ kannst du die CLI verwenden:

```bash
uv run fastmcp list http://localhost:8000/mcp
uv run fastmcp call http://localhost:8000/mcp fibonacci n=10
```

### Schritt 6: Eine UI mit Prefab hinzufügen

Installiere das Extra:

```bash
uv add "fastmcp[apps]"
```

Füge ein Tool hinzu, das eine visuelle Karte zurückgibt:

```python
from prefab_ui.app import PrefabApp
from prefab_ui.components import Badge, Column, Heading, Row, Text


@mcp.tool(app=True)
def fibonacci_card(n: int) -> PrefabApp:
    """Shows a Fibonacci number as a visual card."""
    with Column(gap=4, css_class="p-6") as view:
        Heading(f"Fibonacci #{n}")
        with Row(gap=2, align="center"):
            Text("Result")
            Badge(str(fibonacci(n)), variant="success")
    return PrefabApp(view=view)
```

Beende den Server und starte ihn im Dev-Modus:

```bash
uv run fastmcp dev apps fibonacci_mcp.py
```

Ein Browserfenster öffnet sich. Wähle `fibonacci_card` aus und führe es aus.

### Musterlösung:

Lösung: [code/hello_mcp.py](code/hello_mcp.py), [code/fibonacci_mcp.py](code/fibonacci_mcp.py), [code/client.py](code/client.py)
