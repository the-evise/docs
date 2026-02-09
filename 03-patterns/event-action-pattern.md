# Event-Action Pattern

## Problem Signals
- Components emit many granular events that are hard to coordinate across features
- Event handlers become large and imperative, mixing interpretation with side effects
- Interaction logic is duplicated and consumers lack a stable event-to-action contract

## Core Idea
The Event-Action pattern separates raw UI events from domain actions. Components emit low-level events, while an action layer interprets them into semantic commands and triggers side effects.

This keeps components focused on interaction signals and centralizes behavioral rules in a single, testable layer.

## Reference Details

### Definition
Events are interaction signals emitted by UI components (e.g., `onChange`, `onSubmit`, `onSelect`). Actions are domain-level intents (e.g., `SaveDraft`, `CommitSelection`). The pattern introduces a translation layer that maps events to actions and executes the resulting behavior.

### When to Use
- Many components emit similar events that need consistent interpretation
- You want to keep UI components stateless or behavior-light
- Domain actions require validation, analytics, or side effects
- You need a stable, testable boundary between UI and business rules
### When to Avoid
- The component is small and the event handling is trivial
- Actions add indirection with no reuse or coordination benefit
- The UI is already controlled by a centralized state machine or store
- Event-to-action mapping would be one-off and short-lived
### Mechanics
1. **Emit events:** Components only emit interaction signals.
2. **Map to actions:** A controller translates events into domain actions.
3. **Execute actions:** Side effects and state updates happen in the action layer.
4. **Keep contracts typed:** Events and actions are explicit TypeScript unions.

### TypeScript Example

```ts
export type FormEvent =
    | { type: "field_changed"; field: "name" | "email"; value: string }
    | { type: "submitted" }
    | { type: "reset" };

export type FormAction =
    | { type: "save_draft"; payload: Record<string, string> }
    | { type: "submit_form"; payload: Record<string, string> }
    | { type: "clear_form" };

export type FormState = {
    name: string;
    email: string;
};

export function mapEventToAction(
    event: FormEvent,
    state: FormState
): FormAction | null {
    switch (event.type) {
        case "field_changed":
            return { type: "save_draft", payload: { ...state, [event.field]: event.value } };
        case "submitted":
            return { type: "submit_form", payload: state };
        case "reset":
            return { type: "clear_form" };
        default:
            return null;
    }
}
```

```tsx
import { useCallback, useState } from "react";
import { FormAction, FormEvent, FormState, mapEventToAction } from "./events";

export function FormController() {
    const [state, setState] = useState<FormState>({ name: "", email: "" });

    const handleAction = useCallback((action: FormAction) => {
        switch (action.type) {
            case "save_draft":
                setState(action.payload);
                return;
            case "submit_form":
                // Submit to API, analytics, etc.
                return;
            case "clear_form":
                setState({ name: "", email: "" });
                return;
        }
    }, []);

    const handleEvent = useCallback(
        (event: FormEvent) => {
            const action = mapEventToAction(event, state);
            if (action) handleAction(action);
        },
        [state, handleAction]
    );

    return (
        <form
            onSubmit={(e) => {
                e.preventDefault();
                handleEvent({ type: "submitted" });
            }}
        >
            <input
                value={state.name}
                onChange={(e) =>
                    handleEvent({
                        type: "field_changed",
                        field: "name",
                        value: e.target.value,
                    })
                }
            />
            <input
                value={state.email}
                onChange={(e) =>
                    handleEvent({
                        type: "field_changed",
                        field: "email",
                        value: e.target.value,
                    })
                }
            />
            <button type="submit">Save</button>
            <button
                type="button"
                onClick={() => handleEvent({ type: "reset" })}
            >
                Reset
            </button>
        </form>
    );
}
```

### Scalability Notes

#### Strengths
- Centralizes domain logic for consistent behavior
- UI components stay focused on emitting events
- Actions are easy to test in isolation
#### Costs
- Adds a layer of indirection between UI and effects
- Requires careful typing to avoid bloated unions
- Mapping logic can drift without ownership rules
#### System-Level Considerations
- Keep event and action vocabularies small and stable
- Co-locate mapping with the feature domain
- Consider reducers or state machines if actions become complex
### Related Patterns
- Controlled Components
- Container-Presentation Components
- Render Props
- Hooks
## Summary
- Events describe interactions; actions describe domain intent
- A translation layer centralizes behavior and side effects
- Best when multiple components need consistent interpretation
## Next Steps
- Consider reducers or state machines for complex action flows
- Use Controlled Components when external orchestration is required
