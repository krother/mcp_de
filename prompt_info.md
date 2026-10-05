
# Den Prompt füttern

Das LLM sieht deinen Python-Code nie. Es sieht nur, was FastMCP in den Prompt schreibt. In diesem Kapitel bringst du an einigen Stellen nützliche Anweisungen unter:

- serverweite Anweisungen
- Beschreibung jedes Tools
- Type Hints für jeden Parameter
- Beschreibung jedes Parameters

Das LLM entscheidet **ausschließlich anhand dieser Texte**, ob und wie es dein Tool aufruft.

----

## Übung 1: Beschreibe deine Tools

Starte den Server und liste die Tools auf:

```bash
uv run fastmcp list http://localhost:8000/mcp
```

Notiere, was fehlt.

## Übung 2: Serverweite Anweisungen hinzufügen

```python
mcp = FastMCP(
    "Fibonacci",
    instructions="Use this server for anything related to Fibonacci numbers.",
)
```

Liste die Tools erneut auf.

## Übung 3: Einen Docstring hinzufügen

```python
@mcp.tool
def fibonacci(n: int) -> int:
    """Calculates the n-th number of the Fibonacci series."""
```

## Übung 4: Parameter mit `Annotated` beschreiben

```python
from typing import Annotated


@mcp.tool
def fibonacci(
    n: Annotated[int, "position in the Fibonacci series, starting at 0"],
) -> int:
    ...
```


## Übung 5: Parameter im Docstring beschreiben

Füge ein zweites Tool hinzu, das mehrere Zahlen zurückgibt:

```python
@mcp.tool
def fibonacci_series(start: int, stop: int) -> list[int]:
    """Returns a slice of the Fibonacci series.

    Args:
        start: position of the first number (inclusive)
        stop: position of the last number (exclusive)
    """
    return [fibonacci(i) for i in range(start, stop)]
```

## Übung 6: Das vollständige JSON-Schema ansehen

Ergänze in `client.py`:

```python
for t in tools:
    pprint(t.inputSchema)
```

Finde deine Beschreibungen in der Ausgabe.

Lösung: [code/fibonacci_mcp.py](code/fibonacci_mcp.py)
