

# Task Scheduler — Priority Queue DSA

A desktop task management application built in Java with JavaFX, designed as a hands-on demonstration of core Data Structures & Algorithms: a **Max-Heap priority queue** for O(log n) scheduling, and a **Doubly Linked List** for full undo/redo history.

---

## Features

- **Priority-based scheduling** — tasks are always served highest-priority first using a Max-Heap
- **Add tasks** with a name, priority (1–100), and an optional deadline
- **Complete the top task** — extracts the highest-priority task in O(log n)
- **Reprioritize** any existing task by ID
- **Undo / Redo** — full action history backed by a doubly linked list with a cursor
- **Live stats bar** — shows queue size and the current top task at a glance
- **History panel** — visual timeline of every ADD, REMOVE, and UPDATE action
- **Toast notifications** — non-blocking feedback for every operation
- **Glassmorphism UI** — dark theme with translucent card components

---

## Data Structures

### Max-Heap (`MaxHeap.java`)
Backed by an `ArrayList<Task>`. Maintains the heap property (parent priority ≥ child priority) through `heapifyUp` and `heapifyDown`.

| Operation | Complexity |
|---|---|
| Insert | O(log n) |
| Extract Max | O(log n) |
| Peek | O(1) |
| Update Priority | O(n) find + O(log n) fix |
| Remove by ID | O(n) find + O(log n) fix |

### Doubly Linked List (`DoublyLinkedList.java`)
Each node (`HistoryNode`) stores a deep copy of the task at the moment of the action, plus a `prev`/`next` pointer. A `current` cursor tracks the "now" position to support branching undo/redo — recording a new action while undone discards the redo branch.

| Operation | Complexity |
|---|---|
| Record action | O(1) |
| Undo / Redo | O(1) |

---

## Project Structure

```
TaskScheduler/
├── pom.xml
├── run.cmd                          # One-click launcher for Windows (auto-downloads Java + Maven)
└── src/main/java/com/scheduler/
    ├── App.java                     # JavaFX entry point
    ├── module-info.java
    ├── model/
    │   ├── Task.java                # Task data model
    │   └── ActionType.java          # ADD | REMOVE | UPDATE_PRIORITY enum
    ├── ds/
    │   ├── MaxHeap.java             # Priority queue implementation
    │   ├── DoublyLinkedList.java    # Undo/redo history list
    │   ├── HistoryNode.java         # Node in the history list
    │   └── Scheduler.java          # Facade — ties together heap + history
    └── gui/
        ├── MainView.java            # Root layout, wires all components
        └── components/
            ├── ControlPanel.java    # Input forms with validation
            ├── TaskTablePanel.java  # Sortable task table
            ├── HistoryPanel.java    # Undo/redo history view
            ├── StatsBar.java        # Live queue stats
            ├── SidebarPanel.java    # Left sidebar decoration
            ├── ToastNotification.java
            └── GlassStyles.java     # Shared CSS-in-Java style constants
```

---

## Prerequisites

- Java 17+
- Maven 3.6+ (or use the included `mvnw` wrapper)

> **Windows users:** just double-click `run.cmd` — it auto-downloads Java 17 (Eclipse Temurin) and Maven 3.9.6 on first run. No manual setup needed.

---

## Running the App

**Using Maven:**
```bash
mvn javafx:run
```

**Using the Maven wrapper:**
```bash
./mvnw javafx:run       # macOS / Linux
mvnw.cmd javafx:run     # Windows
```

**Windows one-click:**
```
Double-click run.cmd
```
First run downloads Java and Maven (~230 MB total) and caches them in `.tools/` — subsequent runs start immediately.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| GUI | JavaFX 21.0.1 |
| Build | Maven 3.x |
| DSA | Custom Max-Heap + Doubly Linked List |

---

## How It Works

The `Scheduler` class is the single facade the GUI talks to. Every mutating operation touches both structures simultaneously:

```
addTask()        → MaxHeap.insert()    + DoublyLinkedList.record(ADD)
completeTask()   → MaxHeap.extractMax() + DoublyLinkedList.record(REMOVE)
reprioritize()   → MaxHeap.updatePriority() + DoublyLinkedList.record(UPDATE_PRIORITY)

undoLast()       → DoublyLinkedList.undo() → reverse the action on the heap
redoLast()       → DoublyLinkedList.redo() → replay the action on the heap
```

The GUI never touches `MaxHeap` or `DoublyLinkedList` directly — all coordination lives in `Scheduler`.
