# Compound Components Pattern

## Problem Signal
- Passing large JSX elements via props or duplicating components to achieve slight content variations is a destructive composition pattern
- Consumers of a component actively sacrifice clarity or type safety
- The API is cluttered or inflexible and the UI component is complex when building
- Resorting to numerous props or deeply nested conditional rendering which makes the component API cumbersome

## Core Idea

The Slots and Composition pattern encourages designing components that define placeholders (slots) for content, which users of the component can fill by composition (using children or designated subcomponents) rather than configuration.

This pattern is rooted in React's ethos of "composition over inheritance" and is analogous to the slots concept in Web Components.

By using slots, the component can provide a structural framework while allowing callers to inject arbitrary JSX at predefined locations. This results in extensibility (the ability that consumers can supply their own content without modifying the component's internal code)

## Evidence & Implementation

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

### Avoid When

- Introduing slots for simple components results in overcomplicated APIs or footprint
- Type safety is a priotity in architecture for all parts of the component
- Too many slots leads to complexity and at some point, it might be better to break the component into smaller parts or a design rethought is needed
- The primary need is to inject behavior or logic rather than elements
- Server-Side Rendering constraints wouldn't allow advanced slot techniques, be careful with patterns that delay slot rendering until client-side

### Mechanics

1. Default slot (`children`)
   - Expore `children?: React.ReactNode`
   - Render `{children}` at the intended insertion point
2. Multiple named slot props (explicit props per region)
   - Define props per slot, e.g. `header?: React.ReactNode`, `footer?: React.ReactNode`
   - Render each prop in a fixed region (`<header>`, `<footer>`, etc.)
   - 
3. Compound Components (static members, e.g. `Card.Header`)
    - Create subcomponents for each slot: `Card.Header`, `Card.Footer`, etc.
    - Attach as static members on the parent component
    - In parent:
      - Iterate children (`React.Children.forEach`)
      - Validate elements (`React.isValidElement`
      - Match by component identity (`child.type === CardHeader`, etc.)
      - Route matched children into slot containers; treat remaining nodes as default/body
4. Geeneric `<Slot name="...">` elements (string-keyed routing)
   - Implement a generic `Slot` component with `name: string`
   - Parent scans children for `Slot` elements and groups by `name`
5. Context-based slot registry (registration model)
    - Parent provides a "slot registry" via Context
    - Slot components register/unregister content on mount/unmount (effects)
    - Parent renders registered content into the correct regions
6. Portal-based Slot/Fill (cross-tree injection)
   - Define `<Slot>` targets and `<Fill>` providers keyed by name/id
   - Route Fill content into Slot using Context/events-bus + React Portals
7. `asChild` / prop-cloning composition (wrapper replacement)
   - Add `asChild? boolean` (or `as` prop) to allow overriding the rendered element
   - When `asChild`:
     - Require exactly one valid React element child
     - `React.cloneElement` to merge props/handlers/ARIA/className onto the child
   - Requirements:
     - Ensure ref forwarding (`forwardRef`)
     - Ensure unknown props are spread through (accessibility + interoperability)

// TODO: refactor Portal-based Slot/Fill into Portal pattern

### TypeScript & implementation hard requirements (apply to all patterns)
- Define slot content types explicitly (`React.ReactNode` or stricter)
- Preserve accessibility semantics (correct elements/ARIA, avoid confusing DOM reshuffles)
- Support ref forwarding where composition expects it (compound/context/asChild)
- Support prop spreading for extensibility (especially `asChild`/cloning patterns)
- Document:
  - Valid slot regions/names
  - Ordering rules and constraints
  - SSR expectations/limitations for registry/portal approaches

### Implementation

#### Compound Component Slot Pattern

```tsx
// Card.tsx
import React, { ReactElement, ReactNode } from "react";

/*
    Slot-based Compound Component Pattern (Static Subcomponents)

    Intent:
    - Provide a declarative, named-slot API (Card.Header / Card.Footer)
    - Parse children once at the parent boundary
    - Enforce structure without prop drilling
    - Preserve consumer freedom for body content
*/

/** Shared slot props: slots are structural markers, not render owners. */
export interface CardSlotProps {
    children?: ReactNode;
}

/** Slot markers (renderless by design). */
function HeaderSlot(props: CardSlotProps) {
    return null;
}

function FooterSlot(props: CardSlotProps) {
    return null;
}

/** Parent props */
export interface CardProps {
    children?: ReactNode;
}

/** Public component surface (Root + static slots). */
export type CardComponent = React.FC<CardProps> & {
    Header: React.FC<CardSlotProps>;
    Footer: React.FC<CardSlotProps>;
};

type Slots = {
    header: ReactNode | null;
    footer: ReactNode | null;
    body: ReactNode[];
};

/** Deterministic child parsing: extracts known slots, routes remainder to body. */
function extractSlots(children: ReactNode): Slots {
    let header: ReactNode | null = null;
    let footer: ReactNode | null = null;
    const body: ReactNode[] = [];

    React.Children.forEach(children, (child) => {
        if (!React.isValidElement(child)) {
            if (child !== undefined && child !== null) body.push(child);
            return;
        }

        if (child.type === HeaderSlot) {
            header = (child as ReactElement<CardSlotProps>).props.children ?? null;
            return;
        }

        if (child.type === FooterSlot) {
            footer = (child as ReactElement<CardSlotProps>).props.children ?? null;
            return;
        }

        body.push(child);
    });

    return { header, footer, body };
}

export const Card: CardComponent = ({ children }) => {
    const { header, footer, body } = extractSlots(children);

    /*
        Rendering policy:
        - Header renders first if present
        - Body renders as provided
        - Footer renders last if present
        - Markup is intentionally minimal (styling omitted)
    */

    return (
        <div>
            {header !== null && <header>{header}</header>}
            {body}
            {footer !== null && <footer>{footer}</footer>}
        </div>
    );
};

Card.Header = HeaderSlot;
Card.Footer = FooterSlot;
```

```tsx
// Consumer.tsx
import { Card } from "./Card";

export function Consumer() {
    return (
        <Card>
            <Card.Header>Title</Card.Header>
            <p>Body content.</p>
            <Card.Footer>Footer content.</Card.Footer>
        </Card>
    );
}
```

This subpattern is optimal for:
- Declarative layout
- JSX-first composition with clear semantics
- Type Safety — Strong TS support out of the box
- API surface that reads like layout structure

If order must be enforced, document rules or validate (avoid silent DOM reordering)
Use Context for coordinated behavior (state sharing across subcomponents)

#### Named Slot via Props

```tsx
// NamedSlotCard.tsx
import React, { ReactNode } from "react";

/*
    Named Slots via Props (Composition Subpattern)

    Intent:
    - Provide explicit structural insertion points (header/footer) without child parsing
    - Keep slot API discoverable and statically typed
    - Preserve flexible body composition via children
    - Prefer when slot structure is fixed and known at the call site
*/

export interface NamedSlotCardProps {
    /** Optional header slot content. */
    header?: ReactNode;

    /** Optional footer slot content. */
    footer?: ReactNode;

    /** Body content (default slot). */
    children?: ReactNode;
}

export function NamedSlotCard(props: NamedSlotCardProps) {
    const { header, footer, children } = props;

    /*
        Rendering policy:
        - Render header first if provided
        - Render body as the default slot
        - Render footer last if provided
        - Markup is intentionally minimal (styling omitted)
    */

    return (
        <div>
            {header !== undefined && header !== null && (
                <header>{header}</header>
            )}

            <div>{children}</div>

            {footer !== undefined && footer !== null && (
                <footer>{footer}</footer>
            )}
        </div>
    );
}
```

```tsx
// Parent.tsx
import { NamedSlotCard } from "./NamedSlotCard";

export function Parent() {
    return (
        <NamedSlotCard
            header={<h1>Title</h1>}
            footer={<button>Continue</button>}
        >
            <p>Body content goes here.</p>
        </NamedSlotCard>
    );
}
```

This subpattern is optimal for:
- Fixed, well-known layout
- API clarity
- Composition with non-React callers
- Strict TS typing per region

Trade-offs
- Composition moves from JSX children to props
- Slot structure/positions become fixed by prop names
- Reordering requires changing prop usage (not JSX order)

#### Generic Slot (String-Based Slot Resolution)

```tsx
// Slot.tsx
import React, { ReactNode } from "react";

/*
    Generic Slot Marker (String-Based Slot Resolution)

    Intent:
    - Provide a generic slot carrier that can represent many named insertion points
    - Allow containers to resolve slot placement via a string key
    - Keep slot instances renderless / structural
*/

export interface SlotProps<TName extends string = string> {
    /** Slot identifier used by the parent container for resolution. */
    name: TName;
    children?: ReactNode;
}

/** Slot is a structural marker. It does not render UI. */
export function Slot<TName extends string = string>(
    _props: SlotProps<TName>
) {
    return null;
}
```

```tsx
// GenericSlotContainer.tsx
import React, { Children, ReactElement, ReactNode } from "react";
import { Slot, SlotProps } from "./Slot";

/*
    Generic Slot Container (String-Based Resolution)

    Intent:
    - Parse children once
    - Extract slot content based on Slot.name
    - Route all non-slot children into the default body slot
    - Keep rendering policy explicit and deterministic
*/

export type SlotName = "header" | "footer";

export interface GenericSlotContainerProps {
    children?: ReactNode;
}

type ResolvedSlots = {
    header: ReactNode | null;
    footer: ReactNode | null;
    body: ReactNode[];
};

function isSlotElement<TName extends string>(
    node: ReactNode
): node is ReactElement<SlotProps<TName>> {
    return React.isValidElement(node) && node.type === Slot;
}

function resolveSlots(children: ReactNode): ResolvedSlots {
    let header: ReactNode | null = null;
    let footer: ReactNode | null = null;
    const body: ReactNode[] = [];

    Children.forEach(children, (child) => {
        if (!child && child !== 0) return;

        if (!isSlotElement<SlotName>(child)) {
            body.push(child);
            return;
        }

        switch (child.props.name) {
            case "header":
                header = child.props.children ?? null;
                return;

            case "footer":
                footer = child.props.children ?? null;
                return;

            default:
                body.push(child);
                return;
        }
    });

    return { header, footer, body };
}

export function GenericSlotContainer(props: GenericSlotContainerProps) {
    const { children } = props;

    const { header, footer, body } = resolveSlots(children);

    return (
        <div>
            {header !== null && <header>{header}</header>}
            <div>{body}</div>
            {footer !== null && <footer>{footer}</footer>}
        </div>
    );
}
```

```tsx
// Parent.tsx
import { GenericSlotContainer } from "./GenericSlotContainer";
import { Slot } from "./Slot";

export function Parent() {
    return (
        <GenericSlotContainer>
            <Slot name="header">
                <h2>Header</h2>
            </Slot>

            <p>Body content</p>

            <Slot name="footer">
                <button>OK</button>
            </Slot>
        </GenericSlotContainer>
    );
}
```

This subpattern is optimal for:
- Many potential slot names
- Generic slot primitive usable across multiple containers
- Config-driven slot assignment
- A single slot wrapper component instead of many subcomponents

Trade-offs:
- Slot names are "stringly" typed -> typos can faill silently
- Valid slot names rely on docs or custom TS augmentation

#### Context-Based Slot Registry

```tsx
// SlotRegistryContext.tsx
import React, {
    createContext,
    useContext,
    useMemo,
    useRef,
    useLayoutEffect,
    ReactNode,
} from "react";

/*
    Context-Based Slot Registry (Registry + Receivers)

    Intent:
    - Allow distant descendants (plugins/feature modules) to contribute slot content
    - Decouple slot declaration (Slot) from slot placement (SlotReceiver)
    - Avoid child parsing and static subcomponent identity constraints
    - Enable composition across subtree boundaries

    Core Mechanism:
    - A Provider owns a registry: Map<slotName, ReactNode>
    - Slot registers content into the registry
    - SlotReceiver reads and renders resolved content at layout positions
*/

type SlotRegistry = Map<string, ReactNode>;

export interface SlotRegistryContextValue {
    register: (name: string, content: ReactNode) => void;
    read: (name: string) => ReactNode | null;
}

const SlotRegistryContext = createContext<SlotRegistryContextValue | null>(
    null
);

function useSlotRegistryContext(): SlotRegistryContextValue {
    const ctx = useContext(SlotRegistryContext);
    if (!ctx) {
        throw new Error(
            "Slot registry hooks must be used within <SlotRegistryProvider />"
        );
    }
    return ctx;
}

/** Provider owns the registry boundary. */
export function SlotRegistryProvider(props: { children: ReactNode }) {
    const registryRef = useRef<SlotRegistry>(new Map());

    const value = useMemo<SlotRegistryContextValue>(() => {
        return {
            register: (name: string, content: ReactNode) => {
                registryRef.current.set(name, content);
            },
            read: (name: string) => {
                return registryRef.current.get(name) ?? null;
            },
        };
    }, []);

    return (
        <SlotRegistryContext.Provider value={value}>
            {props.children}
        </SlotRegistryContext.Provider>
    );
}

/*
    Slot registration hook:
    - Registers content during layout phase to make it available as early as possible.
    - This is a registry write; it intentionally does not render.
*/
export function useRegisterSlot(name: string, content: ReactNode) {
    const ctx = useSlotRegistryContext();

    useLayoutEffect(() => {
        ctx.register(name, content);
    }, [ctx, name, content]);
}

/** Slot read hook used by layout receivers. */
export function useSlot(name: string): ReactNode | null {
    const ctx = useSlotRegistryContext();
    return ctx.read(name);
}
```

```tsx
// Slot.tsx
import React, { ReactNode } from "react";
import { useRegisterSlot } from "./SlotRegistryContext";

/*
    Slot (Writer)

    Intent:
    - Declare slot content from anywhere in the provider subtree
    - Contribute content into the registry
    - Renderless by design
*/

export interface SlotProps<TName extends string = string> {
    name: TName;
    children: ReactNode;
}

export function Slot<TName extends string = string>(props: SlotProps<TName>) {
    useRegisterSlot(props.name, props.children);
    return null;
}
```

```tsx
// Slot.tsx
import React, { ReactNode } from "react";
import { useRegisterSlot } from "./SlotRegistryContext";

/*
    Slot (Writer)

    Intent:
    - Declare slot content from anywhere in the provider subtree
    - Contribute content into the registry
    - Renderless by design
*/

export interface SlotProps<TName extends string = string> {
    name: TName;
    children: ReactNode;
}

export function Slot<TName extends string = string>(props: SlotProps<TName>) {
    useRegisterSlot(props.name, props.children);
    return null;
}
```

```tsx
// SlotReceiver.tsx
import React from "react";
import { useSlot } from "./SlotRegistryContext";

/*
    SlotReceiver (Reader)

    Intent:
    - Define structural placement points in a layout
    - Pull registered content from the registry by name
    - Render nothing when content is absent
*/

export interface SlotReceiverProps<TName extends string = string> {
    name: TName;
    as?: keyof JSX.IntrinsicElements;
}

export function SlotReceiver<TName extends string = string>(
    props: SlotReceiverProps<TName>
) {
    const { name, as: Tag = "div" } = props;
    const content = useSlot(name);

    if (content === null) return null;
    return <Tag>{content}</Tag>;
}
```

```tsx
// ParentLayout.tsx
import React from "react";
import { SlotRegistryProvider } from "./SlotRegistryContext";
import { SlotReceiver } from "./SlotReceiver";
import { Slot } from "./Slot";

/*
    Usage Pattern

    Layout defines placement (receivers).
    Descendants contribute content (slots).
    The registry connects them without prop drilling or child parsing.
*/

export function ParentLayout() {
    return (
        <SlotRegistryProvider>
            <div>
                <SlotReceiver name="header" as="header" />

                <main>
                    {/* Layout-owned content */}
                    <p>Main content here.</p>
                </main>

                <SlotReceiver name="footer" as="footer" />
            </div>

            <ChildPlugin />
        </SlotRegistryProvider>
    );
}

function ChildPlugin() {
    return (
        <>
            <Slot name="header">
                <h1>Injected Header</h1>
            </Slot>

            <Slot name="footer">
                <button>Close</button>
            </Slot>
        </>
    );
}
```

This subpattern is optimal for:
- Plugin-style composition
- Avoids brittle identity checks and child parsing overhead

The main trade-off is implicit coupling via shared registry keys and the need for collision/precedence rules if multiple writers target the same slot.

#### asChild / Prop Cloning Slot

```tsx
// AsChild.tsx
import React, {
    cloneElement,
    forwardRef,
    isValidElement,
    ReactElement,
    ReactNode,
    Ref,
} from "react";

/*
    asChild / Prop Cloning Slot (Composition Subpattern)

    Intent:
    - Allow a component to "delegate" its rendered element to a consumer-provided child
    - Preserve the component’s behavior (props, handlers, ARIA, className, ref)
    - Avoid additional wrapper DOM nodes
    - Provide a slot-like "render target" via the first child element
*/

/** Minimal contract: asChild expects exactly one valid React element. */
export interface AsChildProps {
    children: ReactElement;
    /** Props to inject into the child element. */
    inject: Record<string, unknown>;
    /** Optional forwarded ref applied to the child. */
    forwardedRef?: Ref<unknown>;
}

export function AsChild(props: AsChildProps): ReactElement | null {
    const { children, inject, forwardedRef } = props;

    if (!isValidElement(children)) return null;

    /*
        Design constraints:
        - Child must accept injected props (e.g. className, onClick, aria-*)
        - Child must support ref if forwardedRef is provided (depends on element/component)
        - Injection semantics must be defined by the caller (merge strategy is component-specific)
    */

    return cloneElement(children, {
        ...inject,
        ref: forwardedRef,
    });
}
```

```tsx
// Button.tsx
import React, { ButtonHTMLAttributes, forwardRef, ReactElement } from "react";
import { AsChild } from "./AsChild";

/*
    Component using asChild

    Intent:
    - Default: render a semantic <button>
    - asChild: render consumer’s element, but inject Button behavior into it
    - No extra wrapper node
*/

export interface ButtonProps
    extends ButtonHTMLAttributes<HTMLButtonElement> {
    asChild?: boolean;
    children: ReactElement;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
    (props, ref) => {
        const { asChild = false, children, ...rest } = props;

        if (asChild) {
            return (
                <AsChild
                    inject={rest}
                    forwardedRef={ref}
                >
                    {children}
                </AsChild>
            );
        }

        return (
            <button ref={ref} {...rest}>
                {children}
            </button>
        );
    }
);

Button.displayName = "Button";
```

```tsx
// Parent.tsx
import React from "react";
import { Button } from "./Button";

export function Parent() {
    return (
        <Button asChild>
            <a href="/next">Next</a>
        </Button>
    );
}
```

This subpattern is optimal for:
- Zero extra wrapper DOM
- Behavior injection is needed but with consumer-controlled semantics / user-provided elements
- Prop merging

This reference uses a simple spread; production components typically define explicit merge rules to avoid overriding consumer props unintentionally.

### Scalability Notes

#### Strengths

- Flexibility & Reuse
- Semantic & Readable Usage
- Design System Alignment

#### Costs
- Type Complexity
- non React idoimatic if done extrememly leading to performance overhead
- Debugging Complexity
- High Maintenance cost and Evolution

#### System-Level Considerations
- Promotes an open/closed principle: components are closed for modification but open for extension

### Related Concepts

- Render Props
- Compound Components
- Hooks and Headless Hooks
- Higher-Order Components (HOC)
- Portal Rendering