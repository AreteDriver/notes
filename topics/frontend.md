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

## Next.js Migration Notes

### Next.js 14 → 15 (Pages Router)
- **Zero code changes** for Pages Router apps
- React 18 → 19, ESLint 8 → 9 come along for the ride
- ESLint 9 works fine with `.eslintrc.json` (no flat config migration needed)
- `next lint` deprecated in v15 — migrate to ESLint CLI: `npx @next/codemod@canary next-lint-to-eslint-cli .`
- `next build` auto-updates `next-env.d.ts` — include in commits
- `next-i18next@15` compatible with Next.js 15

### Why v15 not v16
- Next.js 16 deprecates Pages Router entirely
- v15 is the last version with full Pages Router support
- App Router migration is a separate, larger effort
- v15 gets you React 19 + modern ESLint without architecture changes

### ESLint 9 Flat Config (Next.js)
- `eslint-config-next` v15 does NOT support flat config — legacy format only
- Use `@next/eslint-plugin-next` directly: `nextPlugin.flatConfig.coreWebVitals`
- Need `eslint-plugin-react` `configs.flat["jsx-runtime"]` for JSX parsing (legacy config bundled this automatically)
- The `next-lint-to-eslint-cli` codemod is broken — generates 3 errors. Write manually:

```javascript
import reactPlugin from "eslint-plugin-react";
import nextPlugin from "@next/eslint-plugin-next";

export default [
    {
        ignores: [".next/", "out/", "android/", "ios/", "coverage/"],
    },
    reactPlugin.configs.flat["jsx-runtime"],
    nextPlugin.flatConfig.coreWebVitals,
    {
        rules: {
            // your overrides
        },
    },
];
```

**IMPORTANT:** `eslint .` lints everything — unlike `next lint`, it has no implicit ignores. Always add ignores for build artifact dirs (`.next/`, `out/`, platform dirs, `coverage/`).

---

## SSE Streaming for Chat Interfaces

### Server-Sent Events Pattern
For AI chat interfaces that stream responses token-by-token:

```typescript
async function streamDecision(
  request: DecisionRequest,
  onChunk: (chunk: string) => void,
  onComplete?: (response: FullResponse) => void
): Promise<void> {
  const response = await fetch(`${API_URL}/decide/stream`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(request),
  });

  const reader = response.body?.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value, { stream: true });
    // Parse SSE format: "data: content\n\n"
    const events = chunk
      .split('\n')
      .filter((line) => line.startsWith('data: '))
      .map((line) => line.slice(6));

    for (const event of events) {
      if (event === '[DONE]') continue;
      onChunk(event);
    }
  }
}
```

### Streaming Message Component
```tsx
function StreamingMessage({ content, isStreaming }: Props) {
  const [showCursor, setShowCursor] = useState(true);

  // Blinking cursor while streaming
  useEffect(() => {
    if (!isStreaming) return;
    const interval = setInterval(() => setShowCursor((p) => !p), 500);
    return () => clearInterval(interval);
  }, [isStreaming]);

  return (
    <div className="message">
      <p>{content}</p>
      {isStreaming && showCursor && (
        <span className="inline-block w-2 h-5 bg-primary animate-pulse" />
      )}
    </div>
  );
}
```

### State Management for Streaming
```typescript
const [streamingContent, setStreamingContent] = useState('');
const [isStreaming, setIsStreaming] = useState(false);

const handleSend = async (content: string) => {
  setStreamingContent('');
  setIsStreaming(true);

  try {
    await api.decideStream(
      request,
      (chunk) => setStreamingContent((prev) => prev + chunk),
      (response) => {
        // Add final message to history
        setIsStreaming(false);
      }
    );
  } finally {
    setIsStreaming(false);
  }
};
```

---

## zkillboard WebSocket Integration

### Connection Pattern
```typescript
const ZKILLBOARD_WS_URL = 'wss://zkillboard.com/websocket/';

const ws = new WebSocket(ZKILLBOARD_WS_URL);

ws.onopen = () => {
  // Subscribe to killstream
  ws.send(JSON.stringify({
    action: 'sub',
    channel: 'killstream'
  }));
};

ws.onmessage = (event) => {
  // zKillboard sends raw killmail JSON (NOT wrapped in type/payload)
  const killmail = JSON.parse(event.data);
  // killmail has: killmail_id, solar_system_id, killmail_time, victim, attackers, zkb
};
```

### Key Points
- zkillboard WebSocket sends ESI killmail + zkb metadata merged
- NOT wrapped in `{type, payload}` - raw killmail JSON
- Subscribe message: `{action: 'sub', channel: 'killstream'}`
- Kill frequency varies based on EVE Online activity
- During quiet periods (downtime, weekdays), kills can be sparse

### Environment-Based Mock/Real Toggle
```typescript
// .env.local
NEXT_PUBLIC_USE_MOCK_KILLS=true  // Use mock data
NEXT_PUBLIC_USE_MOCK_KILLS=false // Use real zkillboard

// In hook
const envUseMock = process.env.NEXT_PUBLIC_USE_MOCK_KILLS === 'true';
const { useMock = envUseMock } = options;
```

### Reconnection with Exponential Backoff
```typescript
const backoffDelay = Math.min(
  reconnectDelay * Math.pow(2, reconnectAttempts - 1),
  60000  // Cap at 60 seconds
);
setTimeout(connect, backoffDelay);
```

---

*Last updated: 2026-01-30*
