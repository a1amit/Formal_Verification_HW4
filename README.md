# Homework 5: Verification with Finite Automata

This project focuses on formal verification by extending a Python `TransitionSystem` class. The implementation adds methods for verifying critical system properties like invariants, persistence, safety, and liveness using algorithms based on finite automata.

## Overview 📜

The core of this assignment is to bridge the gap between a system model (a Transition System) and formal properties (specified by logic or automata). This is achieved by implementing the synchronous product of a transition system with a finite automaton, which allows us to use graph algorithms to check for property violations.

The implemented methods can automatically verify if a given system model guarantees certain behaviors, such as never reaching a "bad" state (safety) or always eventually reaching a "good" state (liveness).

***

## Core Concepts 🧠

-   **Transition System (TS)**: A graph-based model representing system behavior. It consists of states, actions, and transitions. States can be labeled with atomic propositions that describe what is true in that state.

-   **Finite Automaton (FA)**: A mathematical model used here to specify properties of execution traces. We use automata to define both finite "bad prefixes" (for safety) and infinite "bad traces" (for liveness).

-   **Product Construction (TS × FA)**: A fundamental technique in model checking. It creates a new transition system whose states are pairs `(system_state, automaton_state)`. An execution in the product corresponds to a simultaneous run in both the original system and the automaton, allowing us to check if the system's behavior is accepted by the property automaton.

-   **Verification Properties**:
    -   **Invariant (`□k`)**: A property `k` must hold in **all** reachable states.
    -   **Persistence (`◇□k`)**: Every execution must **eventually** reach and permanently remain in a state where property `k` holds.
    -   **Safety**: "Nothing bad ever happens." We verify this by checking that the system cannot produce any finite execution trace that is recognized by a "bad prefix" automaton.
    -   **Liveness**: "Something good eventually happens." We verify this by checking that the system cannot produce any infinite execution trace that is recognized by an automaton for "bad" (violating) behaviors.

***

## Implemented Methods 💻

The `TransitionSystem` class was extended with the following verification methods:

### `product(self, a: FiniteAutomaton)`
Constructs the synchronous product of the transition system with a given finite automaton.
-   **States**: Pairs `(s, q)` where `s` is a TS state and `q` is an FA state.
-   **Transitions**: A transition `((s, q), act, (t, p))` exists in the product if `(s, act, t)` is a TS transition and there's a corresponding FA transition `(q, φ, p)` where the condition `φ` is true in the target state `t`.

### `verify_invariant(self, k: str)`
Checks if the propositional formula `k` holds for every reachable state of the system.
-   **Implementation**: It performs a reachability analysis (e.g., BFS) from the initial states and checks the formula `k` against the labels of each reached state.

### `verify_persistence(self, k: str)`
Verifies that for every possible execution, the system eventually enters and stays within a set of states where `k` holds true.
-   **Implementation**: This algorithm first identifies the largest set of states `C` where `k` is invariant. It then checks the system's graph for any reachable, non-trivial **Strongly Connected Components (SCCs)** that are entirely outside of `C`. The existence of such a cycle indicates a persistence violation.

### `verify_safety(self, a: FiniteAutomaton)`
Verifies that the system satisfies a safety property. The automaton `a` is designed to accept all "bad prefixes" that violate the property.
-   **Implementation**: It builds the product `TS × A` and checks if any reachable state in the product corresponds to an **accepting state** of the automaton `a`. If so, a bad prefix is possible, and the property is violated.

### `verify_liveness(self, a: FiniteAutomaton)`
Verifies that the system satisfies a liveness property. The automaton `a` is designed to accept all infinite traces that violate the property.
-   **Implementation**: It builds the product `TS × A` and checks for a reachable, non-trivial **SCC** that also contains an accepting state of `a`. Such a cycle corresponds to an infinite, violating execution (satisfying the Büchi acceptance condition).

***

## Visualization 📊

### `plot()`
A helper method is provided to visualize the `TransitionSystem` instance as a directed graph. This is extremely useful for debugging models and understanding verification results.
-   **Initial states** are colored `skyblue`.
-   Nodes are labeled with their state name and atomic propositions.
-   Edges are labeled with actions.

***

## Usage Example 🚀

Here is a simple example of how to define a `TransitionSystem`, check an invariant property, and visualize it.

```python
# This example assumes the TransitionSystem class is defined.

# 1. Define a Transition System
ts = TransitionSystem(
    states={'s0', 's1'},
    actions={'a'},
    atomic_props={'p'},
    initial_states={'s0'}
)

# 2. Add labels and a transition where 'p' is not always true
ts.add_label('s0', 'p')
ts.add_transition(('s0', 'a', 's1')) # s1 has no labels, so 'p' is false there

# 3. Verify an invariant property
# We check if 'p' holds in all reachable states. It should fail.
invariant_holds = ts.verify_invariant('p')
print(f"Property 'always p' holds: {invariant_holds}") # Expected: False

# 4. Visualize the system
ts.plot(title="Simple System with Invariant Violation")

```

***

## Dependencies
This project requires the following Python libraries:
-   `networkx`
-   `matplotlib`
