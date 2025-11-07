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

### Reflection on Coverage Classes and manual review.

It stands within reason that a manual reviewer that can prove they have systematically analyzed all coverage classes for all functions as well as all combinations between feasible calls (or a select list of those that would be considered dangerous), has performed a statisfactory review of the codebase.

The interesting aspect about this, is how experience in manual reviews will lead auditors to naturally review the most complex or difficult to reach coverage classes, as well as define Meta Properties, which I'll discuss below.

My conclusion is that we as manual reviewers are able to handle a insane amount of combinatorial complexity, by simplifying it to a level that is manageable, and the best manual reviewers are able to see some Coverage Classes that most other people would miss.

Therefore experience is the ability to apply a heuristic to under represent the system, by focusing on Coverage Classes that are prioritized due to their higher perceived risk and likelyhood of causing a bug.

## External Coverage as part of our systems coverage

Being a Auditor that uses Stateful Fuzzing daily, I have found it useful to extend coverage to not only the code the customer wrote, but also the code that is being consumed as well as the code from mocks or simplifications that we generated.

Having defined Coverage Classes above, we can apply the same line of thinking for a set of gotchas that most manual reviewers can quickly identify that are not easily measured by applying coverage metrics only on the target contracts.

The below considerations are incomplete, subject to change and (by definition? TODO) underrepresent the possible ways in which we can analyze a system of smart contracts.

They are useful because they shed light into some common bugs and by generalizing them we can avoid missing these.

### Actionable Insight

Visualizing the Coverage Classes, enumerating them, checking the list is a way to say "I'm done with the review".

## Implied Lines

Smart Contracts are executed many levels above a CPU, even assembly implies chip operations, which are abstracted away and can result in bugs.

For our purpose, implied lines are lines that exist in some part of the code that interacts with our SUT.

This is to note that these lines of code exist, they don't require any imagination nor the creation of a meta level abstraction.

The most basic example is `msg.value` (e.g. using `msg.value` in a loop) as well as `USDT` not working without low level assembly.

These extremely common bugs are explainable by the fact that a average tester wouldn't know to test for that specific code.

On one hand `msg.value` is code that is handled and present exclusively in the EVM Node, covering it would require instrumenting the node.

On the other hand, `USDT` compatibility is a specific instance of the "arbitrary address / inconsistent behaviour" bug, which is a result of our human inability to account for all scenarios when we use the equivalent of `*` for operations.

### Examples

- `msg.value` causes balance to decrease, the code is in the node.
- Precompile actual code and behaviour, as well as their mere existence
- Different Token Behaviour on Transfer
- Calling / Delegate calling to hardcoded addresses with any parameter
- Calling / Delegate calling to any address

### Notable hacks

- Precompile signature skip bug -> TODO Some wallet
- Parity hack
- Every unsanitized input that doesn't imply math, e.g. ability to become owner, mint / burn.

### Actionable insight

A Smart Contract SUT that presents implied lines should be reviewed with an explicit list of Integrations, and Targets tested.

Project developers should refraing from allowing a `*` to be introduced in the system as it fundamentally makes it impossible to review a SUT with the same rigor.

Successful projects tend to list out the exact integrations they will use, and smart researchers will review all the integrations for gotchas and bugs.

My point is that a review should be done under proper constraints, doing so makes it much easier to reason around the system and overall makes the system much safer as well as the code simpler due not having to account for extremely complex combinations.

## Implied state

TODO | These are states and code that doesn't exist. But that semantically makes sense, and can be easily be represented by a flag of some sort.

The most common example is `reentrant`, while many codebases don't use the `nonReentrant` modifier for well reasoned ways, it's very obvious in hindsight how a contract, in a reentrant context is very different from the same contract in  non reentrant state.

Another interesting example is dealing with donations as well as tokens rebase or have inherent rounding errors, these introduce an implicit state that can be represented by the difference between the values recorded in storage and the values recorded by the token.

Wisdom is being able to identify these patterns and account for them.

However, I believe that after this research, we should collectively be building tools or finding techniques to automatically track for these, flagging reentrancy is not particularly useful in the way it's been done so far, whereas giving an auditor a checklist of states that are implied, in a more explicit way, can help identify these issues earlier.

### Examples

- `reentrant` state being the classic example
- Different token balance tracking, and inconsistency with recorded state. (A bit of stretch)
- Solvent, Insolvent | Liquidatable | Liquidated -> (stretch)

## Metaproperties

Are the definition of `iykyk`, they can't be easily represented as some basic missing flag in the code, however expert Invariant Testers write these very often.

A meta property is simply a property of the system, that is not part of the SUT.

This definition captures everything that may be missed in any form of review, and due to this we're forced to underapproximate the total number of properties that we could identify.

In spite of this, I chose to separate MetaProperties, which most often can be written as simple stateless assertions about the system and sometimes can be written with ghost variables supporting them, from World Splits, which I believe is a completely underexplored area of Invariant Testing, and tends to yield the vast majority of mathematically based exploits.

For any variable, alteration of the variables, etc.. you can write a meta property.

Fundamentally these are the Invariants we and others write.

Reaching all possible Coverage Classes of coverage of the meta properties (LIKELY because union = overestimated) requires covering the union between the original systems Coverage Classes, as well as the set of constraints that are implied by the Meta Properties.

### Examples

- Relation between 2 variables: [similar magnitude, 1 massively bigger than other]

## World Splits

I created this category to emphasize it's complexity as well as net distinction between the behaviours we observe when dealing with a "well behaved" system, in contrast to us hitting the World Split on the system.

In reality all math in our domain is not in a group, meaning it's never well behaved as we'd expect it to, but in practice in most cases it is, and as such I created the idea of World Splits to signal when something completely unintuive happens to the mathematical behaviour of the SUT.

A very interesting example if how given certain values and ratios a 1 wei rounding error can be magnified into a massive error.

Let's examine this formula:
```solidity
uint256 sharesToIssue = amt * totalSupply // totalAssets; /// @audit // to emphasize this is integer division
```

Because (ℤ, //) is not a group, we don't have the reversibility of operations as a guarantee.

This means that every division can cause a step-wise change in the relation between the variables in a way that is non-reversible.

The classic example to this is the donation attack, where a first depositor donates a bunch of assets to a vault with the goal of causing the `sharesToIssue` to lock an immediately loss to future depositors.

The attack has been explored in many ways, and can sometimes be escalated to an implied donation attack, by abusing other rounding errors in the system.

### Examples

- Divisions causing no / negligible truncation
- Truncation causing significant invariants no longer holding

### Notable Bugs

- Bold bug I found in my report
- Euler V1
- Degenbox with one function to allow repay

## References

1. https://dl.acm.org/doi/10.1145/3415153
