# Render Props Pattern

## Problem Signals
- Reuse via props or inheritance grows APIs and creates fragile, coupled contracts
- Behavior abstractions that hide markup control reduce composability
- Sharing complex stateful behavior across UIs increases coordination overhead

## Core Idea
The Render Props pattern externalizes rendering responsibility while retaining behavioral ownership. A component encapsulates state and logic, then delegates how that state is rendered to its consumer via a function prop.

This enables fine-grained composition without inheritance, preserves local state ownership, and allows consumers to project custom UI while reusing behavior.

In modern React, Render Props function as a low-level composition primitive: more explicit than hooks, more flexible than configuration props, but more verbose and performance-sensitive.

## Reference Details

### Definition
A Render Props component exposes its internal state and actions by invoking a function provided as a prop. The function returns React nodes, giving the consumer full control over rendering while the component retains behavioral authority.

The pattern separates behavioral reuse from structural reuse without expanding the component's API surface.

### When to Use

- Behavioral logic must be shared across heterogeneous UIs
- Consumers require full control over layout and markup
- The abstraction represents a process, not a visual component
- Configuration via static props would lead to excessive branching
- Hooks alone are insufficient due to conditional usage, lifecycle coupling, or cross-cutting concerns
- Explicit composition boundaries are needed at the call site
- The behavior must remain self-contained but visually polymorphic
### When to Avoid

- The abstraction is purely logical and can be expressed as a hook
- Render frequency is high and render cost is non-trivial
- The component tree becomes deeply nested with multiple render layers
- Simpler composition patterns (slots, children, compound components) suffice
- The render function becomes an implicit API with unstable contracts
### Mechanics
1. **Internal Ownership:** State and effects are private to the component
2. **Render via Function:** UI is produced by a consumer-provided render function
3. **Minimal Contract:** Only essential state, derived data, and actions are exposed
4. **Layout Freedom:** Consumers fully control structure and styling
5. **Reconciliation Trade-off:** Function boundaries require careful memoization to avoid excess renders

### TypeScript Example

#### State Model

```ts
export type RenderState<T> = {
    value: T;
};
```

Guarantees:
- No optional fields
- No hidden transitions
#### Render Contract
```ts
export interface RenderActions<T> {
    set: (next: T) => void;
    reset: () => void;
}

export type RenderContext<T> = RenderState<T> & RenderActions<T>;
```
This contract is intentionally small, stable across renders and owned entirely by the controller.

#### Component Contract

```ts
import { ReactNode } from "react";

export interface RenderControllerProps<T> {
    initial: T;
    children: (context: RenderContext<T>) => ReactNode;
}
```
Key signal here is that `children` is not content; it is a render delegate.

#### Core Implementation
```ts
import { useState, useCallback } from "react";

export function RenderController<T>({
    initial,
    children,
}: RenderControllerProps<T>) {
    const [value, setValue] = useState<T>(initial);

    const set = useCallback((next: T) => {
        setValue(next);
    }, []);

    const reset = useCallback(() => {
        setValue(initial);
    }, [initial]);

    return children({
        value,
        set,
        reset,
    });
}
```
- Behavioral ownership is fully internal
- Consumers cannot intercept transitions
- No external synchronization hooks are exposed
#### Usage
```tsx
<RenderController initial={0}>
    {({ value, set, reset }) => (
        <>
            <span>{value}</span>
            <button onClick={() => set(value + 1)}>+</button>
            <button onClick={reset}>Reset</button>
        </>
    )}
</RenderController>
```
The consumer controls rendering.
The controller controls state and behavior.

### Scalability Notes

#### Strengths

- Maximum compositional flexibility
- Clear separation of behavior and presentation
- Explicit usage sites improve traceability
- Avoids inheritance and deep prop configuration trees
#### Costs

- Verbose syntax compared to hooks
- Higher render cost due to function invocation
- Harder to memoize effectively
- Contracts are implicit and easy to destabilize
- Can degrade readability with deep nesting
#### System-Level Considerations

- Prefer hooks for pure logic reuse when possible
- Use for behavior tightly coupled to rendering cadence
- Establish strict typing contracts to prevent API drift
- Combine with memoization (`useMemo`, `memo`)
- Avoid chaining multiple Render Props layers without clear justification
### Related Patterns
- Hooks
- Compound Components
- Slots and Composition
- Higher-Order Components (HOC)
## Summary
- Render Props keep behavior internal while giving consumers full render control
- Explicit render functions make composition flexible but more verbose
- Performance and readability require careful memoization and restraint
## Next Steps
- Use Hooks when render control is not required
- Compare with Compound Components for slot-like composition
