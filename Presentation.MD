# Motivation

How can it be possible for tests to have 100% coverage and still miss bugs?

Why is it that anytime we perform manual review we end up identifying combinations not covered by tests?

Is there a way to systematically identify and explore all smart contract state combinations?

The reality is that there's a huge (factorial) amount of combinations that we don't see which belongs to mathematical properties we typically don't talk about.

## Examples

- TODO: Add both motivating examples and a few bugs
- Linear Vault being wasteful
- Lack of dictionary for fuzzer
- Extreme difficulty in reaching specific line
- Impossibility of identifying economic issues without specific instrumentation

Why is it, that bugs are gothcas?
-> Because they are
-> Because the metric we use IGNORES them.


## References to past work

Enabledness Preserving Abstractions [https://dl.acm.org/doi/10.1145/3415153] is a technique to enumerate program states, the paper referenced uses execution to explore and document the states producing an underapproximation of the system.

Having access to the source code allows us to enumerate branches, as well as determine state variables that imply different possible coverage results.

I label these groups Coverage Classes for simplicity.

EPAs are discovered at runtime by instrumenting the code with additional functions that return `true` when their respective method is available to the class and `false` when the method is not available.

This is indicative of the wisdom of SWE before us, who would never dare to use state to cause circular dependencies.

However, that's how Smart Contracts work, as the state is not just indicative of their current logical state, but also an hidden input for the functions.

Therefore we can extend EPAs to our Smart Contract use case by identifying all possible branches that a function can have, determining the conditions for said branches, and then labelling each of these possible possible function executions as if it were a distinct function.

With this technique, used by all whitebox formal tools, we have enumerate all branching conditions, reverting, non-reverting as well as assertion breaking conditions.

Solving for all of these is clearly a very different challenge (many time asymptotically supralinear time). However, it's important to emphasize that enumerating these is bounded by 2^M, where M are all the branching, reverting, non reverting and assertion breaking conditions.

Through this discussion we create a logical connection between EPAs and EPAs for Smart Contract.

I call EPAs for Smart Contracts Coverage Classes, because I think it's easier to understand but you are free to call them EPAs if you like.

## Coverage Classes




## External Coverage as part of our systems coverage

## Implied Lines

- msg.value causes balance to decrease, the code is in the node.

## Implied state

- `reentrant` state being the classic example

## Metaproperties

- Relation between 2 variables: [similar magnitude, 1 massively bigger than other]


## World splits

- Divisions causing no / negligible truncation
- Truncation causing significant invariants no longer holding
