# BPMN Generator

Generate **BPMN 2.0** diagrams from a **natural-language** process description, using a local LLM (Ollama) for the logic and a deterministic engine for the layout. Designed to run on **Google Colab with GPU**.

![Example of a generated diagram](GeneratoreFlussi.jpg)

---

## How it works

The architecture cleanly separates *what* the process should do (decided by the LLM) from *where* to place the nodes (computed by the code):

```
[Text description]
        │
   [Ollama LLM]  ──▶  structured JSON (lanes, nodes, flows)   ← LOGIC only
        │
  [Auto-layout]  ──▶  deterministic x/y coordinates           ← POSITIONS
        │
 [BPMN engine]   ──▶  export .bpmn / .drawio / .svg
```

**Why this separation:** an LLM is unreliable at computing clean coordinates and tends to produce overlapping nodes. By delegating only the process logic to the LLM (tasks, gateways, who-does-what) and computing positions with a deterministic algorithm, the diagram always comes out tidy and readable.

---

## Features

- **Natural-language input** — describe the process in plain words; the model converts it into a BPMN structure.
- **Deterministic auto-layout** — level-based node placement (BFS) from left to right, with no overlaps.
- **Supported BPMN constructs** — start/end events, tasks, subprocesses, exclusive (XOR), parallel (AND) and inclusive (OR) gateways, rework loops, and swimlanes.
- **Validation and auto-fixing** — checks the consistency of the generated logic and automatically corrects the most common mistakes made by small models.
- **Multiple export formats** — BPMN 2.0 XML (openable in Camunda Modeler or bpmn.io), draw.io XML (app.diagrams.net), and vector SVG.
- **GPU acceleration** — optimized to run on Google Colab with an NVIDIA GPU via Ollama.

---

## Requirements

- Python 3.10+
- [Ollama](https://ollama.com) running, with a model pulled
- `pip install requests ipywidgets`

Recommended models (in order of increasing capability):

| Model | Notes |
|---|---|
| `gemma4:e4b` | lightweight, fits Colab's T4 |
| `qwen2.5:7b` | good quality/speed trade-off |
| `gemma4:26b` | more capable, for complex processes |

---

## Running on Google Colab

**1. Enable the GPU:** `Runtime → Change runtime type → T4 GPU`

**2. Install and start Ollama:**

```python
!apt-get install -y zstd pciutils -qq
!curl -fsSL https://ollama.com/install.sh | sh

import subprocess, time, os
os.environ["OLLAMA_HOST"] = "0.0.0.0:11434"
subprocess.Popen(["ollama", "serve"])
time.sleep(5)
```

**3. Pull the model:**

```python
!ollama pull gemma4:e4b
```

**4. Launch the app:** paste the script into a cell and run it. The interface appears: type the process description, click **Generate BPMN**, and download in your preferred format.

---

## Example

**Input:**

> The customer submits an order. The sales office checks stock availability. If the product is available, logistics prepares the shipment and the customer receives the goods. If it is not available, production creates a work order, then logistics ships.

**Output:** a BPMN diagram with swimlanes for Customer, Sales, Logistics and Production, an exclusive gateway on availability, and the two branches converging back at shipping.

---

## Project structure

| File | Description |
|---|---|
| `bpmn_generator_colab.py` | full application (LLM + layout + export) |
| `BPMN-COLAB.html` | HTML export of the notebook |
| `GeneratoreFlussi.jpg` | example of a generated diagram |

---

## Technical notes

The engine builds an internal data model (`BProcess` → `BPool` → `BLane` → `BShape`) from which all export formats are derived. The layout algorithm assigns each node a *rank* (column) by traversing the flows forward from the start events, then distributes the nodes across the swimlanes while avoiding overlaps. Validation ensures that every branch of the process ends with an end node.

---

## License

Specify your chosen license here (e.g. MIT). See the `LICENSE` file.
