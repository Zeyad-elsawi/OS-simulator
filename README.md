# OS Simulator

A **clock-driven operating system simulator** written in C: multiprogramming with simulated RAM, demand paging (swap to disk), process scheduling, and synchronization primitives. An optional **Python + CustomTkinter** GUI visualizes the simulator’s trace as a timeline.

**Repository:** [github.com/Zeyad-elsawi/OS-simulator](https://github.com/Zeyad-elsawi/OS-simulator)

**Demo video:** [os_demo.mkv on Google Drive](https://drive.google.com/file/d/1-rv7xTZBXmCMT2rRHBpIYc48wlYbBm1_/view?usp=sharing)

---

## Features

| Area | Behavior |
|------|------------|
| **Memory** | Fixed-size RAM (40 words). Each process occupies contiguous words: PCB/metadata overhead plus instruction text. |
| **Paging / swap** | When RAM is full, a resident process can be swapped out to `swap_<PID>.txt` and brought back with **swap-in**. |
| **Schedulers** | **Round-Robin** (FIFO ready queue + quantum), **HRRN** (highest response ratio next), **MLFQ** (multiple levels, time slices \(2^{\text{level}}\)). |
| **Synchronization** | Three mutex-like resources: `file`, `userInput`, `userOutput`, each with a FIFO wait queue. |
| **Interpreter** | Executes one instruction per simulated clock tick from RAM (assign, I/O, `print`, `readFile`/`writeFile`, `semWait`/`semSignal`, etc.). |
| **Trace** | Structured console log: dispatch, CPU line per tick, queue states, RAM dump, swap events. |

Source modules live under `OS_proj/src/`; see [`OS_proj/docs/README_src.md`](OS_proj/docs/README_src.md) for a file-by-file walkthrough.

---

## Repository layout

```
.
├── CMakeLists.txt          # Builds the C simulator
├── OS_proj/
│   ├── include/            # Headers (pcb, memory, scheduler, mutex, interpreter, trace, …)
│   ├── src/                # Implementation
│   ├── programs/           # Sample program files (*.txt)
│   └── docs/               # Extra documentation (e.g. README_src.md)
├── gui/
│   ├── os_simulator_gui.py # Visual timeline (runs built os_sim as subprocess)
│   ├── requirements.txt    # customtkinter
│   └── run_gui.bat         # Optional Windows launcher
└── README.md
```

---

## Build (C simulator)

Requires **CMake** (3.16+) and a C11 compiler (GCC, Clang, or MSVC).

```powershell
cd path\to\OS-simulator
cmake -S . -B build
cmake --build build
```

On Windows with MSVC, the executable is typically `build\Release\os_sim.exe` or `build\Debug\os_sim.exe`; MinGW often produces `build\os_sim.exe`. The GUI looks for `build\os_sim.exe` or `build\os_sim`.

---

## Run — command line

```text
os_sim [options] <Program_1.txt> [Program_2.txt ...]
```

| Option | Meaning |
|--------|---------|
| `--rr` | Round-Robin (default if policy omitted). |
| `--hrrn` | Highest Response Ratio Next. |
| `--mlfq` | Multi-Level Feedback Queue. |
| `--quantum N` | Instructions per slice for RR (default `N ≥ 1`). |
| `--arrive t0,t1,...` | Per-process arrival times (comma-separated). Default is `0` for all. |
| `--help` | Print usage. |

With **no program files**, the simulator runs a small built-in two-process demo (RR, quantum 2).

**Examples**

```powershell
.\build\os_sim.exe --rr --quantum 2 OS_proj\programs\Program_1.txt OS_proj\programs\Program_2.txt
.\build\os_sim.exe --hrrn --arrive 0,3 OS_proj\programs\Program_1.txt OS_proj\programs\Program_2.txt
.\build\os_sim.exe --mlfq OS_proj\programs\Program_1.txt
```

Program files are plain text: one instruction per line; empty lines and lines starting with `#` are ignored.

---

## Run — GUI (optional)

From the repository root (after building `os_sim`):

```powershell
pip install -r gui\requirements.txt
python gui\os_simulator_gui.py
```

The GUI runs the simulator executable, parses its trace output, and shows a **CSEN 602–style** visual timeline. On Windows you can use `gui\run_gui.bat` if configured for your environment.

---

## Instruction set (summary)

The interpreter recognizes forms such as:

- `assign <var> input` / `assign <var> <literal>` / `assign <var> readFile <path>`
- `print <var>` / `printFromTo <v1> <v2>`
- `readFile …` / `writeFile …`
- `semWait userInput` | `semSignal userInput` (and `userOutput`, `file`)

Exact parsing and edge cases are implemented in `OS_proj/src/interpreter.c`.

---

## Limits and artifacts

- Up to **8** processes (`MAX_PROCESSES` in `main.c`).
- Swap files `swap_<PID>.txt` may appear in the **current working directory** when swapping occurs; you can delete them between runs.

---

## License

Unless otherwise noted by your course or institution, treat this project as educational coursework—add a license file if you want explicit open-source terms.
