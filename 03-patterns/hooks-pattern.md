# Hooks Pattern (Logic Extraction Pattern)

## Problem Signal
- Component logic becomes tightly coupled to rendering concerns, increasing cognitive load and reducing reusability
- Behavioral logic duplicated across components leads to divergence, subtle bugs, and maintenance overhead
- Shared stateful behavior implemented via inheritance, HOCs, or utilities introduces indirection, prop pollution, or unstable abstractions
- Complex side effects and orchestration logic embedded in components reduce testability and obscure intent

## Core Idea
The Hooks Pattern extracts stateful behavior and side effects into reusable, composable functions that operate independently of rendering.

Custom hooks encapsulate logic, not UI, allwing components to remain declarative while behavior evolves independently.

This pattern leverages React's functional execution model to enable controlled reuse, local reasoning, and explicit dependency boundaries.

In modern React, hooks are the primary abstraction for component behavior composition.

## Evidence & Implementation

### Definition
A custom hook is a function that uses React hooks internally to encapsulate stateful logic and side effects, exposing a stable, typed interface to consuming component.

Hooks represent behavioral units, not components, and must remain pure with respect to rendering.

### When to Use

- Behavioral logic is reused across multiple components
- Component bodies exceed simple orchestration responsibilities
- Side effects require isolation and explicit dependency management
- State transitions benefit from localized reasoning and unit testability
- Logic must be composable without introducing additional render layers
- TypeScript inference and constraints clarify behavioral contracts
- Reuse is required without modifying component hierarchies

### Avoid When

- Logic is trivial and tightly bounded to a single component's UI
- The abstraction would expose implementaion details rather than encapsulate them
- Direct access to DOM structure or layout-specific coupling is required
- Execution order or conditional invocation cannot be statically guaranteed
- Global orchestration or cross-tree coordination is needed
  - Prefer centralized stores, controllers, or services
- The hook would aggregate unrelated concerns ("god hook" anti-pattern)

### Hooks "Must" Rules
1. Encapsulate state and side effects outside components
2. Not render UI or depend on JSX structure
3. Receive all dependencies via parameters and declare them explicitly
4. Expose a minimal, typed public interface
5. Called unconditionally at the top level

### Implementation

#### State Model

```ts
export type AsyncState<T, E = unknown> =
    | { status: "idle" }
    | { status: "pending" }
    | { status: "success"; data: T }
    | { status: "error"; error: E };
```

Guarantees:
- No consumer guesswork
- Closed, types, semantic state model

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

    const [state, setState] = useState<AsyncState<T, E>>({
        status: "idle",
    });

    const run = useCallback(async () => {
        setState({ status: "pending" });

        try {
            const data = await task();
            if (!mountedRef.current) return;

            setState({ status: "success", data });
            options?.onSuccess?.(data);
        } catch (err) {
            if (!mountedRef.current) return;

            setState({ status: "error", error: err as E });
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
        state,
        run,
    };
}
```

#### Usage

```tsx
const { state, run } = useAsync(fetchData, { immediate: true });

if (state.status === "pending") return null;
if (state.status === "error") return null;

return <button onClick={run}>Trigger</button>;
```

### Scalability Notes

#### Strengths

- High reuse without architectural coupling
- Clear separation of behavior and presentation
- Strong TypeScipt inference and constraint modeling
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

### Related Concepts
TBD