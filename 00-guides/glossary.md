# Glossary

This glossary defines the canonical terminology used throughout this documentation.
All patterns, examples, and architectural discussions assume these definitions.
Terms are intentionally framed at the **component design and system architecture level**, not as React tutorials.

This file is a living reference and should be updated as new patterns introduce new primitives.

---

## Controlled Component
A component whose meaningful state is owned externally and driven through props. State transitions are requested via callbacks, making the parent the source of truth.

---

## Uncontrolled Component
A component that owns and manages its own internal state. External consumers cannot directly dictate internal transitions and can only observe outcomes via emitted events.

---

## State Ownership
The architectural decision determining where a component’s authoritative state lives—internally within the component or externally in a parent or global store.

---

## Internal State
State maintained privately inside a component, typically via `useState`, `useRef`, or a localized state machine, and not directly writable by external consumers.

---

## External Control
A design where state transitions are dictated by parent components through props, enabling orchestration but increasing coupling and render scope.

---

## Configuration Props
Props used to configure component behavior or mode (e.g., `disabled`, `readOnly`, feature flags) without directly controlling state transitions.

---

## Control Channel
A prop-based pathway through which external components can directly influence a child’s state (e.g., `value`, `isOpen`, `activeIndex`).

---

## Event Emission
The act of notifying external consumers of internal state changes or completed interactions through callbacks (e.g., `onChange`, `onCommit`, `onReset`).

---

## Output Signal
A discrete event emitted after an internal transition completes, representing an observable outcome rather than intermediate state.

---

## Internal Transition
A state change processed entirely within a component, including intermediate or transient values that are not exposed externally.

---

## Ephemeral State
Short-lived, high-frequency, or interaction-bound state (e.g., dragging, scrubbing, hover transitions) that typically does not benefit from external control.

---

## Behavioral State
State that directly governs how a component behaves or transitions, as opposed to static configuration or purely visual props.

---

## Partial Control
A hybrid approach where a component mixes internal state with externally controlled inputs. Common in real-world systems and a frequent source of complexity.

---

## Reconciliation Footprint
The portion of the component tree affected by React’s reconciliation process during a state update.

---

## Re-render Cascade
A chain of component re-renders triggered by state updates propagating upward or across the component tree.

---

## Shallow Reconciliation Boundary
A design where state updates are contained within a narrow component subtree, minimizing render propagation.

---

## Encapsulation Boundary
A deliberate isolation layer where internal logic and state are hidden behind a minimal public API.

---

## Deterministic Flow
A system where state transitions are externally reproducible, traceable, and controllable (e.g., step-based workflows, wizards).

---

## Orchestration
The coordination of multiple components’ behavior through centralized state or control logic.

---

## Interaction-Heavy Island
A localized region of the UI with dense, high-frequency interactions that benefits from internal state ownership and isolation.

---

## Controller
An external abstraction that coordinates multiple components by consuming their output signals without directly controlling their internal state.

---

## Facade
A higher-level interface that simplifies interaction with one or more components while preserving their internal autonomy.

---

## Static Configuration
Non-dynamic inputs that influence component behavior without participating in runtime state transitions.

---

## Internal Update Authority
The principle that only the component itself can mutate its internal state, ensuring predictable behavior across renders.

---

## Macro-Level Coordination
System-wide state synchronization across multiple components or domains, typically incompatible with fully uncontrolled patterns.

---

## Micro-Level Interaction
Fine-grained, localized UI behavior best handled within a component’s internal state.

---

## Reference-Grade Pattern
A documented pattern that emphasizes trade-offs, constraints, and scalability over prescriptive usage.

---

### Usage Notes
- All future patterns should reference these terms instead of redefining them.
- New terms should be added here before being introduced elsewhere.
- Terminology consistency is a design constraint, not a suggestion.
