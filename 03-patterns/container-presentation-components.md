# Container-Presentation Components Pattern

## Problem Signals
- UI components mix data fetching, state, side effects, and rendering in one unit
- Visual components depend on business logic and become hard to reuse
- Testing UI behavior requires mocking unrelated state and effects

## Core Idea
The Container-Presentation component pattern separates behavioral responsibility from visual responsibility.

Container components own data acquisition, state management, and side effects, while presentation components focus exclusively on rendering based on props.

This separation improves reuse, testability, and long-term maintainability, especially in TypeScript-heavy and large-scale React systems.

## Reference Details

### Definition
A design pattern where container components manage state, effects, and data flow, while presentation components receive data and callbacks via props and remain stateless or minimally stateful.

### When to Use

- The same UI must be reused across multiple data sources or behavioral implementations
- Business logic is expected to change frequently while the UI structure remains stable
- UI components must support isolated unit testing without dependency or side effect initialization
- Clear, explicit, and type-safe boundaries are required between the view layer and the business logic layer
### When to Avoid

- Component scope is intentionally small and localized
- Component responsibilities are stable and not expected to expand
- Extracting logic would introduce indirection without clear reuse or composability benefits
- Additional abstraction would reduce clarity or increase cognitive overhead
- Prop drilling or indirection would increase render frequency or reconciliation cost

### Mechanics

1. The container:
   - Fetches or derives data
- Owns state transitions
- Defines event handlers
2. The presentation component:
    - Receives fully prepared props
- Emits events via callbacks
- Contains no side effects or data access
3. TypeScript enforces the contract between the two layers
   

### TypeScript Example

```tsx
// Container.tsx
import { useCallback, useState } from "react";
import { Presentation } from "./Presentation";

export interface ContainerProps<TInput> {
    input: TInput;
}

export function Container<TInput, TState, TViewModel>(
    props: ContainerProps<TInput>
) {
    /*
        Container owns:
        - State
        - Transitions
        - Side effects
        - Mapping logic
    */

    const [state, setState] = useState<TState | null>(null);

    const computeViewModel = (state: TState | null): TViewModel => {
        // Mapping logic (intentionally opaque)
        return {} as TViewModel;
    };

    const handleAction = useCallback((action: unknown) => {
        /*
            - Interpret action
            - Trigger transitions
            - Perform orchestration
        */
    }, []);

    const viewModel = computeViewModel(state);

    return (
        <Presentation
            viewModel={viewModel}
            onAction={handleAction}
        />
    );
}
```

```tsx
// Parent.tsx
import { Container } from "./Container";

export function Parent() {
    const input = {};

    return (
        <Container input={input} />
    );
}
```

### Scalability Notes

- Containers can be composed, replaced or tested independently
- Presentation components remain stable across refactors
- Works well with:
- Feature-based folder structures
- Domain-driven architectures
- Server-state libraries (React Query, SWR)
- In larger systems, containers often collapse into hooks, preserving the same separation
### Related Patterns
- Hooks
- Higher-Order Components (HOC)
- Render Props
## Summary
- Split behavior (container) from rendering (presentation) to reduce coupling
- Clear boundaries improve testability and reuse across data sources
- Overuse adds indirection, so keep the split purposeful
## Next Steps
- Consider Hooks when behavior can be shared without wrappers
- Compare with HOCs for cross-cutting concerns
