# Hooks Pattern (Logic Extraction Pattern)

## Problem Signals
- Logic and side effects are coupled to rendering, reducing reuse and testability
- Behavioral logic is duplicated across components, leading to divergence and bugs
- Inheritance, HOCs, or utilities add indirection, prop pollution, or unstable abstractions

## Core Idea
The Hooks Pattern extracts stateful behavior and side effects into reusable, composable functions that operate independently of rendering. Custom hooks encapsulate logic, not UI, allowing components to remain declarative while behavior evolves independently.

This pattern leverages React's functional execution model to enable controlled reuse, local reasoning, and explicit dependency boundaries.

## Reference Details

### Definition
A custom hook is a function that uses React hooks internally to encapsulate stateful logic and side effects, exposing a stable, typed interface to consuming components.

Hooks represent behavioral units, not components, and must remain pure with respect to rendering.

### When to Use

- Behavioral logic is reused across multiple components
- Component bodies exceed simple orchestration responsibilities
- Side effects require isolation and explicit dependency management
- State transitions benefit from localized reasoning and unit testability
- Logic must be composable without introducing additional render layers
- TypeScript inference and constraints clarify behavioral contracts
- Reuse is required without modifying component hierarchies
### When to Avoid

- Logic is trivial and tightly bounded to a single component's UI
- The abstraction would expose implementation details rather than encapsulate them
- Direct access to DOM structure or layout-specific coupling is required
- Execution order or conditional invocation cannot be statically guaranteed
- Global orchestration or cross-tree coordination is needed
- Prefer centralized stores, controllers, or services
- The hook would aggregate unrelated concerns ("god hook" anti-pattern)
### Mechanics
1. **Encapsulated State:** Own state and side effects inside the hook, not the component, to keep behavior modular.
2. **UI-Agnostic Logic:** Do not render UI or depend on JSX structure; return data and actions only.
3. **Explicit Dependencies:** Receive dependencies via parameters and declare them in hook dependency arrays.
4. **Minimal Contract:** Expose a small, typed API to prevent leaking internal details.
5. **Stable Invocation:** Call hooks unconditionally at the top level to preserve React's execution guarantees.

### TypeScript Example

#### Hook Contract & Implementation
```ts
import { useCallback, useEffect, useRef, useState } from "react";

export function useAsync<T, E = unknown>(
    task: () => Promise<T>,
    options?: {
        immediate?: boolean;
        onSuccess?: (data: T) => void;
        onError?: (error: E) => void;
    }
) {
    const mountedRef = useRef(true);

    const [status, setStatus] = useState<"idle" | "pending" | "success" | "error">(
        "idle"
    );
    const [data, setData] = useState<T | null>(null);
    const [error, setError] = useState<E | null>(null);

    const run = useCallback(async () => {
        setStatus("pending");

        try {
            const data = await task();
            if (!mountedRef.current) return;

            setData(data);
            setError(null);
            setStatus("success");
            options?.onSuccess?.(data);
        } catch (err) {
            if (!mountedRef.current) return;

            setError(err as E);
            setStatus("error");
            options?.onError?.(err as E);
        }
    }, [task, options]);

    useEffect(() => {
        if (options?.immediate) run();
        return () => {
            mountedRef.current = false;
        };
    }, [run, options?.immediate]);

    return {
        status,
        data,
        error,
        run,
    };
}
```

#### Usage

```tsx
const { status, run } = useAsync(fetchData, { immediate: true });

if (status === "pending") return null;
if (status === "error") return null;

return <button onClick={run}>Trigger</button>;
```

### Scalability Notes

#### Strengths

- High reuse without architectural coupling
- Clear separation of behavior and presentation
- Strong TypeScript inference and constraint modeling
- Testable logic without rendering
- Flat component trees
#### Costs

- Poorly designed hooks can leak internal assumptions
- Over-composition leads to opaque execution paths
- Debugging requires tracing through function layers
- Mismanaged dependencies can cause subtle bugs
#### System-Level Considerations

- Treat hooks as internal modules, not utilities
- Co-locate hooks near the domain they serve
- Avoid cross-domain hooks without explicit ownership
- Enforce naming conventions (`useX`) and responsibility limits
- Prefer small, focused hooks over monolithic abstractions
### Related Patterns
- Container-Presentation Components
- Render Props
- Higher-Order Components (HOC)
## Summary
- Hooks extract reusable behavior without adding render layers
- Clear dependency and typing contracts keep logic predictable
- Over-composition can create opaque execution paths
## Next Steps
- Use Container-Presentation Components when UI and behavior must be separated
- Consider Render Props for behavior tightly coupled to render control
