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

Enabledness Preserving Abstractions [1] is a technique to enumerate program states, the paper referenced uses execution to explore and document the states producing an underapproximation of the system.

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

Given our definition of Coverage Classes.

We call a CC Feasible if there exist some combination of inputs such that the function will execute the coverage that defines that class. That is the current state of the contract is such that the CC can be executed.

It's worth noting that solving for this in non trivial, however, in practice, detecting the available coverage classes is most of the times possible for each given contract state. Meaning that while we are computationally unable to compute and define relations between state and functions that would allow us to automatically define all the possible trasitions, we can at any time define the current transitions available to us. And we can then map them out to all the Coverage Classes. Meaning that in practice we can now, when we are done, even though we can't predict the difficulty of being done until we have done the work.

(SOLVER can time out, but most of the times it won't if we do it at run time).

This gives us a baseline for enumerating all coverage based combinations for a smart contract.

Following the same technique in [1] we can detect Coverage Classes transition, as I said in whitebox environment we already have access to the source and therefore all the possible classes, however, we may not have the list of all feasible transitions, therefore we will underpresent it following the technique in [1] in which we store a valid new transition whenever this happen in our testing.

### Important simplifcation of Coverage Classes in the context of Assertion Testing. | TODO: Does this even matter?

We're going to be discussing MetaProperties, as properties that "cut" a contract in a different way.

Unclear if enumerable or not: TODO: Find is math properties.

### Reflection on Coverage Classes and manual review.

It stands within reason that a manual reviewer that can prove they have systematically analyzed all coverage classes for all functions as well as all combinations between feasible calls (or a select list of those that would be considered dangerous), has performed a statisfactory review of the codebase.

The interesting aspect about this, is how experience in manual reviews will lead auditors to naturally review the most complex or difficult to reach coverage classes, as well as define Meta Properties, which I'll discuss below.

My conclusion is that we as manual reviewers are able to handle a insane amount of combinatorial complexity, by simplifying it to a level that is manageable, and the best manual reviewers are able to see some Coverage Classes that most other people would miss.

Therefore experience is the ability to apply a heuristic to under represent the system, by focusing on Coverage Classes that are prioritized due to their higher perceived risk and likelyhood of causing a bug.

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



## References

1. https://dl.acm.org/doi/10.1145/3415153
