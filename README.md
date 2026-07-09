# ADMBw-Batch-Generator v1.0

![Status](https://img.shields.io/badge/Status-Active-success)
![Standard](https://img.shields.io/badge/Standard-ADMBw%20v2025.10-blue)
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

ArAMIS WEB nutzt **Neo4j als Graphdatenbank** (Podman-Container). Der JSON-Batch wird als **Cypher-Script** geladen:

### Variante A: Cypher-Shell (Podman)
```bash
# Neo4j-Container ermitteln
podman ps | grep neo4j

# Cypher ausführen
podman exec -i <neo4j-container> cypher-shell -u neo4j -p <passwort> << 'CYPHER'
MERGE (n1:Capability {name: "Einsatzführung"})
  SET n1.stereotype = "Capability", n1.metaclass = "Class";

MATCH (src {name: "Gefechtsstand"}), (tgt {name: "Einsatzführung"})
MERGE (src)-[:EXHIBITS {connector: "Exhibits"}]->(tgt);
CYPHER
```

### Variante B: Neo4j Browser
- URL: `http://<aramis-host>:7474`
- Cypher-Statements dort einfügen und ausführen

> 💡 Die Verbindungsdaten (Host, Benutzer, Passwort) sind dieselben, die ArAMIS WEB für seine Neo4j-Verbindung nutzt (siehe `podman-compose/.env`).

---

## 📚 Wissensbasis

Vier Knowledge-Dateien (in OpenWebUI als RAG-Wissen eingebunden):

| Datei | Inhalt |
|-------|--------|
| `ADMBw-Knowledge-Stereotypes.md` | 317 Stereotype mit AppliesTo |
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
