# LoopNodes

**ClockRush Loop Map** is a single-file (`index.html`) narrative mapping tool for the time-loop game *ClockRush*. Open it in a browser (it loads [vis-network](https://visjs.github.io/vis-network/) from a CDN, so it needs an internet connection the first time). There is no build step.

## Modes

| | Edit Mode | Simulation Mode |
|---|---|---|
| Canvas | drag nodes, click a node or arrow to edit it | read-only |
| Right sidebar | properties of the selected node / edge | hidden |
| Left sidebar | hidden | time-in-loop slider (0:00 – 7:00) + a checkbox for every Knowledge Gain and Permanent Unlock node |

**Add Edge** uses vis-network's native connect mode: click it, then drag from the source node to the target node. **Add next node** (in a node's sidebar) creates a linked node to its right, inheriting act, location and time. `Delete` removes the selection and `Esc` cancels. Every destructive action (delete, clear, import, arrange) offers an **Undo** toast. The map autosaves to the browser's `localStorage`.

## The Start node

Every map has exactly one **Start** node (dark, with a ▶ START flag), and every other node should come from it.

* It can't be deleted, nothing can point *into* it, and its type, act and time are fixed (title, location and text are editable). **Clear** keeps it.
* A node that can't be reached from Start by following arrows gets a **dashed outline**, and a warning chip at the top of the canvas counts them (click it to jump to the next one).
* Maps and JSON files that have no Start get one added automatically, feeding the nodes that have no incoming arrow.

## Locations, Acts and arranging

Each node has a **Location** (free text with autocomplete; `cellar` and `Cellar` are treated as the same place). Each node also shows an **act badge** (1 / 2 / 3 / ∞, in teal / violet / pink / grey).

* **Locations** (canvas toggle): draws a labelled region around each location's nodes.
* **Act bands** (canvas toggle): tints the area covered by each Act. With *Locations* on, you get one band per Act inside each location.
* Both overlays are computed from the nodes' live positions, so they follow nodes as you drag them (a messy map gives messy, overlapping regions), and they work in Simulation Mode too.
* **Arrange ▾** (Edit Mode) re-positions every node, animated and undoable:
  * **Tidy up**: a left → right flow from Start, with one column band per Act.
  * **Group by Location**: one region per location, each holding its own flow (Acts as columns), packed into a landscape grid with the family that contains Start first.

### Edge types

| Type | Look | Meaning |
|---|---|---|
| Cause-Effect | solid black | source leads to target; never gates availability |
| Knowledge Prerequisite | dashed blue | target needs the knowledge gained from the source |
| Mechanical Prerequisite | thick red | target needs the permanent unlock from the source |

### Availability (Simulation Mode)

A node is **available** when its time requirement is `<=` the slider **and** the source of every inbound Knowledge/Mechanical prerequisite edge is ticked as obtained. Available nodes glow; unavailable nodes are faded to 30 % opacity. Hover a node to see exactly what it is still missing.

## JSON format

```json
{
  "format": "clockrush-loop-map",
  "version": 2,
  "view": { "locations": true, "acts": true },
  "nodes": [
    { "id": "n1", "start": true, "title": "Wake Up", "type": "Interaction", "act": "Act 1",
      "location": "Boarding House", "time": "", "text": "Where every loop begins.", "x": 0, "y": 0 },
    { "id": "n2", "title": "Overhear the Landlord", "type": "Knowledge Gain", "act": "Act 1",
      "location": "Boarding House", "time": "0:30", "text": "Dialogue / description", "x": 270, "y": -120 }
  ],
  "edges": [
    { "id": "e1", "from": "n1", "to": "n2", "type": "Cause-Effect" }
  ]
}
```

* `type` (nodes): `Interaction`, `Knowledge Gain`, `Permanent Unlock`
* `act`: `Act 1`, `Act 2`, `Act 3`, `Any`
* `location`: any text, or empty
* `time`: `M:SS`, or empty for no requirement
* `start`: `true` on the Start node. If a file has none, one is added; if it has several, only the first is kept.
* `type` (edges): `Cause-Effect`, `Knowledge Prerequisite`, `Mechanical Prerequisite`. Arrows into Start are dropped.
* `view`, `x` and `y` are optional on import. With no positions at all the nodes are laid out automatically, grouped by location when the file names two or more. Short aliases (`knowledge`, `unlock`, `mechanical`, `1`, …) are accepted too, and files from version 1 (no `location` / `start`) still load.
