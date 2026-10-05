
# Vektorsuche

*"Der Kopierer qualmt"* und *"Drucker brennt"* haben kein Wort gemeinsam, meinen aber fast dasselbe.
Eine Vektordatenbank findet solche Ähnlichkeiten.

## Embeddings

Ein **Embedding** ist ein Vektor, der die Bedeutung eines Textes beschreibt.
Ein Modell bildet ähnliche Texte auf nahe beieinander liegende Vektoren ab.
Die Suche berechnet dann einfach Abstände.

### Übung 1: Embeddings ansehen

Öffne den [TensorFlow Embedding Projector](https://projector.tensorflow.org/).
Suche nach einem Wort und sieh dir die nächsten Nachbarn an.

----

## Übung 2: Ähnliche Incidents finden

Wir verwenden **ChromaDB**: eine Vektordatenbank, die sich als Python-Bibliothek installieren lässt und ein Embedding-Modell mitbringt.

```bash
uv add chromadb
```

Beim ersten Aufruf lädt ChromaDB ein kleines Embedding-Modell (ca. 80 MB) herunter.

### Schritt 1: Daten laden

Führe den Code in [code/incident_search.py](code/incident_search.py) aus.
Er benötigt eine CSV-Datei mit incidents: [code/incidents.csv](code/incidents.csv)


### Schritt 2: Suchen

```python
result = collection.query(query_texts=["the copier is smoking"], n_results=3)
```

Sieh dir `result["documents"]` und `result["distances"]` an. Je kleiner der Abstand, desto ähnlicher.

### Schritt 3: Ein MCP Tool daraus machen

Schreibe ein Tool `search_similar_incidents(text: str, n: int = 3) -> list[dict]`.

### Schritt 4: Mit einem LLM ausprobieren

> Mein Laptop findet kein WLAN. Gab es das schon mal?

- Findet das LLM den passenden alten Incident?
- Was passiert bei einer Suche, zu der es nichts Passendes gibt?

