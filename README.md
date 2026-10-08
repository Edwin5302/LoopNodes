# LoopNodes

**ClockRush Loop Map** is a single-file (`index.html`) narrative mapping tool for the time-loop game *ClockRush*. Open it in a browser (it loads [vis-network](https://visjs.github.io/vis-network/) from a CDN, so it needs an internet connection the first time). There is no build step.

## Modes

| | Edit Mode | Simulation Mode |
|---|---|---|
| Canvas | drag nodes, click a node or arrow to edit it | read-only |
| Right sidebar | properties of the selected node / edge | hidden |
| Left sidebar | hidden | time-in-loop slider (0:00 – 7:00) + a checkbox for every Knowledge Gain and Permanent Unlock node |

**Add Edge** uses vis-network's native connect mode: click it, then drag from the source node to the target node. `Delete` removes the selection and `Esc` cancels. Every destructive action (delete, clear, import) offers an **Undo** toast. The map autosaves to the browser's `localStorage`.

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
  "version": 1,
  "nodes": [
    { "id": "n1", "title": "Overhear the Landlord", "type": "Knowledge Gain",
      "act": "Act 1", "time": "0:30", "text": "Dialogue / description", "x": 270, "y": -120 }
  ],
  "edges": [
    { "id": "e1", "from": "n1", "to": "n2", "type": "Knowledge Prerequisite" }
  ]
}
```

* `type` (nodes): `Interaction`, `Knowledge Gain`, `Permanent Unlock`
* `act`: `Act 1`, `Act 2`, `Act 3`, `Any`
* `time`: `M:SS`, or empty for no requirement
* `type` (edges): `Cause-Effect`, `Knowledge Prerequisite`, `Mechanical Prerequisite`
* `x` / `y` are optional on import; nodes without them are laid out left-to-right automatically. Short aliases (`knowledge`, `unlock`, `mechanical`, `1`, …) are accepted too.
