# Frontend Patterns

React, TypeScript, and UI development patterns.

---

## State Management with Zustand

### Why Zustand over Redux
- Simpler mental model, less boilerplate
- Built-in `persist` middleware for localStorage
- No provider wrapper needed
- Good enough for most scales

### Pattern: Separate Stores by Concern
```typescript
useUIStore          // Transient UI state (sidebar, filters)
usePreferencesStore // Persisted preferences (theme, notifications)
useDataStore        // Server-synced data
```

### Persisted Store Example
```typescript
export const usePreferencesStore = create<PreferencesState>()(
  persist(
    (set) => ({
      theme: 'system',
      setTheme: (theme) => set({ theme }),
    }),
    { name: 'app-preferences' }  // localStorage key
  )
);
```

---

## Dark Mode Implementation

### CSS Variables Approach
```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
}
```

### Tailwind Config
```javascript
module.exports = {
  darkMode: ["class"],  // Toggle via class on <html>
  // ...
}
```

### Theme Provider
```typescript
function ThemeProvider({ children }) {
  const theme = usePreferencesStore((s) => s.theme);

  useEffect(() => {
    const root = document.documentElement;
    root.classList.remove('light', 'dark');

    if (theme === 'system') {
      const systemTheme = window.matchMedia('(prefers-color-scheme: dark)').matches
        ? 'dark' : 'light';
      root.classList.add(systemTheme);
    } else {
      root.classList.add(theme);
    }
  }, [theme]);

  return <>{children}</>;
}
```

### Theme Toggle (Cycle Pattern)
```typescript
const cycleTheme = () => {
  const themes = ['light', 'dark', 'system'];
  const next = (themes.indexOf(theme) + 1) % themes.length;
  setTheme(themes[next]);
};
```

---

## shadcn/ui Patterns

### Why shadcn/ui
- Copy-paste components, not npm packages
- Full control over styling
- Consistent design language
- Radix primitives for accessibility

### Component Organization
```
src/components/
  ui/           # Base shadcn components (button, card, etc.)
  feature/      # Feature-specific composed components
  ThemeToggle.tsx
  Layout.tsx
```

---

## React Query Patterns

### Basic Query Hook
```typescript
export function useItems() {
  return useQuery({
    queryKey: ['items'],
    queryFn: () => apiClient.getItems(),
  });
}
```

### Mutation with Invalidation
```typescript
export function useCreateItem() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data) => apiClient.createItem(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['items'] });
    },
  });
}
```

### Usage Pattern
```typescript
const { data, isLoading, error, refetch } = useItems();
const createMutation = useCreateItem();

// In handler
createMutation.mutate(newItem);
```

---

## Component Patterns

### Toggle/Switch (Pure CSS)
```tsx
<label className="relative inline-flex items-center cursor-pointer">
  <input type="checkbox" className="sr-only peer" checked={value} onChange={...} />
  <div className="w-11 h-6 bg-muted rounded-full peer
                  peer-checked:after:translate-x-full
                  peer-checked:bg-primary
                  after:content-[''] after:absolute
                  after:top-[2px] after:left-[2px]
                  after:bg-background after:rounded-full
                  after:h-5 after:w-5 after:transition-all">
  </div>
</label>
```

### Card with Expandable Config
```tsx
function ConfigCard({ item, isExpanded, onToggle }) {
  return (
    <Card>
      <CardHeader onClick={onToggle}>
        {/* Summary view */}
      </CardHeader>
      {isExpanded && (
        <CardContent>
          {/* Expanded config form */}
        </CardContent>
      )}
    </Card>
  );
}
```

---

## Project Structure

```
src/
  api/           # API client
  components/
    ui/          # Base components
    feature/     # Feature components
  hooks/         # Custom React hooks
  pages/         # Route-level components
  stores/        # Zustand stores
  types/         # TypeScript interfaces
  lib/           # Utilities (cn, etc.)
```

---

## Feature Implementation Flow

1. Define types in `types/`
2. Add API methods in `api/client.ts`
3. Create React Query hooks in `hooks/`
4. Build UI components
5. Add route in `App.tsx`
6. Update navigation

---

## Mock Data Pattern

For development without backend:
```typescript
const { data: apiData, isLoading } = useItems();

// Fallback to mock data
const items = apiData || mockItems;
```

Benefits:
- Frontend developed independently
- Demo mode works without API
- Easy to test edge cases

---

## Common Issues

### Port Conflicts
```bash
# Check what's using a port
lsof -i :5173

# Use different port in Vite
vite --port 5174
```

### VITE_API_URL Not Working
Create `.env.local` (not `.env`):
```
VITE_API_URL=http://localhost:8001
```

### Build Artifact in Git
`tsconfig.tsbuildinfo` is a build artifact. Either:
- Add to `.gitignore`
- Or `git stash` before pull/rebase

---

## React Hooks Lint Fixes

### set-state-in-effect Rule

**Problem:** ESLint react-hooks/set-state-in-effect flags setState calls in effects.

