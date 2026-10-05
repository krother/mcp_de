
## Übung 3: Begriffe zuordnen

Schneide die Karten aus und ordne jedem Begriff seine Beschreibung zu.

| Begriff |
|---------|
| Microservice |
| Docker |
| HTTP Transport |
| MCP Gateway |
| Proxy |
| zentrales Routing |
| Authentifizierung |
| Tool-Filterung |

| Beschreibung |
|--------------|
| entscheidet anhand des Tool-Namens, welcher Server eine Anfrage bekommt |
| ein kleiner, unabhängig deploybarer Dienst mit genau einer Aufgabe |
| blendet Tools aus, die ein bestimmter Client nicht sehen oder benutzen soll |
| verpackt einen Server mit allen Abhängigkeiten in einen Container |
| ein zentraler Eingang für viele MCP Server, mit Auth, Logging und Filtern |
| prüft, wer eine Anfrage stellt, z.B. per OAuth-Token |
| ein Server, der Anfragen unverändert an einen anderen Server weiterreicht |
| lässt einen MCP Server als Webservice laufen, auf den viele Clients zugreifen |



## Übung 2: Kartenset Software Engineering

Jede Gruppe zieht eine Karte und diskutiert: *Was bedeutet das für unseren MCP Server?*

```{card} 12-Factor App
Zwölf Regeln für Webdienste, z.B.: Konfiguration in Umgebungsvariablen, Logs als Stream, zustandslose Prozesse.
Unser Server speichert Incidents im Arbeitsspeicher – welche Regel verletzt das?
```

```{card} Funktionale Anforderungen
*Was* soll das System tun? Z.B. "Ein Incident kann angelegt, gelistet und geschlossen werden."
Wer legt fest, welche Tools ein LLM braucht?
```

```{card} Nicht-funktionale Anforderungen
*Wie gut* soll das System es tun? Antwortzeit, Kosten pro Anfrage, Datenschutz, Wartbarkeit.
Welche davon ändern sich, wenn ein LLM im Spiel ist?
```

```{card} Die 4 Anforderungen an Software
**Verfügbarkeit** – das System ist erreichbar.
**Zuverlässigkeit** – es liefert richtige Ergebnisse.
**Sicherheit (Security)** – es ist vor Angriffen geschützt.
**Betriebssicherheit (Safety)** – es richtet keinen Schaden an.
```

```{card} Software-Entropie
Software, die verändert wird, wird mit der Zeit unordentlicher – wenn niemand aktiv aufräumt.
Was passiert mit Tool-Beschreibungen, wenn sich das Modell dahinter ändert?
```
