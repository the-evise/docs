# Higher Order Component—HOC Pattern

## Problem Signal
- Repeating identical wrapper logic across multiple components leads to structural duplication, higher maintenance cost, and inconsistent behavior over time.
- Injecting shared data or callbacks directly through props forces presentational components to accept responsibilities outside their rendering scope, eroding their role purity. 
- Manually composing behavioral wrappers at usage sites increases verbosity and obscures intent, weakening declarative readability at the component boundary.
- Without a formal abstraction, injected props lack compiler-enforced guarantees, requiring consumers to manually satisfy contracts that should be implicit and type-safe.
- Scaling cross-cutting concerns through ad-hoc composition expands component APIs unnecessarily and increases the risk of prop collisions and misuse.

## Core Idea
A Higher-Order Component is a function that takes a component and returns a new component with additional behavior injected at the boundary.

This allows shared logic to be composed structurally, without modifying the wrapped component or polluting its public API.

HOCs preserve declaretive usage, enforce type-safe prop injection, and enable behavioral reuse while keeping wrapped components dumb annd presentation-focused.

They are most effective for cross-cutting concerns that must be applied consistently acroos multiple components.

## Evidence & Implementation

### Definition
A Higher-Order Component is a function of the form:
```text
(Component) -> EnhancedComponent
```
The returned component:
- Injects additional props or behavior
- Sheilds consumers from internal wiring
- Preserves the wrapped component's role and intent
TypeScript is used to:
- Enforce correct prop injection
- Prevent consumers from supplying injected props manually

### When to Use

- Multiple components require the same injected data or callbacks
- Components must remain dumb and presentation-only
- Behavior must be composable at the component boundary
- Declarative usage should remain clean and intention-revealing
- Enforcing prop ownership and injection
- Cross-cutting concerns must be applied uniformly (auth, data, feature flags)

### Avoid When

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

### Implementation

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

#### HOC Asssembly

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

How the Uncontrolled pattern behaves at scale:

#### Strengths

- Abstraction shared behavior without duplicating logic
- Enforce presentational component purity (no side effects, no data access)
- Inject cross-cutting concerns at the component boundary (auth, data, permissions, analytics)
- Preserve declarative component usage at call sites
- Provide strong compile-time guarantees via generic constraints and injected prop typing
- Enable stable, reusable APIs independent of component internals

#### Costs

- Introduce additional indirection during debugging and error tracing
- Requires explicit ref forwarding `forwardRef` when refs must pass through
- Wrapper stacking can reduce readablity and increase cognitive load
- Name collisions between injected and consumer props must be explicitly prevented
- Component display names may degrade without manual assignment

#### System-Level Considerations

- Prefer HOCs for cross-component, boundary-level concerns
- Limit HOC depth in performance-critical render paths
- Ensure injected props are stable (memoized) to avoid unnecessary re-renders
- Consider hooks or render props when behavior is instance-specific or highly dynamic

### Related Concepts
TBD