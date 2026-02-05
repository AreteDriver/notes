# Design Concepts Archive

Harvested concepts from loose project files (2026-02-04).

---

## PUDE: Power User Desktop Environment

**Vision**: "Bloomberg Terminal meets EVE Online meets Unix"

A desktop environment built for users who want transparency, consistency, information density, and control.

### Core Philosophy

| Principle | Description |
|-----------|-------------|
| **Transparency** | Don't hide what's happening. System state visible at a glance |
| **Consistency** | Same action = same result, everywhere. No "smart" context-dependent behaviors |
| **Composability** | Small tools that chain together. Scriptable interactions |
| **User Authority** | The user decides. No overriding choices |
| **Information Density** | Show more, click less. Wasted space is wasted opportunity |

### Radial Context Menu System

Right-click on anything, get contextual actions in a radial layout.

**Behavior:**
- Right-click tap (~<200ms): Traditional context menu (fallback)
- Right-click hold (~>200ms): Radial menu appears
- Drag toward action + release: Execute
- Release in center: Cancel

**Consistent Menu Layout:**
```
          [Primary Action]
               ^
    [Info]   <   >   [Secondary/Submenu]
               v
          [Destructive]
```

- **Top (12 o'clock)**: Primary action (open, run, execute)
- **Bottom (6 o'clock)**: Destructive action (delete, close, kill)
- **Left (9 o'clock)**: Information/properties
- **Right (3 o'clock)**: Secondary actions, submenus

### Window Manager Concepts

- **Magnetic Docking**: Windows snap to screen edges AND to each other
- **Window Groups**: Docked windows become a unit—move together, minimize together
- **Titlebar Shade**: Double-click titlebar collapses window to just the bar
- **Tabbed Stacking**: Drag window onto another to create tabbed group
- **Spatial Memory**: Windows remember position per-workspace and per-context
- **Opacity Rules**: Configurable transparency for focused/unfocused windows

### Visual Design Language (EVE-Inspired Default)

| Role | Color | Hex |
|------|-------|-----|
| Background | Deep space black | `#0a0a12` |
| Panel background | Dark blue-grey | `#12141c` |
| Primary accent | Cyan/teal | `#00d4ff` |
| Secondary accent | Amber | `#ff9900` |
| Warning | Orange-red | `#ff4444` |
| Success | Green | `#44ff88` |
| Text primary | Off-white | `#e0e0e0` |
| Text secondary | Grey | `#888888` |
| Border/grid | Subtle blue | `#1a1c28` |

### Technical Stack (Proposed)

| Layer | Technology |
|-------|------------|
| Display Server | Wayland (wlroots) |
| Compositor/WM | Custom (Rust or C) |
| Radial Daemon | PyQt6 → Rust |
| Panels/Widgets | Qt6 |
| IPC | D-Bus |
| Config | TOML |

---

## Real-Time Sync Patterns

From VDC portfolio architecture.

### Polling (Simpler, Streamlit-native)

```python
class DatabasePoller:
    def __init__(self, db_path: str, poll_interval_seconds: int = 3):
        self.db_path = db_path
        self.poll_interval = poll_interval_seconds
        self.last_hash = {}  # Track query results hash

    def has_data_changed(self, query_name: str, df: pd.DataFrame) -> bool:
        current_hash = hashlib.md5(pd.util.hash_pandas_object(df).values).hexdigest()
        if query_name not in self.last_hash:
            self.last_hash[query_name] = current_hash
            return True
        changed = self.last_hash[query_name] != current_hash
        self.last_hash[query_name] = current_hash
        return changed
```

**Pros**: No WebSocket complexity, works with Streamlit's refresh model
**Cons**: Minor latency (2-5 sec), higher DB read load

### Offline Queue Pattern

```python
def queue_offline_move(move: dict):
    if 'offline_queue' not in st.session_state:
        st.session_state['offline_queue'] = []
    move['queued_at'] = datetime.now().isoformat()
    st.session_state['offline_queue'].append(move)

def sync_offline_queue(db_conn):
    queue = st.session_state.get('offline_queue', [])
    for move in queue:
        move['synced_from_offline'] = True
        insert_movement(db_conn, move)
    st.session_state['offline_queue'] = []
```

Key points:
- Detect connection loss via failed DB write
- Store in session state with timestamp
- On reconnect, replay in order preserving original timestamps
- Mark `synced_from_offline=TRUE` for audit trail

---

## Tablet UI Patterns

### Mobile Responsiveness for Industrial Use

- **Font sizes**: 18px+ for readability on 7-10" tablets
- **Touch targets**: Buttons 20px+ padding for finger input
- **Landscape mode**: Assume devices used in landscape (wider layout)
- **Barcode scanner**: Configure as "keyboard wedge" (emits barcode + Enter key)
- **Feedback**: Visual + auditory confirmation on scan

### TV Dashboard (Kiosk Mode)

- Font size: 48px+ for main numbers, 32px+ for labels
- High contrast colors (readable in bright facility)
- No scrolling—all content visible at once
- No user interaction required
- Auto-refresh every 10-15 minutes
- "Last Updated" timestamp visible
- Optimized for 32-36" TVs at 15-30ft viewing distance

---

## Design Inspirations Reference

| Source | What to Take |
|--------|--------------|
| EVE Online | Radial menus, magnetic window docking, window stacking/tabbing, consistent right-click behavior, information density |
| Bloomberg Terminal | Data-dense panels, keyboard shortcuts, everything visible |
| Plan 9 | Consistency (everything is a file), right-click behaviors, acme-style interaction |
| NeXTSTEP | Services menu, dock, consistent UI guidelines |
| Classic Unix | Composability, scriptability, transparency |

---

*Harvested: 2026-02-04*
