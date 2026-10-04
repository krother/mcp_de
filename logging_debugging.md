# Logging und Debugging

Beim Transport **stdio** **ist** `stdout` der JSON-RPC-Kanal.
Alles andere, was nach `stdout` geschrieben wird, beschädigt die Nachrichten:

```python
print("starting server")                  # macht stdio-Server kaputt
print("starting server", file=sys.stderr) # OK
```

Besser: Verwende Logging, das standardmäßig nach `stderr` schreibt:

```python
import logging

logging.basicConfig(level=logging.INFO)
logging.info("starting server")
```

Wo du die Fehlermeldungen findest:

- `uv run fastmcp run server.py --log-level DEBUG`
- MCP Inspector: Panel **Server Notifications**
- Claude Code: `claude --debug`, oder `/mcp`, um den Serverstatus zu sehen
- ollmcp: `uv run ollmcp --debug`

----

## Übung: Debugging

Der Server [code/broken_mcp.py](code/broken_mcp.py) ist kaputt.
Eine Kollegin hat ihn in Claude Code registriert mit:

```bash
claude mcp add broken -- uv run --directory /home/alice/mcp_course broken_mcp.py
```

1. Registriere den Server auf die gleiche Weise in deinem Claude-Code- (oder ollmcp-) Client.
2. Finde heraus, warum sich der Server nicht verbindet. Korrigiere die Konfiguration.
3. Starte Claude mit `claude --debug` und tippe `/mcp`. Finde heraus, warum der Server Fehler meldet. Behebe sie.
4. Rufe das Tool direkt auf. Finde heraus, warum es fehlschlägt. Behebe es.

   ```bash
   uv run fastmcp call broken_mcp.py fibonacci n=20
   ```

<details>
<summary>Hinweise</summary>

- falscher `--directory`-Pfad in der Konfiguration (`claude mcp remove broken`, dann erneut hinzufügen)
- `print()` schreibt nach `stdout`
- der Parameter `n` hat keinen Type Hint, daher akzeptiert das Schema alles – auch einen String

</details>
