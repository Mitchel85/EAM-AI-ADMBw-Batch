# ADMBw-Batch-Generator v1.0

![Status](https://img.shields.io/badge/Status-Active-success)
![Standard](https://img.shields.io/badge/Standard-MDG%202026.05-blue)
![Output](https://img.shields.io/badge/Output-JSON%20Batch-orange)
![Ziel](https://img.shields.io/badge/Ziel-ArAMIS%20WEB%20%7C%20Neo4j-purple)

KI-gestützter Agent zur Extraktion von ADMBw-konformen **Knoten und Kanten** aus Prosa-Dokumenten. Output ist ein **JSON-Batch**, der direkt in **ArAMIS WEB** (Neo4j) geladen werden kann.

> ⚠️ Erzeugt JSON-Batches zur manuellen Prüfung. Kein automatischer Import.

---

## 🎯 Was macht das Tool?

| Schritt | Beschreibung |
|---------|-------------|
| 1. Prosa-Analyse | Architekturrelevante Entitäten im Text identifizieren |
| 2. Stereotyp-Mapping | Jede Entität einem ADMBw-Stereotyp zuweisen |
| 3. Beziehungs-Extraktion | Kanten gemäß Topology-Regeln ableiten |
| 4. JSON-Batch | Strukturierte Ausgabe mit `nodes[]` und `edges[]` |
| 5. Ladeanweisung | Exakter Cypher-Befehl für ArAMIS WEB / Neo4j |

---

## 📦 Output-Format

```json
{
  "metadata": { "generator": "...", "node_count": 5, "edge_count": 4 },
  "nodes": [
    { "id": "n1", "name": "...", "stereotype": "Capability", "metaclass": "Class", "labels": ["Capability"], "properties": {} }
  ],
  "edges": [
    { "id": "e1", "source_id": "n1", "target_id": "n2", "type": "SATISFY", "connector": "Satisfy", "metaclass": "Dependency", "properties": {} }
  ]
}
```

→ Vollständiges Beispiel: [`examples/output-example.json`](examples/output-example.json)

---

## 🚀 In ArAMIS WEB laden

ArAMIS WEB nutzt **Neo4j mit APOC**. Der JSON-Batch wird über die APOC-Prozeduren geladen:

### 1. JSON bereitstellen
```bash
# In Neo4j-Importverzeichnis kopieren (Podman)
podman cp batch.json <neo4j-container>:/var/lib/neo4j/import/
```

### 2. KNOTEN laden (Neo4j Browser / cypher-shell)
```cypher
CALL apoc.load.json("file:///import/batch.json") YIELD value
UNWIND value.nodes AS n
MERGE (node {id: n.id})
SET node.name = n.name,
    node.stereotype = n.stereotype,
    node.metaclass = n.metaclass
SET node.shortName = n.properties.shortName,
    node.serviceType = n.properties.serviceType
WITH node, n
CALL apoc.create.addLabels(node, n.labels) YIELD node AS labeledNode
RETURN count(labeledNode) AS KnotenErstellt;
```

### 3. KANTEN laden
```cypher
CALL apoc.load.json("file:///import/batch.json") YIELD value
UNWIND value.edges AS e
MATCH (src {id: e.source_id}), (tgt {id: e.target_id})
CALL apoc.merge.relationship(
  src, e.type,
  {connector: e.type},
  {metaclass: e.metaclass},
  tgt
) YIELD rel
RETURN count(rel) AS BeziehungenErstellt;
```

### 4. Prüfen
```cypher
MATCH (n)-[e]-(m) RETURN n, e, m;
```

> ⚠️ Properties **einzeln setzen**, nicht als Map! Nur gefüllte Properties auflisten.

---

## 📚 Wissensbasis

Vier Knowledge-Dateien (in OpenWebUI als RAG-Wissen eingebunden):

| Datei | Inhalt |
|-------|--------|
| `ADMBw-Knowledge-Stereotypes.md` | 443 eindeutige Stereotype mit AppliesTo (MDG 2026.05) |
| `ADMBw-Knowledge-Viewpoints.md` | 53 Viewpoints mit erlaubten Elementen |
| `ADMBw-Knowledge-Topology.md` | Source → Connector → Target Regeln |
| `ADMBw-Knowledge-Connectors.md` | EA-Metatypen + Connector-Übersicht |

---

## 🔧 Einrichtung (OpenWebUI)

1. **Modell anlegen** → Inhalt von `system_prompt.md` ins Feld **System Prompt**
2. **Knowledge** → Vier `ADMBw-Knowledge-*.md`-Dateien hochladen und verknüpfen
3. Fertig. Erste Nachricht: Prosa-Dokument einfügen oder: *„Extrahiere ADMBw-Knoten und Kanten aus folgendem Text: …"*

---

## 📁 Repository

| Datei | Zweck |
|-------|-------|
| `system_prompt.md` | System-Prompt (Extraktionslogik + JSON-Format) |
| `ADMBw-Knowledge-Stereotypes.md` | Stereotyp-Katalog |
| `ADMBw-Knowledge-Viewpoints.md` | Viewpoint-Regeln |
| `ADMBw-Knowledge-Topology.md` | Topologie-Regeln |
| `ADMBw-Knowledge-Connectors.md` | Konnektor-Regeln |
| `examples/output-example.json` | Beispiel-JSON-Batch |

---

**Autor:** Michael Estel (mit KI-Agent)
