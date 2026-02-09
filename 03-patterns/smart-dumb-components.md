# Smart vs Dumb Components Pattern

## Problem Signals
- Presentational components accumulate data, state, and side effects, reducing reuse
- UI changes require touching business logic because behavior and rendering are coupled
- Reuse across data sources or testing requires heavy mocking and setup

## Core Idea
The Smart vs Dumb Components pattern separates behavior from presentation. Smart components own data acquisition, state, and side effects, while Dumb components render UI based purely on props.

This separation keeps view components stable and reusable while allowing behavior to evolve independently.

## Reference Details

### Definition
A Smart component (container) orchestrates data and interactions. A Dumb component (presentation) receives props and renders without owning side effects or external state.

### When to Use
- UI must be reused across multiple data sources or behaviors
- Business logic changes frequently while UI structure stays stable
- You want to isolate rendering for straightforward unit testing
- Clear boundaries between data/behavior and view improve ownership
### When to Avoid
- The component is small and unlikely to be reused
- Splitting logic adds indirection with little benefit
- Behavior is entirely local and better expressed in a single component
- You already use hooks and composition to keep logic isolated
### Mechanics
1. **Smart layer:** Fetches data, owns state, and defines handlers.
2. **Dumb layer:** Receives typed props and renders UI only.
3. **Explicit contract:** Prop types define the boundary clearly.
4. **Stable views:** Presentational components remain logic-free and easy to test.

### TypeScript Example

```tsx
import { useEffect, useState } from "react";

export type User = {
    id: string;
    name: string;
};

export interface UserListViewProps {
    users: User[];
    isLoading: boolean;
    onSelect: (userId: string) => void;
}

export function UserListView(props: UserListViewProps) {
    const { users, isLoading, onSelect } = props;

    if (isLoading) return <div>Loading...</div>;

    return (
        <ul>
            {users.map((user) => (
                <li key={user.id}>
                    <button onClick={() => onSelect(user.id)}>
                        {user.name}
                    </button>
                </li>
            ))}
        </ul>
    );
}
```

```tsx
import { useCallback, useEffect, useState } from "react";
import { User, UserListView } from "./UserListView";

async function fetchUsers(): Promise<User[]> {
    return [
        { id: "1", name: "Ada" },
        { id: "2", name: "Linus" },
    ];
}

export function UserListSmart() {
    const [users, setUsers] = useState<User[]>([]);
    const [isLoading, setIsLoading] = useState(true);

    useEffect(() => {
        let active = true;

        fetchUsers().then((data) => {
            if (!active) return;
            setUsers(data);
            setIsLoading(false);
        });

        return () => {
            active = false;
        };
    }, []);

    const handleSelect = useCallback((userId: string) => {
        // Smart layer coordinates side effects or navigation.
        console.log("Selected user", userId);
    }, []);

    return (
        <UserListView
            users={users}
            isLoading={isLoading}
            onSelect={handleSelect}
        />
    );
}
```

### Scalability Notes

#### Strengths
- Reusable views with stable, predictable props
- Business logic changes without touching UI components
- Easier to test presentation separately from behavior
#### Costs
- Extra files and indirection in small features
- Risk of duplicated props and boilerplate
- Boundaries can blur if smart components leak UI concerns
#### System-Level Considerations
- Keep smart components thin; push logic into hooks or services
- Align with folder structure and ownership boundaries
- Use for high-variance behavior with stable UI
### Related Patterns
- Container-Presentation Components
- Hooks
- Render Props
- Higher-Order Components (HOC)
## Summary
- Smart components own behavior; dumb components render UI
- Clear boundaries improve reuse and testability
- Overuse can introduce unnecessary indirection
## Next Steps
- Consider Container-Presentation Components for similar separation
- Use hooks when logic reuse does not require a component boundary
