
# MCP als Backend-Worker

Ein MCP Server braucht kein LLM. Jedes Python-Skript, jede Pipeline und jeder Cronjob kann seine Tools aufrufen.

## Lange laufende Aufgaben

Manche Tools brauchen Minuten: Berichte erzeugen, Daten importieren, Modelle trainieren.
Ein normaler Tool-Aufruf würde so lange blockieren.
Mit **Tasks** läuft das Tool im Hintergrund:

| Begriff | Bedeutung |
|---------|-----------|
| **Task** | ein Tool-Aufruf, der im Hintergrund läuft und eine ID bekommt |
| **Progress** | das Tool meldet Zwischenstände mit `ctx.report_progress()` |
| **Cancellation** | der Client bricht einen Task ab |

----

## Übung 1: Einen Task anlegen

```bash
uv add fastmcp-tasks
```

```python
import asyncio

from fastmcp import Context
from fastmcp_tasks import TasksExtension

mcp.add_extension(TasksExtension())


@mcp.tool(task=True)
async def generate_report(ctx: Context) -> dict:
    """Analyzes all incidents and creates a report. Takes a while."""
    total = len(incidents)
    for i, incident in enumerate(incidents.values(), start=1):
        await asyncio.sleep(1)  # pretend to do something expensive
        await ctx.report_progress(i, total, f"analyzed incident #{incident['id']}")
    return {"total": total}
```


Rufe den Task aus einem Skript auf und frage zwischendurch den Status ab:

```python
from fastmcp_tasks import call_tool_task

task = await call_tool_task(client, "generate_report")
status = await task.status()
print(status.status, status.status_message)
report = await task.result()
```

## Übung 2: CSV-Import

Schreibe ein Skript, das alle Zeilen aus [code/incidents.csv](code/incidents.csv) über `create_incident` importiert
und anschließend den Bericht erzeugt. Es wird kein LLM benötigt.

```python
async with client:
    with open("incidents.csv") as f:
        for row in csv.DictReader(f):
            result = await client.call_tool("create_incident", row)
```

Lösung: [code/incident_worker.py](code/incident_worker.py), [code/import_csv.py](code/import_csv.py)

----

### Frage

Wann sind Tasks die richtige Abstraktion, und wann ist REST-API einfacher?
