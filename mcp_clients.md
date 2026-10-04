
# MCP Clients

**Ziel: In diesem Kapitel verbindest du verschiedene LLMs mit deinem MCP Server.**

Wir probieren möglichst viele davon aus:

- MCP Inspector (ein Node-basiertes Client-Programm, kein LLM)
- Qwen3 0.6, ein kleines lokales Ollama-Modell
- Claude
- Copilot (hier nicht dokumentiert)

----

## Transport-Modi

| Transport | Funktionsweise | Einsatzzweck |
|-----------|-----|---------|
| **stdio** | der Host startet den Server als Subprozess und kommuniziert über stdin/stdout | lokal, ein Benutzer |
| **Streamable HTTP** | der Server läuft als Webservice unter `/mcp` und unterstützt Streaming | remote, viele Clients |
| **SSE** | ältere HTTP-Variante | veraltet, ersetzt durch Streamable HTTP |

Wechsle den Transport in deinem Server nach Bedarf:

```python
mcp.run()                                       # stdio (Standard)
mcp.run(transport="http", host="127.0.0.1", port=8000)   # HTTP
```

----

## Übung: MCP Inspector

### Schritt 1: Node.js installieren

Prüfe, ob `npx` verfügbar ist:

```bash
npx --version
```

Falls nicht, installiere Node.js von [nodejs.org](https://nodejs.org/).

### Schritt 2: Den Inspector starten

```bash
npx @modelcontextprotocol/inspector
```

Ein Browserfenster sollte sich öffnen.

### Schritt 3: Mit deinem Server verbinden

Starte deinen Fibonacci-Server über HTTP (siehe [hello_mcp.md](hello_mcp.md)). Im Inspector:

1. Transport Type: **Streamable HTTP**
2. URL: `http://localhost:8000/mcp`
3. klicke auf **Connect**
4. gehe zu **Tools** → **List Tools** → wähle `fibonacci` → **Run Tool**

### Schritt 4: Den Rohdatenverkehr untersuchen

Öffne unten das Panel **History**.
Klicke auf jeden Eintrag und finde:

- den `initialize`-Request und die Capabilities des Servers
- die `tools/list`-Response mit deinen Docstrings
- den `tools/call`-Request und die zugehörige Response

**Tipp:** FastMCP kann den Inspector zusammen mit deinem Server starten:

```bash
uv run fastmcp dev inspector fibonacci_mcp.py
```

----

## Übung: Lokales LLM mit Ollama

### Schritt 1: Ollama installieren

Herunterladen und installieren von [ollama.com/download](https://ollama.com/download).

### Schritt 2: Ein kleines Modell ausführen

```bash
ollama run qwen3:0.6b
```

Probiere ein paar Prompts aus, z.B. *"Was ist die 30. Fibonacci-Zahl?"*. Beenden mit `/bye`.

### Schritt 3: Einen MCP Client für Ollama installieren

```bash
uv add ollmcp
```

### Schritt 4: Das Modell mit deinem Server verbinden

Starte deinen Fibonacci-Server über HTTP. In einem zweiten Terminal:

```bash
uv run ollmcp -m qwen3:0.6b -u http://localhost:8000/mcp
```

Frage erneut: *"Was ist die 30. Fibonacci-Zahl?"*
Beobachte, ob das Modell das Tool aufruft.

Wenn das Modell das Tool ignoriert, probiere ein größeres: `qwen3:1.7b` oder `qwen3:4b`.

----

## Übung: Claude

### Schritt 1: Den Server in Claude Code (CLI) registrieren

Während der Server über HTTP läuft:

```bash
claude mcp add --transport http fibonacci http://localhost:8000/mcp
claude mcp list
```

Alternativ über stdio – Claude startet den Server dann selbst:

```bash
claude mcp add fibonacci-stdio -- uv run --directory /full/path/to/your/project fibonacci_mcp.py
```

Für stdio muss der Server `mcp.run()` ohne `transport="http"` aufrufen.

### Schritt 2: Einen Prompt in der CLI ausführen

```bash
claude
```

Tippe `/mcp`, um die verbundenen Server zu sehen. Frage dann:

> Welche Fibonacci-Zahlen stehen an den Positionen 50 bis 60?

Claude fragt um Erlaubnis, bevor es das Tool aufruft.

### Schritt 3: Denselben Prompt in VS Code ausführen

1. öffne deinen Projektordner in VS Code
2. öffne das Claude-Code-Panel
3. tippe `/mcp` und prüfe, ob `fibonacci` verbunden ist (VS Code verwendet dieselbe Konfiguration wie die CLI)
4. führe denselben Prompt aus

### Schritt 4: Vergleichen

- Haben alle Clients (Ollama, Claude CLI, VS Code) das Tool aufgerufen?
- Welches Tool haben sie gewählt: `fibonacci` oder `fibonacci_series`?
