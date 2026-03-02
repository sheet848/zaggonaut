---
title: React Performance Optimization Techniques
slug: react-performance
description: 10 React performance techniques that cut render times from using memoization, virtualization, and bundle optimization.
longDescription: 10 React performance techniques that cut render times from using memoization, virtualization, and bundle optimization.
cardImage: "https://zaggonaut.dev/michael-dam-unsplash.webp"
tags: ["react", "performance"]
readTime: 10
featured: true
timestamp: 2026-03-02T02:39:03+00:00
---

React apps start snappy but bloat with features: 100+ components, lists, heavy renders. Poor performance loses users and, Google penalizes slow sites. We need to optimize proactively.

## Measure First: Tools Setup
Profile before optimizing.

- **React DevTools Profiler:** Record renders, spot offenders.
- **Chrome Lighthouse:** Core Web Vitals (LCP/CLS/FID).
- **WhyDidYouRender:** Detect unnecessary re-renders.


## Target: <100ms render, 60fps
### Technique: Memoization Trio
**React.memo:** Skip pure components.
```tsx
const ExpensiveChild = React.memo(({ data }: { data: number[] }) => {
  // Heavy computation
  const sum = data.reduce((a, b) => a + b, 0);
  return <div>Sum: {sum}</div>;
});

```

**useCallback:** Stable functions.
```tsx
const Parent = () => {
  const [count, setCount] = useState(0);
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []); // Empty deps for stability

  return <ExpensiveChild onClick={handleClick} />;
};

```

**useMemo:** Cache expensive values.
```tsx
const FilteredList = ({ items }: { items: string[] }) => {
  const filtered = useMemo(() => 
    items.filter(item => item.includes('perf')), 
    [items]
  );
  return <ul>{filtered.map(i => <li key={i}>{i}</li>)}</ul>;
};

```

## Impact: 40-60% render reduction in lists
### Technique: List Virtu & Keys + Code Splitting
**Proper Keys:** `key={uniqueId}` prevents full re-mounts.

**React Window (Virtualization):** Render 1000+ items as 20.
```tsx
import { FixedSizeList as List } from 'react-window';

const VirtualList = ({ items }: { items: string[] }) => (
  <List height={500} itemCount={items.length} itemSize={35}>
    {({ index, style }) => (
      <div style={style}>{items[index]}</div>
    )}
  </List>
);

```

**Code Splitting:** Lazy load routes/features.
```tsx
const LazyDashboard = lazy(() => import('./Dashboard'));

<Suspense fallback={<Spinner />}>
  <LazyDashboard />
</Suspense>

```

## Vite bundles: **150KB** gzipped vs **2MB** unsplit
### Technique: State + Bundle Optimization
**Split State:** Local > Context > Redux. Use `useReducer` for complex local.

**Analyze Bundle:** `vite-bundle-visualizer`—tree-shake lodash, moment.

```bash
npm i -D rollup-plugin-visualizer
# vite.config.ts: plugins: [visualizer()]

```

Remove: `lodash.get` → native optional chaining.

### Technique: Custom Optimizers
**useWhyDidIRender:** Dev-only hook.
```tsx
// Add to components during dev
useWhyDidIRender(Parent, { trackAllPureComponents: true });

```

**Throttle/Resizing:**
```tsx
const useWindowSize = () => {
  const [size, setSize] = useState({ width: 0, height: 0 });
  useEffect(() => {
    const handleResize = throttle(() => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    }, 100);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  return size;
};

```

### Technique: Production Checklist
| Technique | Tool/Metric | FPS Gain |
|*************|**************|*************|
| Memoization | React Profiler | 50% |
| Virtualization | react-window | 90% on lists |
| Lazy Loading | Suspense | 70% init load |
| Bundle Split | Vite Analyzer | 80% size |
| Keys Fixed | Console warnings | 30% |

## Checklist:
- Run Lighthouse >90 score.
- `<StrictMode>` in dev.
- `React.Profiler` in prod builds.
- Server-side rendering for LCP.

In PhotoSnap, virtualization + memo = buttery scrolls on 10k images.

## Benchmark Your Gains
Before/after: Use `performance.mark()`.
```tsx
performance.mark('render-start');
ReactDOM.render(<App />, root);
performance.mark('render-end');
console.log(`Render: ${performance.measure('render', 'render-start', 'render-end').duration}ms`);

```

## Action Items
1. Profile your app today.
2. Memo 3 components.
3. Virtualize any list >50 items.
