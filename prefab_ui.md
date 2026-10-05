
# UI mit Prefab

Ein MCP Server kann nicht nur Text, sondern auch **Benutzeroberflächen** zurückgeben.
Clients wie Claude zeigen sie direkt im Chat an.
FastMCP verwendet dafür die Komponentenbibliothek **Prefab**.

```bash
uv add "fastmcp[apps]"
```

Es gibt zwei Varianten:

| | Decorator | Funktionsweise | Beispiel |
|--|-----------|---------------|----------|
| **App-Tool** | `@mcp.tool(app=True)` | der Server baut die Ansicht einmal und gibt sie zurück | Dashboard, Karte, Tabelle |
| **FastMCPApp** | `@app.ui()` + `@app.tool()` | die Ansicht ruft bei Klicks Tools auf dem Server auf | Formulare, Bestätigungen |

----

## Übung 1: Incidents laden

Der folgende Code sollte in `incident_mcp.py` einige beispielhafte incidents laden.

Füge die JSON-Datei [code/incidents.json](code/incidents.json) im gleichen Verzeichnis hinzu.

```
import json

incidents: dict[int, dict] = {
    i["id"]: i
    for i in json.loads((Path(__file__).parent / "incidents.json").read_text())
}
```

## Übung 2: Ein Dashboard

Stelle die Incidents als Tabelle dar:

```python
from prefab_ui.app import PrefabApp
from prefab_ui.components import Column, DataTable, DataTableColumn, Heading, Metric, Row


@mcp.tool(app=True)
def incident_dashboard() -> PrefabApp:
    """Shows all incidents as a table."""
    open_incidents = [i for i in incidents.values() if i["status"] == "open"]
    with Column(gap=4, css_class="p-6") as view:
        Heading("Incidents")
        with Row(gap=4):
            Metric(label="open", value=len(open_incidents))
            Metric(label="total", value=len(incidents))
        DataTable(
            columns=[
                DataTableColumn(key="id", header="#"),
                DataTableColumn(key="title", header="Title", sortable=True),
                DataTableColumn(key="location", header="Location"),
                DataTableColumn(key="status", header="Status"),
            ],
            rows=list(incidents.values()),
            search=True,
        )
    return PrefabApp(view=view)
```

## Übung 3: Das Beispiel ausführen

Starte die Vorschau:

```bash
uv run fastmcp dev apps incident_ui.py
```

Wähle `incident_dashboard` aus. Sortiere und durchsuche die Tabelle.

## Übung 4: Den Katalog durchstöbern

