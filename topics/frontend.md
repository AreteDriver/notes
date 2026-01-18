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

*Last updated: 2026-01-18*
