# Higher-Order Component (HOC) Pattern

## Problem Signals
- Shared wrapper logic is repeated across components, causing duplication and drift
- Injected props erode presentational purity and obscure intent at usage sites
- Without a formal abstraction, injected props lack type guarantees and risk collisions

## Core Idea
A Higher-Order Component is a function that takes a component and returns a new component with additional behavior injected at the boundary.

This allows shared logic to be composed structurally, without modifying the wrapped component or polluting its public API.

HOCs preserve declarative usage, enforce type-safe prop injection, and enable behavioral reuse while keeping wrapped components dumb and presentation-focused.

They are most effective for cross-cutting concerns that must be applied consistently across multiple components.

## Reference Details

### Definition
A Higher-Order Component is a function of the form `(Component) -> EnhancedComponent`. The returned component injects additional props or behavior, shields consumers from internal wiring, and preserves the wrapped component's role and intent. TypeScript enforces correct prop injection and prevents consumers from supplying injected props manually.

### When to Use

- Multiple components require the same injected data or callbacks
- Components must remain dumb and presentation-only
- Behavior must be composable at the component boundary
- Declarative usage should remain clean and intention-revealing
- Enforcing prop ownership and injection
- Cross-cutting concerns must be applied uniformly (auth, data, feature flags)
### When to Avoid

- Behavior is localized and unlikely to be reused
- Logic depends heavily on component internals
- Hook-based composition provides clearer ownership
- Wrapper depth becomes excessive and obscures the tree
- Ref forwarding or instance access is required frequently
- Fine-grained control over render timing
### Mechanics

1. **Encapsulation Strategy:** Behavioral logic is isolated at the HOC boundary to prevent leakage into component implementations, preserving separation of concerns
2. **Prop Ownership Model:** The HOC exclusively owns injected props, removing them from the consumer contract via type-level exclusion (`Omit`, generics)
3. **Usage Model Stability:** Enhanced components are consumed identically to standard components, preserving declarative rendering
4. **Role Isolation:** Components retain a single responsibility; rendering based on received props
5. **Static Guarantees:** The type system prevents consumer override of injected props and enforces their availability internally
6. **Composable Abstractions:** Behavioral layers can be stacked in a controlled and predictable manner to build higher-level abstractions

### TypeScript Example

#### Injected Contract (Explicit Boundary)

```tsx
// injected.ts
export interface Injected<TInjected> {
    injected: TInjected;
}
```

Injected props must be:
- explicit
- isolated
- typed
#### Generic HOC Factory
```tsx
// createHOC.tsx
import { ComponentType } from "react";

export type Without<T, K> = Omit<T, keyof K>;

export function createHOC<TInjected>(
    useInjected: () => TInjected
) {
    return function withInjection<P extends Injected<TInjected>>(
        Component: ComponentType<P>
    ) {
        return function HOC(
            props: Without<P, Injected<TInjected>>
        ) {
            /*
                - Side effects, subscriptions, orchestration
                - Cross-cutting concerns live here
                - Wrapped component remains unaware
            */

            const injected = useInjected();

            return (
                <Component
                    {...(props as P)}
                    injected={injected}
                />
            );
        };
    };
}
```
- Injection logic is centralized
- Wrapped components stay deterministic
- No prop drilling
- No container explosion
#### Domain-Agnostic Injection Source

```tsx
// useInjected.ts
export interface InjectionContext {
    // Opaque external capability
}

export function useInjected(): InjectionContext {
    /*
        Source of truth:
        - Context
        - Store
        - Service
        - Cache
        - Environment
    */

    return {};
}
```

#### Pure Presentational Component (Injection-Aware, Logic-Free)

```tsx
// View.tsx
import { Injected } from "./injected";
import { InjectionContext } from "./useInjected";

export type ViewProps = Injected<InjectionContext>;

export function View(props: ViewProps) {
    const { injected } = props;

    /*
        - No knowledge of where data comes from
        - No side effects
        - Only consumes injected capability
    */

    return null;
}
```

#### HOC Assembly

```tsx
// enhance.tsx
import { createHOC } from "./createHOC";
import { useInjected } from "./useInjected";
import { View } from "./View";

const withInjection = createHOC(useInjected);

export const EnhancedView = withInjection(View);
```

#### Consumer Usage Pattern

```tsx
// Parent.tsx
import { EnhancedView } from "./enhance";

export function Parent() {
    /*
        Parent does not:
        - Pass injected props
        - Know about injection logic
        - Manage cross-cutting concerns
    */

    return <EnhancedView />;
}
```

### Scalability Notes

How the HOC pattern behaves at scale:

#### Strengths

- Abstract shared behavior without duplicating logic
- Enforce presentational component purity (no side effects, no data access)
- Inject cross-cutting concerns at the component boundary (auth, data, permissions, analytics)
- Preserve declarative component usage at call sites
- Provide strong compile-time guarantees via generic constraints and injected prop typing
- Enable stable, reusable APIs independent of component internals
#### Costs

- Introduce additional indirection during debugging and error tracing
- Requires explicit ref forwarding (`forwardRef`) when refs must pass through
- Wrapper stacking can reduce readability and increase cognitive load
- Name collisions between injected and consumer props must be explicitly prevented
- Component display names may degrade without manual assignment
#### System-Level Considerations

- Prefer HOCs for cross-component, boundary-level concerns
- Limit HOC depth in performance-critical render paths
- Ensure injected props are stable (memoized) to avoid unnecessary re-renders
- Consider hooks or render props when behavior is instance-specific or highly dynamic
### Related Patterns
- Hooks
- Render Props
- Container-Presentation Components
## Summary
- HOCs inject behavior at the component boundary without modifying wrapped components
- Strong typing prevents consumers from supplying injected props directly
- Overuse can add indirection and make the tree harder to debug
## Next Steps
- Prefer Hooks for instance-level logic reuse
- Consider Render Props when consumers need explicit render control
