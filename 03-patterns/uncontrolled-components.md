# Uncontrolled Components Pattern

## Problem Signals
- External control patterns like lifted state force unnecessary re-renders and widen reconciliation scope
- Centralized ownership increases coupling and expands component APIs
- Routing ephemeral interaction state through controlled pipelines adds overhead without architectural value

## Core Idea
Uncontrolled components manage their own internal state and behavior, exposing only the outputs or events that matter to their consumers. Parent components cannot directly influence these internal transitions; they can only react to the external signals the component emits.

This pattern reduces configuration complexity for consumers and isolates UI-local concerns, but it also limits cross-component coordination and reduces external orchestration capabilities.

In practice, most components are partially uncontrolled, mixing internal state with controlled inputs, so the distinction reflects an architectural choice rather than a strict classification.

## Reference Details

### Definition
An uncontrolled React component maintains a self-contained state model. It emits events outward but does not accept state directives from its parent. Props act as configuration parameters rather than control channels.
By retaining update authority internally, the component minimizes reconciliation pressure and preserves predictable behavior.

### When to Use

- A bounded public API (config in, events out) is preferred
- Component behavior must remain stable despite parent re-renders
- Logic is domain-specific and gains nothing from external orchestration
- State is rapid, granular, or ephemeral where React's controlled pipeline adds overhead
- Parents only need signals, not continuous control (e.g., "user changed X", "interaction completed")
- External ownership of state would cause broad re-render cascades or expand the reconciliation footprint
- High-frequency, localized updates/interactions (dragging, scrubbing, timelines, sliders, physics-driven motion) demand internal control
### When to Avoid

- Parent logic needs to override internal transitions
- Hydration or SSR requires server-defined initial state
- Multiple components require centralized coordination
- Internal behavior must mirror global state (Redux, Zustand, Server Cache)
- Deterministic, externally reproducible flows are required (controlled wizards, step-based UI)
- State must be externally authorized (synchronization, undo/redo, global consistency, time-travel debugging)
### Mechanics

**1. Internal State Ownership**
   All behavioral state is owned by the component itself, managed via `useRef`, `useState`, or localized state machines.

**2. Local Interaction Processing**
   User interactions are fully processed within the component. Transitional and intermediate values are not propagated to parent layers.

**3. Output-Only Emission**
   The component emits discrete output signals only after internal state transitions complete, enabling parents to observe outcomes without influencing behavior.

**4. Static Configuration Props**
   External inputs are restricted to non-behavioral configuration props (e.g., `disabled`, `readOnly`, mode flags) that do not alter the internal control flow.

**5. Transition Continuity Across Renders**
   Internal state transitions persist across parent-driven re-renders, ensuring ongoing interactions are never reset or interrupted.

**6. Shallow Reconciliation Boundary**
   The component maintains a minimal reconciliation surface, supporting micro-rendering and preventing unnecessary render propagation up the component tree.

### TypeScript Example

```tsx
// UncontrolledComponent.tsx
import { useRef, useState, useCallback } from "react";

export interface UncontrolledEvents<TOutput> {
    onChange?: (value: TOutput) => void;
    onCommit?: (value: TOutput) => void;
    onReset?: () => void;
}

export interface UncontrolledProps<TOutput>
    extends UncontrolledEvents<TOutput> {
    disabled?: boolean;
    readOnly?: boolean;
}

export function UncontrolledComponent<TInternal, TOutput>(
    props: UncontrolledProps<TOutput>
) {
    const internalRef = useRef<TInternal | null>(null);
    const [state, setState] = useState<TInternal | null>(null);

    const internalTransition = useCallback((next: TInternal) => {
        internalRef.current = next;
        setState(next);
    }, []);

    const emitChange = useCallback(
        (value: TOutput) => props.onChange?.(value),
        [props.onChange]
    );

    const emitCommit = useCallback(
        (value: TOutput) => props.onCommit?.(value),
        [props.onCommit]
    );

    const emitReset = useCallback(() => props.onReset?.(), [props.onReset]);

    // Component UI driven entirely by internal state (omitted intentionally)
    return null;
}
```

```tsx
// Parent.tsx
import { UncontrolledComponent } from "./UncontrolledComponent";

export function Parent() {
    const onChange = (output: unknown) => {};
    const onCommit = (output: unknown) => {};
    const onReset = () => {};

    return (
        <UncontrolledComponent
            onChange={onChange}
            onCommit={onCommit}
            onReset={onReset}
        />
    );
}
```

### Scalability Notes

How the uncontrolled pattern behaves at scale.

#### Strengths

- Small, stable API surfaces
- Reduced reconciliation cost
- Predictable performance characteristics
- Strong encapsulation boundaries
#### Weaknesses

- Harder to orchestrate across components
- Limited internal state visibility during debugging
- Testing relies primarily on output events
- Risk of logic drifting from higher-level architectural rules
#### System-Level Considerations

- Avoid blanket adoption across the entire UI
- Use in interaction-heavy islands within large applications
- Combine with Controllers or Facades when coordination is required without sacrificing isolation
### Related Patterns
- Controlled Components
- Compound Components
- Container-Presentation Components
## Summary
- Internal state ownership reduces reconciliation scope and API surface area
- Output-only events limit orchestration but keep behavior stable and local
- Best for interaction-heavy islands with ephemeral or high-frequency state
## Next Steps
- Compare with Controlled Components when external orchestration is required
- Use Compound Components when coordinated subcomponents need shared state
