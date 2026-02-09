# Renderless Component Pattern

## Problem Signals
- Multiple UIs need the same behavior and accessibility logic without shared markup
- Hooks alone lack a lifecycle boundary for orchestration or imperative APIs
- Presentational components reimplement interaction state machines, causing drift

## Core Idea
A renderless component owns state and behavior but renders no UI. It exposes a minimal, typed contract to consumers (via render props or context), allowing them to render any structure while reusing the same logic.

This pattern centralizes behavioral complexity and accessibility rules without constraining layout or styling.

## Reference Details

### Definition
A renderless component encapsulates internal state, side effects, and event wiring, then delegates rendering to its consumer. The component itself returns the consumer's render output or `null`, and does not emit its own DOM.

### When to Use
- Behavior must be reused across multiple layouts or design variants
- Accessibility logic (keyboard, focus, ARIA) should be enforced consistently
- The behavior requires a stable lifecycle boundary (subscriptions, timers, imperative APIs)
- You need a reusable abstraction that can be consumed without adopting hooks directly
### When to Avoid
- The behavior is simple and can be expressed as a hook without extra component layers
- Only a single UI consumes the behavior and abstraction adds indirection
- Render props cause unnecessary re-renders in tight performance paths
- The component must output its own markup (use a standard component instead)
### Mechanics
1. **Internal ownership:** State and side effects live inside the renderless component.
2. **Render delegate:** The component calls a `children` function with its contract.
3. **Stable API:** Expose minimal, typed data and actions to avoid leaking internals.
4. **Prop getters:** Provide helpers to wire accessibility and events consistently.
5. **No DOM output:** The component returns the render delegate output directly.

### TypeScript Example

```tsx
import { useCallback, useMemo, useState } from "react";

export type ToggleRenderProps = {
    isOn: boolean;
    toggle: () => void;
    getButtonProps: (
        props?: React.ButtonHTMLAttributes<HTMLButtonElement>
    ) => React.ButtonHTMLAttributes<HTMLButtonElement>;
};

export interface ToggleControllerProps {
    initialOn?: boolean;
    children: (props: ToggleRenderProps) => React.ReactNode;
}

export function ToggleController({
    initialOn = false,
    children,
}: ToggleControllerProps) {
    const [isOn, setIsOn] = useState(initialOn);

    const toggle = useCallback(() => {
        setIsOn((prev) => !prev);
    }, []);

    const getButtonProps = useCallback(
        (
            props: React.ButtonHTMLAttributes<HTMLButtonElement> = {}
        ) => ({
            type: "button",
            "aria-pressed": isOn,
            onClick: (event) => {
                props.onClick?.(event);
                if (!event.defaultPrevented) toggle();
            },
            ...props,
        }),
        [isOn, toggle]
    );

    const api = useMemo(
        () => ({ isOn, toggle, getButtonProps }),
        [isOn, toggle, getButtonProps]
    );

    return <>{children(api)}</>;
}
```

```tsx
export function Example() {
    return (
        <ToggleController>
            {({ isOn, getButtonProps }) => (
                <button {...getButtonProps()}>
                    {isOn ? "On" : "Off"}
                </button>
            )}
        </ToggleController>
    );
}
```

### Scalability Notes

#### Strengths
- Behavior is centralized while layout stays flexible
- Accessibility and event wiring remain consistent across UIs
- Consumers can render any structure without new abstractions
#### Costs
- Render props add indirection and may increase render frequency
- Debugging requires tracing through a delegate boundary
- API drift can occur if the contract grows without discipline
#### System-Level Considerations
- Keep renderless components small and domain-specific
- Prefer hooks for pure logic reuse when no lifecycle boundary is needed
- Document prop getter behavior and merge semantics explicitly
### Related Patterns
- Render Props
- Hooks
- Compound Components
- Slots and Composition
## Summary
- Renderless components reuse behavior without dictating UI structure
- They provide a stable lifecycle boundary and consistent accessibility wiring
- Overuse can add complexity and render indirection
## Next Steps
- Compare with Hooks for logic-only reuse
- Pair with Compound Components when subcomponents need shared state
