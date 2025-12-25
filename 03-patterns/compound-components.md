# Compound Components Pattern

## Problem Signal
- Coordinating related UI elements through explicit prop wiring forces prop drilling across intermediate components, increasing coupling and reducing composability
- Centralizing conditional rendering logic in a single parent component collapses layout flexibility and results in rigid, monolithic component structures
- Sharing state across sibling components without an internal coordination mechanism leads to duplicated logic, inconsistent behavrior, or ad-hoc synchronization hacks
- Exposing internal orchestration details through public props expands component APIs unnecessarily and leaks implementation concerns to consumers

## Core Idea

The compound component pattern enables a set of related components to share state and behavior implicitly, while allwing consumers to cntrol structures and layout through composition.

Instead of passsing props explicitly between siblings, shared logic is centralized and exposed through a controlled internal contract.

This pattern balances flexibility with encapsulation in complex, composable UI systems.

## Evidence & Implementation

### Definition

A design pattern where multiple components:
- designed to work together
- share internal state and behavior
- communicate implicitly via a shared context or coordination mechanism
The parent component defines the contract; child components consume it.

### When to Use

- Multiple related subcomponents depend on the same internal state or behavioral logc
- Composition control, layout or ordering of subcomponents is needed
- Shared state is required without introducing global state or excessive prop drilling
- Public API should support declarative, semantic composition patterns
- Responsibilities are logically cohesive but structurally decoupled

### Avoid When

- Only a single component consumes the shared state or logic
- Component relationships are static, linear, or unlikely to change
- The implicit usage contract increases cognitive load or misuse risk
- Performance constraints make context propagation or re-renders unacceptable
- Simpler patterns (local state, explicit props) provide equivalent clarity

### Mechanics

1. **Centralized State Authority:**
   A single root component owns all shared behavioral state, acting as the authoritative source of truth

2. Implicit State Distribution:
    State and actions are distributed internally through a localized coordination layer (e.g. React Context), eliminating sibling prop wiring

3. Role-Bound Subcomponents:
   Subcomponents are role-specific and derive all behavior exclusively from the shared contract

4. Declarative Consumer Composition:
   Consumers compose subcomponents declaratively, controlling structure without managing behavior

5. Internal Contract Enforcement:
    The root enforces a strict internal contract to prevent invalid usage

6. Context-Scoped Reconciliation Boundary:
   State updates are scoped to the compound components context, ensuring controlled reconciliation

### Implementation

#### Core Types

```tsx
// types.ts
export interface CompoundState {
    // Internal shared state (opaque to consumers)
}

export interface CompoundActions {
    // Internal transitions
    dispatch?: (action: unknown) => void;
}

export interface CompoundContextValue {
    state: CompoundState;
    actions: CompoundActions;
}

```

#### Context Definition

```tsx
// context.ts
import { createContext, useContext } from "react";
import { CompoundContextValue } from "./types";

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

export { CompoundContext };
```

#### Root Component (State Owner & Orchestrator)

```tsx
// Root.tsx
import { useCallback, useMemo, useState } from "react";
import { CompoundContext } from "./context";
import { CompoundState, CompoundActions } from "./types";

export interface CompoundRootProps {
    children: React.ReactNode;
}

export function Root(props: CompoundRootProps) {
    /*
        Root owns:
        - Shared state
        - Transitions
        - Coordination logic
        - Context boundary
    */

    const [state, setState] = useState<CompoundState>({});

    const dispatch = useCallback((action: unknown) => {
        /*
            Interpret and apply transitions.
            Details intentionally hidden.
        */
    }, []);

    const actions: CompoundActions = useMemo(
        () => ({ dispatch }),
        [dispatch]
    );

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
```

#### Child Components (Implicitly Wired)

Each child component:
- Accesses shared state via context
- Does not receive coordination props
- Remains unaware of siblings

```tsx
// Item.tsx
import { useCompoundContext } from "./context";

export interface ItemProps {
    id: string;
}

export function Item(props: ItemProps) {
    const { state, actions } = useCompoundContext();

    /*
        - Reads from shared state
        - Emits actions
        - No prop drilling
        - No awareness of other children
    */

    return null;
}
```

#### Another Child

```tsx
// Trigger.tsx
import { useCompoundContext } from "./context";

export interface TriggerProps {}

export function Trigger(props: TriggerProps) {
    const { actions } = useCompoundContext();

    /*
        Emits coordination signals into the shared system.
    */

    return null;
}
```

#### Public API Assembly

```tsx
// index.ts
import { Root } from "./Root";
import { Item } from "./Item";
import { Trigger } from "./Trigger";

export const Compound = {
    Root,
    Item,
    Trigger,
};
```

#### Consumer Usage Pattern

```tsx
// Parent.tsx
import { Compound } from "./compound";

export function Parent() {
    return (
        <Compound.Root>
            <Compound.Trigger />
            <Compound.Item id="a" />
            <Compound.Item id="b" />
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

### Related Concepts
TBD