**When it's valid:** Data fetching, syncing props to state are legitimate uses.

**Fix:** Use block-level eslint-disable:

```tsx
// Data fetching on mount - valid use case
/* eslint-disable react-hooks/set-state-in-effect */
useEffect(() => {
  loadData();
}, [loadData]);
/* eslint-enable react-hooks/set-state-in-effect */

// Syncing props to local state when selection changes
/* eslint-disable react-hooks/set-state-in-effect */
useEffect(() => {
  setLocalValue(propValue || defaultValue);
}, [propValue, selectedItem]);
/* eslint-enable react-hooks/set-state-in-effect */
```

### Function Ordering for useCallback

**Problem:** Using a function before it's declared causes "accessed before declared" error.

**Fix:** Define functions in dependency order (dependencies first):

```tsx
// WRONG - handleMessage used before declared
const connect = useCallback(() => {
  ws.onmessage = (e) => handleMessage(e);  // Error!
}, []);

const handleMessage = useCallback((e) => { ... }, []);

// CORRECT - handleMessage defined first
const handleMessage = useCallback((e) => { ... }, []);

const connect = useCallback(() => {
  ws.onmessage = (e) => handleMessage(e);
}, [handleMessage]);  // Include in deps
```

### Reconnection Callback Pattern

**Problem:** Reconnect timer needs to call `connect()` but can't have stale closure.

**Fix:** Use ref to hold current function:

```tsx
const connectRef = useRef<() => void>(() => {});

const connect = useCallback(() => {
  const ws = new WebSocket(url);
  ws.onclose = () => {
    // Use ref to avoid stale closure
    setTimeout(() => connectRef.current(), 2000);
  };
}, [handleMessage]);

// Keep ref updated
useEffect(() => {
  connectRef.current = connect;
}, [connect]);
```

---

## WebSocket + REST API Architecture

### Pattern: Hardware Control Web GUI

For controlling hardware devices via web interface:

```
┌─────────────────────┐     WebSocket      ┌─────────────────────┐
│   React Frontend    │ ◄──────────────────► │  FastAPI Backend    │
│   (localhost:5173)  │                      │  (localhost:8765)   │
└─────────────────────┘     REST API       └──────────┬──────────┘
                                                       │
                                                       │ hidapi/libusb
                                                       ▼
                                              ┌─────────────────┐
                                              │   Hardware      │
                                              └─────────────────┘
```

**WebSocket for:** Real-time events (button presses, state changes)
**REST API for:** CRUD operations (profiles, macros, settings)

### Frontend WebSocket Hook

```typescript
export function useDeviceWebSocket() {
  const [state, setState] = useState<DeviceState>(initialState);
  const [connected, setConnected] = useState(false);
  const wsRef = useRef<WebSocket | null>(null);
  const connectRef = useRef<() => void>(() => {});

  const handleMessage = useCallback((msg: { type: string; [key: string]: unknown }) => {
    switch (msg.type) {
      case 'state': setState(msg.data as DeviceState); break;
      case 'button_pressed':
        setState(prev => ({ ...prev, pressed: [...prev.pressed, msg.button as string] }));
        break;
      // ... other message types
    }
  }, []);

  const connect = useCallback(() => {
    const ws = new WebSocket('ws://127.0.0.1:8765/ws');
    ws.onopen = () => { setConnected(true); ws.send(JSON.stringify({ type: 'get_state' })); };
    ws.onclose = () => { setConnected(false); setTimeout(() => connectRef.current(), 2000); };
    ws.onmessage = (e) => handleMessage(JSON.parse(e.data));
    wsRef.current = ws;
  }, [handleMessage]);

  useEffect(() => { connectRef.current = connect; }, [connect]);
  useEffect(() => { connect(); return () => wsRef.current?.close(); }, [connect]);

  const sendMessage = useCallback((msg: object) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify(msg));
    }
  }, []);

  return { state, connected, sendMessage };
}
```

### Backend with Simulation Mode

```python
# FastAPI backend that works with or without hardware
from fastapi import FastAPI, WebSocket

class DeviceState:
    def __init__(self):
        self.connected = False  # True when hardware detected
        self.pressed_keys: set[str] = set()

device_state = DeviceState()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    await websocket.send_json({"type": "state", "data": device_state.to_dict()})

    while True:
        data = await websocket.receive_json()
        if data["type"] == "simulate_press":
            # Works even without hardware
            device_state.pressed_keys.add(data["button"])
            await broadcast({"type": "button_pressed", "button": data["button"]})

async def device_event_loop():
    """Poll hardware, broadcast events. Falls back to simulation if no device."""
    try:
        device = find_device()
        device_state.connected = True
        while True:
            events = device.read()
            for event in events:
                await broadcast({"type": "button_pressed", "button": event.button})
    except DeviceNotFoundError:
        pass  # Run in simulation mode
```

**Benefits:**
- Development without hardware
- UI fully testable
- Same code path for real and simulated input

---

*Last updated: 2026-01-26*
