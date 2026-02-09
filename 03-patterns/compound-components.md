# Compound Components Pattern

## Problem Signals
- Prop drilling across related UI elements increases coupling and reduces composability
- Centralized conditional rendering makes layouts rigid and monolithic
- Sibling state sync without a contract causes inconsistent behavior and bloated APIs

## Core Idea

The compound component pattern enables a set of related components to share state and behavior implicitly, while allowing consumers to control structure and layout through composition. Shared logic is centralized behind an internal contract instead of being passed through sibling props.

This pattern balances flexibility with encapsulation in complex, composable UI systems.

## Reference Details

### Definition

A compound component pattern is a set of components designed to work together and share internal state and behavior through a shared coordination layer (often context). The parent component defines the contract and the child components consume it.

### When to Use

- Multiple related subcomponents depend on the same internal state or behavioral logic
- Composition control, layout or ordering of subcomponents is needed
- Shared state is required without introducing global state or excessive prop drilling
- Public API should support declarative, semantic composition patterns
- Responsibilities are logically cohesive but structurally decoupled
### When to Avoid

- Only a single component consumes the shared state or logic
- Component relationships are static, linear, or unlikely to change
- The implicit usage contract increases cognitive load or misuse risk
- Performance constraints make context propagation or re-renders unacceptable
- Simpler patterns (local state, explicit props) provide equivalent clarity
### Mechanics

1. **Centralized State Authority:** A single root component owns all shared behavioral state, acting as the authoritative source of truth.
2. **Implicit State Distribution:** State and actions are distributed internally through a localized coordination layer (e.g. React Context), eliminating sibling prop wiring.
3. **Role-Bound Subcomponents:** Subcomponents are role-specific and derive all behavior exclusively from the shared contract.
4. **Declarative Consumer Composition:** Consumers compose subcomponents declaratively, controlling structure without managing behavior.
5. **Internal Contract Enforcement:** The root enforces a strict internal contract to prevent invalid usage.
6. **Context-Scoped Reconciliation Boundary:** State updates are scoped to the compound components context, ensuring controlled reconciliation.

### TypeScript Example

```tsx
import { createContext, useCallback, useContext, useMemo, useState } from "react";

type CompoundState = {
    activeId: string | null;
};

type CompoundAction = {
    type: "select";
    id: string;
};

type CompoundActions = {
    dispatch: (action: CompoundAction) => void;
};

type CompoundContextValue = {
    state: CompoundState;
    actions: CompoundActions;
};

const CompoundContext = createContext<CompoundContextValue | null>(null);

export function useCompoundContext(): CompoundContextValue {
    const ctx = useContext(CompoundContext);

    if (!ctx) {
        throw new Error(
            "Compound components must be used within a Compound.Root"
        );
    }

    return ctx;
}

export function Root(props: { children: React.ReactNode }) {
    const [state, setState] = useState<CompoundState>({
        activeId: null,
    });

    const dispatch = useCallback((action: CompoundAction) => {
        if (action.type === "select") {
            setState({ activeId: action.id });
        }
    }, []);

    const actions: CompoundActions = useMemo(() => ({ dispatch }), [dispatch]);

    const value = useMemo(
        () => ({ state, actions }),
        [state, actions]
    );

    return (
        <CompoundContext.Provider value={value}>
            {props.children}
        </CompoundContext.Provider>
    );
}

export function Item(props: { id: string; children: React.ReactNode }) {
    const { state } = useCompoundContext();
    const isActive = state.activeId === props.id;

    if (!isActive) return null;
    return <div>{props.children}</div>;
}

export function Trigger(props: { id: string; children: React.ReactNode }) {
    const { actions } = useCompoundContext();
    return (
        <button
            type="button"
            onClick={() => actions.dispatch({ type: "select", id: props.id })}
        >
            {props.children}
        </button>
    );
}

export const Compound = {
    Root,
    Item,
    Trigger,
};
```

```tsx
export function Parent() {
    return (
        <Compound.Root>
            <Compound.Trigger id="a">Show A</Compound.Trigger>
            <Compound.Trigger id="b">Show B</Compound.Trigger>
            <Compound.Item id="a">Panel A</Compound.Item>
            <Compound.Item id="b">Panel B</Compound.Item>
        </Compound.Root>
    );
}
```

### Scalability Notes

#### Strengths

- Elegant APIs
- Flexible composition
- No prop drilling
#### Costs
- Implicit coupling via context
- Harder to statically analyze
- Shared re-render scope
#### System-Level Considerations
- Context boundaries remain local and predictable
- Contracts must be documented clearly to avoid misuse
- Works well for
- Menu
- Tab
- Accordion
- Modals with slots
- In large systems, compound roots often delegate logic to hooks
### Related Patterns
- Slots and Composition
- Render Props
- Hooks
## Summary
- Shared state is centralized in a root while layout stays consumer-controlled
- Context-based wiring removes prop drilling but introduces implicit coupling
- Best for cohesive subcomponents that must coordinate without global state
## Next Steps
- Compare with Slots and Composition for layout-first extensibility
- Consider Render Props when consumers need full render control
