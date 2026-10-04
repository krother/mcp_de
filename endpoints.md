

## Tools

Tools sind die zentralen Bausteine, mit denen dein LLM mit externen Systemen interagieren, Code ausführen und auf Daten zugreifen kann, die nicht in seinen Trainingsdaten enthalten sind. In FastMCP sind Tools Python-Funktionen, die LLMs über das MCP-Protokoll zur Verfügung gestellt werden. Tools in FastMCP verwandeln gewöhnliche Python-Funktionen in Fähigkeiten, die LLMs während einer Konversation aufrufen können. Wenn ein LLM sich entscheidet, ein Tool zu verwenden:

1. Es sendet einen Request mit Parametern, die auf dem Schema des Tools basieren.
2. FastMCP validiert diese Parameter anhand der Signatur deiner Funktion.
3. Deine Funktion wird mit den validierten Eingaben ausgeführt.
4. Das Ergebnis wird an das LLM zurückgegeben, das es in seiner Antwort verwenden kann.

So können LLMs Aufgaben erledigen wie Datenbanken abfragen, APIs aufrufen, Berechnungen durchführen oder auf Dateien zugreifen – und ihre Fähigkeiten über das hinaus erweitern, was in ihren Trainingsdaten steckt.

## Resources

Resources stellen Daten oder Dateien dar, die ein MCP Client lesen kann. Resource Templates erweitern dieses Konzept, indem sie Clients erlauben, dynamisch erzeugte Resources anzufordern, abhängig von Parametern in der URI.

FastMCP vereinfacht die Definition statischer und dynamischer Resources, vor allem mit dem Decorator @mcp.resource.

## Prompts

Prompts sind wiederverwendbare Nachrichtenvorlagen, die LLMs helfen, strukturierte, zielgerichtete Antworten zu erzeugen. FastMCP vereinfacht die Definition dieser Vorlagen, vor allem mit dem Decorator @mcp.prompt.

Was sind Prompts?
Prompts stellen parametrisierte Nachrichtenvorlagen für LLMs bereit. Wenn ein Client einen Prompt anfordert:

1. FastMCP findet die passende Prompt-Definition.
2. Falls sie Parameter hat, werden diese anhand deiner Funktionssignatur validiert.
3. Deine Funktion wird mit den validierten Eingaben ausgeführt.
4. Die erzeugte(n) Nachricht(en) werden an das LLM zurückgegeben, um seine Antwort zu steuern.

So kannst du konsistente, wiederverwendbare Vorlagen definieren, die LLMs über verschiedene Clients und Kontexte hinweg nutzen können.
