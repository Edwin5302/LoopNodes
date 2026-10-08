# LoopNodes

**ClockRush Loop Map** is a single-file (`index.html`) narrative mapping tool for the time-loop game *ClockRush*. Open it in a browser (it loads [vis-network](https://visjs.github.io/vis-network/) from a CDN, so it needs an internet connection the first time). There is no build step.

## Modes

| | Edit Mode | Simulation Mode |
|---|---|---|
| Canvas | drag nodes, click a node or arrow to edit it | read-only |
| Right sidebar | properties of the selected node / edge | hidden |
| Left sidebar | hidden | the player's Act, the time in the loop, and the checklists described below |

**Add Edge** uses vis-network's native connect mode: click it, then drag from the source node to the target node. **Add next node** (in a node's sidebar) creates a linked node to its right, inheriting act, location and time. `Delete` removes the selection and `Esc` cancels. Every destructive action (delete, clear, import, arrange) offers an **Undo** toast. The map autosaves to the browser's `localStorage`.

## What gates a node

Every node can have up to five requirements. In Simulation Mode a node is **available** only when **all** of them hold; otherwise it is faded to 30 % opacity (hover it to see exactly what is missing).

| Requirement | Where you set it | Kept between loops? | Simulation control |
|---|---|---|---|
| **Act** | the node's *Act* field (`Any` = none) | yes | *Player is in* Act 1 / 2 / 3. An Act is a requirement, so being in Act 3 keeps Act 1 and 2 nodes available. |
| **Time** | the node's *Time Requirement* (`M:SS`) | no | *Time Passed in Loop* slider, 0:00 – 7:00 (inclusive) |
| **Knowledge** | a **Knowledge Prerequisite** arrow from a *Knowledge Gain* node | yes | *Knowledge* checklist |
| **Permanent unlock** | the node's **Requires permanent unlocks** list, **no arrow** | yes | *Permanent Unlocks* checklist |
| **Done earlier this loop** | a **Mechanical Prerequisite** arrow from the node that must come first | no | *Done this loop* checklist |

**Start new loop** puts the clock back to 0:00 and clears *Done this loop*, but keeps the Act, knowledge and permanent unlocks. **Reset simulation** clears everything (Act 1, 0:00, nothing ticked).

### Node types

| Type | Meaning |
|---|---|
| Interaction | something the player does |
| Knowledge Gain | learning something that is kept between loops (used through Knowledge Prerequisite arrows) |
| Permanent Unlock | **the obtainment of a permanent unlock**, kept between loops. Other nodes list it under *Requires permanent unlocks*; no arrow needed. A node with requirements shows a padlock badge with the count (red; green in the simulation once they are all held). |

### Arrow types

| Type | Look | Meaning |
|---|---|---|
| Cause-Effect | solid black | the default arrow: *after this, you can do that*. Shows flow only and never blocks anything. |
| Knowledge Prerequisite | dashed blue | the target needs the knowledge gained at the source |
| Mechanical Prerequisite | thick red | an **in-loop** requirement: the target needs the source to have been done earlier in the **same loop** |

## The Start node

Every map has exactly one **Start** node (dark, with a ▶ START flag), and every other node should come from it.

* It can't be deleted, nothing can point *into* it, and its type, act and time are fixed (title, location and text are editable). **Clear** keeps it.
* A node that can't be reached from Start (following arrows, or "requires unlock X" links) gets a **dashed outline**, and a warning chip at the top of the canvas counts them (click it to jump to the next one).
* Maps and JSON files that have no Start get one added automatically, feeding the nodes that have no incoming link.

## Locations, Acts and arranging

Each node has a **Location** (free text with autocomplete; `cellar` and `Cellar` are treated as the same place) and an **act badge** (1 / 2 / 3 / ∞, in teal / violet / pink / grey).

* **Locations** (canvas toggle): a labelled region around each location's nodes.
* **Act bands** (canvas toggle): a tinted band per Act. With both on, the two nest: location regions with Act bands inside, or Act regions with location bands inside, depending on how you last arranged.
* Both overlays follow the nodes as you drag them (a messy map gives messy, overlapping regions), and they work in Simulation Mode too.
* **Arrange ▾** (Edit Mode) re-positions every node, animated and undoable:
  * **Tidy up**: a left → right flow from Start, with one column band per Act.
  * **Group by Location**: one region per location, each holding its own flow (Acts as columns); the family containing Start comes first.
  * **Group by Act**: one region per Act (Act 1, 2, 3, Any), each holding its own flow (locations as columns).

## JSON format

```json
{
  "format": "clockrush-loop-map",
  "version": 3,
  "view": { "locations": true, "acts": true, "nest": "location" },
  "nodes": [
    { "id": "n1", "start": true, "title": "Wake Up", "type": "Interaction", "act": "Act 1",
      "location": "Boarding House", "time": "", "text": "Where every loop begins.", "x": 0, "y": 0 },
    { "id": "n4", "title": "Iron Crowbar", "type": "Permanent Unlock", "act": "Act 2",
      "location": "Market Square", "time": "3:00", "text": "Obtaining the crowbar.", "x": 540, "y": 120 },
    { "id": "n9", "title": "Pry Open the Door", "type": "Interaction", "act": "Act 3",
      "location": "Clocktower", "requiresUnlocks": ["n4"], "time": "6:00", "text": "", "x": 1350, "y": 20 }
  ],
  "edges": [
    { "id": "e1", "from": "n1", "to": "n4", "type": "Cause-Effect" },
    { "id": "e2", "from": "n4", "to": "n9", "type": "Cause-Effect" }
  ]
}
```

* `type` (nodes): `Interaction`, `Knowledge Gain`, `Permanent Unlock`
* `act`: `Act 1`, `Act 2`, `Act 3`, `Any`
* `location`: any text, or empty
* `time`: `M:SS`, or empty for no requirement
* `requiresUnlocks`: ids of `Permanent Unlock` nodes this node requires (`requires` also works). Unknown ids, self-references and ids of other node types are dropped on import.
* `start`: `true` on the Start node. If a file has none, one is added; if it has several, only the first is kept.
* `type` (edges): `Cause-Effect`, `Knowledge Prerequisite`, `Mechanical Prerequisite`. Arrows into Start are dropped.
* `view`, `x` and `y` are optional. With no positions at all the nodes are laid out automatically, grouped by location when the file names two or more. Short aliases (`knowledge`, `unlock`, `mechanical`, `1`, …) are accepted too.
* **Older files:** in files of version 1 or 2 a red *Mechanical Prerequisite* arrow out of a Permanent Unlock node meant "needs this unlock". On import such an arrow becomes a plain Cause-Effect arrow (so the node stays connected) plus a `requiresUnlocks` entry on its target. Files without `location` / `start` also still load.
