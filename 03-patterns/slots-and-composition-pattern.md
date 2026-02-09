# Slots and Composition Pattern

## Problem Signals
- Prop-driven content variants or duplicated components create brittle composition
- API surfaces grow with one-off props, reducing clarity and type safety
- Deep conditional rendering makes layouts inflexible and hard to evolve

## Core Idea

The Slots and Composition pattern defines placeholders (slots) that consumers fill by composition rather than configuration props. This keeps layout flexible without inflating component APIs.

It aligns with React's composition model and allows callers to inject JSX into predefined regions while preserving structure and ownership.

## Reference Details

### Definition

A slot represents a placeholder in a component's output where external content can be inserted. Slots allow a parent component to render external child elements in specific parts of its layout.

Using slots means the component is composed with child elements and therefore is a structured form of composition.

### When to Use

- Building components that have multiple semantically distinct regions or extensibility points, common scenarios include:
- Layout or Card Components
- Dialogs, Modals, Page Layouts
- Design System Components
- Avoiding prop drilling and one-off props
- Multiple children requirements in a specific order to enforce structure while still taking arbitrary content
### When to Avoid

- Introducing slots for simple components results in overcomplicated APIs or footprint
- Type safety is a priority in architecture for all parts of the component
- Too many slots leads to complexity and at some point, it might be better to break the component into smaller parts or a design rethought is needed
- The primary need is to inject behavior or logic rather than elements
- Server-Side Rendering constraints wouldn't allow advanced slot techniques, be careful with patterns that delay slot rendering until client-side
### Mechanics

1. **Default Slot:** Use `children` as the primary slot for unstructured content.
2. **Named Slot Props:** Expose fixed regions like `header` and `footer` for predictable layout.
3. **Compound Subcomponents:** Provide static members (e.g. `Card.Header`) and map child types to slots.
4. **Generic Slot Marker:** Accept `<Slot name="...">` nodes and route by name.
5. **Context Registry:** Allow descendants to register slot content via a provider and hook.
6. **Slot/Fill Portals:** Inject content across the tree using named Slot/Fill pairs and portals.
7. **asChild Composition:** Let consumers replace the rendered element while inheriting behavior via prop merging.

### TypeScript Example

Minimal API sketches for common slot strategies:

#### Default Slot
```tsx
export function Card(props: { children?: React.ReactNode }) {
    return <section>{props.children}</section>;
}
```

#### Named Slot Props
```tsx
export function Card(props: {
    header?: React.ReactNode;
    children?: React.ReactNode;
    footer?: React.ReactNode;
}) {
    return (
        <section>
            {props.header}
            {props.children}
            {props.footer}
        </section>
    );
}
```

#### Compound Subcomponents
```tsx
type CardComponent = React.FC<{ children?: React.ReactNode }> & {
    Header: React.FC<{ children?: React.ReactNode }>;
    Footer: React.FC<{ children?: React.ReactNode }>;
};
```

```tsx
<Card>
    <Card.Header>Title</Card.Header>
    <p>Body</p>
    <Card.Footer>Footer</Card.Footer>
</Card>
```

#### Generic Slot Marker
```tsx
export function Slot(props: {
    name: "header" | "footer";
    children?: React.ReactNode;
}) {
    return null;
}
```

#### Context Registry
```tsx
type SlotRegistry = {
    register: (name: string, content: React.ReactNode) => void;
    read: (name: string) => React.ReactNode | null;
};
```

#### Slot/Fill Portals
```tsx
export function Slot(props: { name: string }) {
    return null;
}

export function Fill(props: { name: string; children: React.ReactNode }) {
    return null;
}
```

#### asChild Composition
```tsx
export function Button(props: {
    asChild?: boolean;
    children: React.ReactElement;
}) {
    return props.asChild ? props.children : <button>{props.children}</button>;
}
```

### Scalability Notes

#### Strengths

- Flexibility & Reuse
- Semantic & Readable Usage
- Design System Alignment
#### Costs
- Type Complexity
- Non-React idiomatic usage can lead to performance overhead
- Debugging Complexity
- High maintenance cost and evolution
#### System-Level Considerations
- Promotes an open/closed principle: components are closed for modification but open for extension
### Related Patterns

- Render Props
- Compound Components
- Hooks and Headless Hooks
- Higher-Order Components (HOC)
- Portal Rendering
## Summary
- Slots preserve layout control while keeping component structure explicit
- Multiple slot strategies trade type safety, flexibility, and runtime cost
- Overuse can complicate APIs and blur ownership of structure
## Next Steps
- Consider Compound Components when slot content needs shared behavior
- Explore Render Props for behavior-first composition