Öffne die [Prefab-Komponenten](https://prefab.prefect.io/docs/welcome) und suche dir zwei Komponenten aus,
die das Dashboard verbessern würden, z.B. ein `Badge` für die Severity oder ein Diagramm.
Baue sie ein.

----

## Übung 5: Ein Formular, das den Server aufruft

Eine `FastMCPApp` trennt die Benutzerschnittstelle (`@app.ui`) von den Tools, die die Ansicht aufruft (`@app.tool`).
Eingabefelder mit `name="..."` speichern ihren Wert im Zustand; `{{ title }}` liest ihn wieder aus.

Füge den Code zum Programm hinzu. Die alte Funktion `create_incident()` wird nicht mehr benötigt. Du kannst sie löschen.

```python
from fastmcp import FastMCP, FastMCPApp
from prefab_ui.actions import SetState, ShowToast
from prefab_ui.actions.mcp import CallTool
from prefab_ui.rx import RESULT
from prefab_ui.components import (
    Badge, Button, Card, CardContent, CardFooter, CardHeader, CardTitle,
    Column, DataTable, DataTableColumn, Heading, If, Input, Metric, Row,
    Select, SelectOption, Text, Textarea,
)


app = FastMCPApp("Incident App")


@app.tool()
def create_incident(title: str, location: str, severity: str) -> dict:
    """Records a new incident."""
    incident_id = len(incidents) + 1
    incidents[incident_id] = {
        "id": incident_id,
        "title": title,
        "location": location,
        "severity": int(severity),
        "status": "open",
    }
    return incidents[incident_id]


@app.ui()
def incident_form() -> PrefabApp:
    """Opens a form for reporting an incident."""
    with Column(gap=4, css_class="p-6") as view:
        Heading("Report an incident")
        Input(name="title", placeholder="What happened?", required=True)
        Input(name="location", placeholder="e.g. B2-3", required=True)
        with Select(name="severity", placeholder="Severity"):
            for level in range(1, 6):
                SelectOption(value=str(level), label=str(level))
        Button(
            "Submit",
            on_click=CallTool(
                "create_incident",
                arguments={
                    "title": "{{ title }}",
                    "location": "{{ location }}",
                    "severity": "{{ severity }}",
                },
                on_success=[
                    SetState("created", RESULT),
                    ShowToast("Incident recorded", variant="success"),
                ],
                on_error=ShowToast("{{ $error }}", variant="error"),
            ),
        )
        with If("created"):
            Text("Recorded incident #{{ created.id }}")
    return PrefabApp(view=view, state={"created": None})


mcp.add_provider(app)
```

Starte den Server neu.

Wähle `incident_form` in der Vorschau aus, lege einen Incident an und prüfe ihn anschließend im Dashboard.

**Tipp:** `Form.from_model(Incident, on_submit=CallTool("create_incident"))` erzeugt ein Formular direkt aus einem Pydantic-Modell.

----

## Übung 6: Human-in-the-Loop

Bevor ein Incident geschlossen wird, soll ein Mensch zustimmen.

FastMCP bringt dafür mit `Approval` einen fertigen Bestätigungsdialog mit.
Er stellt das Tool `request_approval` bereit: Das LLM übergibt eine Zusammenfassung, der Mensch klickt auf **Approve** oder **Reject**,
und die Entscheidung landet als Nachricht im Chat.

Füge den folgenden Code zum Server hinzu:

```python
from fastmcp.apps.approval import Approval

mcp.add_provider(Approval(title="Close incident?", approve_text="Close", approve_variant="destructive"))
```

Ergänze den Docstring von `close_incident` so, dass das LLM vorher `request_approval` aufruft und erst nach **Close** weitermacht:

```
    Before calling this tool, call request_approval with a summary of the
    incident and the resolution. Only call close_incident after the user
    has selected Approve.
```

Probiere es in Claude aus.

Was passiert, wenn du auf **Reject** klickst?

----

## Übung 7: Elicitation

In Übung 6 entscheidet am Ende doch das LLM: Die Zustimmung landet nur als Nachricht im Chat,
und das LLM könnte `close_incident` auch ohne sie aufrufen.

Mit **Elicitation** fragt der Server selbst nach. `close_incident` gibt beim ersten Aufruf statt eines Ergebnisses
ein `InputRequiredResult` zurück. Der Client zeigt dem Menschen die Frage an und ruft das Tool mit der Antwort erneut auf.
Das LLM kann diesen Schritt nicht überspringen.

Ersetze `close_incident` durch:

```python
from fastmcp import Context
from mcp_types import ElicitRequest, ElicitRequestFormParams, InputRequiredResult


@mcp.tool()
def close_incident(
    incident_id: int, resolution: str, ctx: Context
) -> dict | InputRequiredResult:
    """Closes an incident."""
    incident = incidents[incident_id]
    answers = ctx.input_responses
    if answers is None:
        # first call: ask the user, the client calls the tool again with the answer
        question = ElicitRequest(params=ElicitRequestFormParams(
            message=f"Close incident #{incident_id} '{incident['title']}'?\nResolution: {resolution}",
            requested_schema={
                "type": "object",
                "properties": {"close": {"type": "boolean", "title": "Close incident"}},
                "required": ["close"],
            },
        ))
        return InputRequiredResult(input_requests={"confirm": question})

    answer = answers["confirm"]
    if answer.action != "accept" or not answer.content["close"]:
        return {"status": "cancelled", "incident": incident}
    incident["status"] = "closed"
    incident["resolution"] = resolution
    return incident
```

Der Hinweis auf `request_approval` im Docstring wird nicht mehr gebraucht.

Probiere es in Claude aus: *"Schließe Incident 2, das VPN läuft wieder."*

* Was passiert, wenn du ablehnst oder den Dialog abbrichst?
* Vergleiche mit Übung 6: Wo wird die Entscheidung jeweils durchgesetzt?

**Hinweis:** In älteren Versionen des MCP-Protokolls (bis 2025-11-25) schrieb man stattdessen `await ctx.elicit(...)`.
Seit Protokollversion 2026-07-28 funktioniert das nicht mehr.
