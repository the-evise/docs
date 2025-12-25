# Uncontrolled Components Pattern

## Problem Signal
- External control patterns—such as lifted or prop-driven state—force unnecessary re-renders across the component tree, increasing reconciliation scope and reducing performance.
- Centralizing state ownership increases coupling, expands component APIs, and introduces avoidable architectural complexity, making the system more rigid and harder to refactor.
- Routing ephemeral interactions and transient UI state through React’s controlled state pipeline adds unnecessary render overhead and coordination cost without providing meaningful architectural value.

## Core Idea
Uncontrolled components manage their own internal state and behavior, exposing only the outputs or events that matter to their consumers. Parent components cannot directly influence these internal transitions; they can only react to the external signals the component emits.
This pattern reduces configuration complexity for consumers and isolates UI-local concerns, but it also limits cross-component coordination and reduces external orchestration capabilities.
In practice, most components are partially uncontrolled—mixing internal state with controlled inputs—so the distinction reflects an architectural choice rather than a strict classification.

## Evidence & Implementation

### Definition
An uncontrolled React component maintains a self-contained state model. It emits events outward but does not accept state directives from its parent. Props act as configuration parameters rather than control channels.
By retaining update authority internally, the component minimizes reconciliation pressure and preserves predictable behavior.

### When to Use

- A bounded public API (config in, events out) is preferred
- Component behavior must remain stable despite parent re-renders
- Logic is domain-specific and gains nothing from external orchestration
- State is rapid, granular, or ephemeral where React's controlled pipeline adds overhead
- Parents only need signals, not continuous control (e.g., “user changed X”, “interaction completed”)
- External ownership of state would cause broad re-render cascades or expand the reconciliation footprint
- High-frequency, localized updates/interactions (dragging, scrubbing, timelines, sliders, physics-driven motion) demand internal control

### Avoid When

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
   User Interactions are fully processed within the component. Transitional and intermediate values are not propagated to parent layers.

**3. Output-Only Emission**
   The component emits discrete output signals only after internal state transitions compelete, enabling parents to observe outcomes without influencing behavior.

**4. Static Configuration Props**
   External inputs are restricted to non-behavioral configuration props (e.g., `disabled`, `readOnly`, mode flags) that do not alter the internal control flow.

**5. Transition Continuity Across Renders**
   Internal state transitions persist across parent-driven re-renders, ensuring ongoing interactions are never reset or interrupted.

**6. Shallow Reconciliation Boundary**
   The component maintains a minimal reconciliation surface, supporting micro-rendering and preventing unnecessary render propagation up the component tree.

### Implementation

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

### Related Concepts

TBD
