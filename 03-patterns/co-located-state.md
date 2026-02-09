# Co-located State Pattern

## Problem Signals
- State is lifted too high, widening render scope and causing cascades
- Parents become coordination hubs and pass-through props accumulate
- Ownership is far from interactions, making reasoning and debugging harder

## Core Idea
The Co-located State pattern keeps state as close as possible to where it is used. Ownership stays within the smallest component or subtree that needs the data, and only the necessary signals are surfaced upward.

This reduces unnecessary render scope and keeps state intent obvious.

## Reference Details

### Definition
Co-located state is state owned by the component or subtree that directly consumes it. It is not lifted unless multiple siblings or parents require direct control.

### When to Use
- State only affects a local UI region or interaction
- The data is ephemeral or high-frequency (dragging, hover, input)
- You want to minimize prop drilling and reconciliation footprint
- A parent only needs output events, not continuous control
### When to Avoid
- Multiple distant components must coordinate on the same state
- Global persistence or time-travel debugging is required
- The state must be controlled externally for deterministic flows
- The component is part of a shared orchestration boundary
### Mechanics
1. **Local ownership:** State lives where it is read and written.
2. **Event surface:** Parents receive outputs rather than state control.
3. **Bounded scope:** Updates are limited to the smallest subtree.
4. **Escalation rule:** Lift state only when multiple consumers truly need it.

### TypeScript Example

```tsx
import { useCallback, useState } from "react";

export function QuantityStepper(props: {
    onCommit?: (value: number) => void;
}) {
    const [value, setValue] = useState(1);

    const increment = useCallback(() => {
        setValue((prev) => prev + 1);
    }, []);

    const decrement = useCallback(() => {
        setValue((prev) => Math.max(1, prev - 1));
    }, []);

    const commit = useCallback(() => {
        props.onCommit?.(value);
    }, [props.onCommit, value]);

    return (
        <div>
            <button onClick={decrement}>-</button>
            <span>{value}</span>
            <button onClick={increment}>+</button>
            <button onClick={commit}>Apply</button>
        </div>
    );
}
```

```tsx
export function CartRow() {
    return (
        <div>
            <span>Item A</span>
            <QuantityStepper onCommit={(v) => console.log(v)} />
        </div>
    );
}
```

### Scalability Notes

#### Strengths
- Minimal render scope and fewer prop chains
- Clear ownership near the interaction point
- Reduced coupling between parent and child components
#### Costs
- Harder to coordinate when multiple consumers need the same state
- State can become duplicated across similar components
- Requires discipline to know when to lift
#### System-Level Considerations
- Use output events to connect local state to higher-level workflows
- Lift state only after a clear coordination requirement emerges
- Prefer local state for interaction-heavy islands
### Related Patterns
- Uncontrolled Components
- Controlled Components
- Compound Components
- Event-Action Pattern
## Summary
- Keep state close to its usage to minimize render scope
- Lift only when coordination demands it
- Output events are often enough for parent workflows
## Next Steps
- Compare with Controlled Components for externally orchestrated flows
- Use Event-Action when multiple components must map events to shared intent
