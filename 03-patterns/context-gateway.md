# Context Gateway Pattern

## Problem Signals
- Context usage spreads across the tree, making dependencies implicit and hard to audit
- Deep components consume multiple contexts, increasing coupling and test setup
- Providers are scattered and inconsistent, making composition and refactors fragile

## Core Idea
A Context Gateway centralizes access to one or more contexts behind a single boundary. Consumers depend on the gateway instead of the raw contexts, making dependencies explicit and easier to evolve.

This pattern improves ownership, composition, and testability without removing the benefits of context.

## Reference Details

### Definition
A Context Gateway is a component or hook that reads from one or more React contexts and exposes a stable, typed interface to its subtree. It acts as a single access point for cross-cutting concerns.

### When to Use
- Multiple contexts are required in a feature or subtree
- You want to enforce a stable dependency contract for consumers
- Context providers must be composed in a consistent, audited order
- You need to isolate context wiring for testing or refactoring
### When to Avoid
- Only one context is used and direct consumption is simple
- The gateway would hide important ownership boundaries
- Context usage is minimal and does not justify an extra abstraction
- The team prefers explicit context use in each component
### Mechanics
1. **Provider composition:** Wrap providers in a single gateway component.
2. **Access point:** Expose a `useGateway` hook for consumers.
3. **Stable contract:** Return a typed object that hides internal context shape.
4. **Isolation:** Tests can mock the gateway instead of multiple contexts.

### TypeScript Example

```tsx
import { createContext, useContext } from "react";

export type User = { id: string; name: string };
export type FeatureFlags = { betaUI: boolean };

const UserContext = createContext<User | null>(null);
const FlagsContext = createContext<FeatureFlags | null>(null);

export function useGateway() {
    const user = useContext(UserContext);
    const flags = useContext(FlagsContext);

    if (!user || !flags) {
        throw new Error("Gateway must be used within providers.");
    }

    return {
        user,
        flags,
    };
}

export function GatewayProvider(props: {
    user: User;
    flags: FeatureFlags;
    children: React.ReactNode;
}) {
    const { user, flags, children } = props;

    return (
        <UserContext.Provider value={user}>
            <FlagsContext.Provider value={flags}>
                {children}
            </FlagsContext.Provider>
        </UserContext.Provider>
    );
}
```

```tsx
import { useGateway } from "./Gateway";

export function AccountPanel() {
    const { user, flags } = useGateway();

    if (!flags.betaUI) return null;
    return <div>Welcome, {user.name}</div>;
}
```

### Scalability Notes

#### Strengths
- Centralizes context wiring for consistent composition
- Consumers depend on a stable contract instead of raw context shapes
- Testing is simpler with a single gateway boundary
#### Costs
- Adds an abstraction layer that can hide underlying dependencies
- Changes to the gateway contract can become high-impact
- Overuse may produce monolithic "god" gateways
#### System-Level Considerations
- Keep gateways domain-scoped, not application-global
- Avoid mixing unrelated concerns in a single gateway
- Prefer multiple gateways over one large contract
### Related Patterns
- Compound Components
- Event-Action Pattern
- Container-Presentation Components
- Hooks
## Summary
- Gateways make context dependencies explicit and centralized
- They reduce wiring duplication and improve testability
- Overuse can hide coupling and create oversized contracts
## Next Steps
- Consider splitting gateways by domain as contracts grow
- Use Compound Components for implicit coordination inside a subtree
