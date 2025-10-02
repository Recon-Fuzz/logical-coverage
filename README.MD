# Logical Coverage

The goal of this document is to discuss Logical Coverage, introduce new terminology to help analyze it, and suggest an algorithm to enumerate all meaningful state in a smart contract system.

This research has 2 goals:
- Given a Chimera like Fuzzing Setup, helping fuzzers reach coverage with zero user input (AutoClamp)
- Giving manual reviewers a better defined checklist to help them determine if they have reviewed all possible meaningful states for their audit

## Motivating Examples

### ERC4626 Linear Code

Given the deposit function for a ERC4626 Vault, we need to perform the following:

- `mint` some `asset` to self
- `approve` the token for `deposit`
- Call `deposit` with `min(allowance, balanceOf)`

A manual reviewer with any meaningful experience intuitively understands this.

A fuzzer will waste (1/ handlers) possibly classing the `deposit` function, and would require many requests before performing a deposit.

Similar reasoning for `withdraw` / `redeem`, in lack of tokens, any call to these functions is completely unnecessary.

This type of linear relation is very simple to express and solve for solvers, implying that our fuzzers can only benefit by applying more formal methods.

An ideal fuzzer would recognize the key relation between `mint`, `approve` and `deposit` (and others) as well as the linear relation, giving it the ability to reach coverage in just a few calls, as well as to maximize the values for `deposit` by optimizing that linear relation.

### Morpho fuzzing

Fuzzers don't currently have an easy way to handle structs, this is arguably a limitation of how their corpus is generated.

However, in practice this limitation makes it so that even fuzzing Morpho (600 loc self contained contract), requires an unreasonably long amount of time.

The key issue with fuzzing Morpho is that the fuzzer doesn't seem to recognize that once it deploys a market, which results in a specific `marketId`, it's forced to continue fuzzing for that specific `marketId`, as all other markets won't be available to it.

In lack of using an enabled `marketId` the fuzzer cannot `supply` nor `deposit` let alone `liquidate` positions. The `marketId` is a simple yet hard to surpass gate for the fuzzer to reach coverage in any meaningful way.

An ideal fuzzer would recognize the relation between a parameter and a storage value, note that the storage value enables new levels of coverage, then proceed the function call to enable the market as the start node for many of it's sequences.

### Convex Optimization Functions

Many optimization properties can be viewed as trying to maximize an unknown convex optimization function.

Convex functions have the property of having only on global minima and have monotonicity around it.

These functions can be fairly easily solved by modern solvers, which leads us to expect that fuzzers should have a easier time in resolving them.

An ideal fuzzer will recognize these "Convex Chains" and loop over them with the goal of unlocking a new region of feasible combinations.

## Definitions

Given a smart contract and its functions we define Coverage as the line coverage that can be achieve by some input.

Given the concept of Coverage, we take the set of all possible coverage results for a function.

We use the term Coverage Classes for the set of all these coverage results.

While inputs to a call can be different, we can view them as belonging to the same Input Class if they result in the same Coverage.

# Types of Coverage Classes

We can define 3 types of Coverage Classes:

- Non Reverting
- Reverting
- Assertion Breaking

Assertion Breaking Coverage Classes are the ones we want to reach.

Reverting Coverage Classes can be useful to identify DOS, and technically contain `Assertion Breaking` CCs.

It's worth noting that due to how we write fuzz tests (assertions as the oracle), we are mostly interested in:

- Non Reverting -> For state space exploration / generation of preconditions
- Assertion Breaking -> To trigger an assertion

This because for any Reverting Coverage Class, we can write an assertion that triggers that call as a Try/Catch.

However, given this fact, we can look at all Reverting Coverage classes directly, and save ourselves the need for a Meta Property.

Given each Coverage Class there's a corresponding Set of Inputs that will allow us to reach that coverage. Note that the Set can be empty for classes that are not feasible.

Given that some Coverage Classes require some specific precondition, the Set of Inputs is best categorized as a Sequence of Sets of Inputs that will allow the Coverage Class to be reached.

## Intuition on Coverage

Given a stateless function (doesn't use storage variables), a function coverage is a direct result of the parameters it is provided. That is we can enumerate all the paths based on the relation between it's logic and the function parameters.

Given a stateful function (which uses storage variables), we can view coverage as the result of the parameters it is provided plus the current state.

It's fairly easy to see that given a `setter(variable)` and `function(variable, params)`, where setter is the only way to alter variable, the coverage of `function` is directly determined by the input of `setter` as well as the inputs it receives.

We can as such argue that some functions' Coverage Classes are influenced by other functions' Coverage Classes.

We can extend this argument by looking at contracts for which multiple functions can alter the same storage variable in various ways.

These tend to be the most complex functions to analyze.

We can view Logical Coverage as a set of paths that uses Coverage Classes to achieve the desired coverage.

My goal with this document is to define how to explore these paths and help determine which paths can be solved, vs which paths cannot be solved.

# Definition

Given a function, and its Coverage Classes.

We define Feasible Coverage Classes, as the Coverage Classes for which there exists a set of Coverage Classes for some functions [f1, f2,...fn] such that the Storage Precondition for these Coverage Classes can be achieved.

Starting from every Function as a starting point, we define Logical Coverage as the sequences of all calls that maximally explore all the Coverage Classes that are influenced by the storage slots that influence and are influenced by the starting Function.

It stands to reason that any Security Reviewer will want to visualize the Coverage Classes as well as many parts of the Logical Coverage.

# Branching Idea

Given a list of smart contracts and their functions.

Functions that don't read nor write state can be viewed as edges without vertices in a graph.

Functions that read state have vertices stemming from the functions that influence said state.

Functions that write to state have vertices originating from them and pointing to the functions that consume the state.

It's intuitive to think that the graph will lead to cycles anytime a function consumes a variable it reads, as well as anytime functions influence each other (both read and write).

At this point in the analysis we cannot say much about these functions, except for the intuition that while a cycle of funtion calls may be identified, that cycle may still resolve, or approximate, to a convex function, meaning in aggregate it's still a manageable complexity for formal techniques.

# Prerequisites for Coverage

Given a function that has no state nor external calls, we can say that it has no prerequisite to achive all possible coverage classes.

Given a function that has state and external calls, we need to analyze the possible storage values that can be achieve by the external calls and other functions in the contract.

## Storage preconditions

We can call those the storage preconditions.

We can determine an overestimation of all possible Coverage Classes by allowing the storage value to have all possible symbolic values within it's type range.

The overestimation allows to quickly determine all interesting storage preconditions for a given function (as it massively reduces path dependence).

## Input Classes

Given a set of storage preconditions (finite by definition), a corresponding set of Inputs can be identified for each Coverage Class that is enabled by the storage precondition.

[Storage Preconditions, [Input Classes]]: Coverage Classes.

The union of all [Storage Preconditions, [Input Classes]] will result in all Coverage Classes (Feasible + Unfeasible).

If we were able to determine all feasible Storage Preconditions, and created this set of [Feasible Storage Preconditions, [Input Classes], we'd be left with only Feasible Coverage Classes. Meaning our Logical Coverage (the set) is smaller than the set of all possible Coverage Coverage Classes.


# Practical Ideas [WIP]
## First Level Approximations

Given a function, determine the relevant coverage classes.
Given storage, determine the bounds (approximate them by excess).
Generate [Storage Preconditions, [Input Classes]].
At each point during a Concrete Run, clamp inputs around [Input Classes] for the function as we have already determine that all calls that don't belong are wasted.

## Second Level Approximations

Separate properties by storage slot influence.

Then apply first level approximation.

This allows skipping functions and operations which are irrelevant for a given property.

This also allows speeding up shrinking, as ultimately we already know there are no relevant operations.

I expect most fuzzers should use this strategy to break specific properties.

## Third Level Approximation

Given a specific Coverage Class, determine the calls to setters that would end up causing the Storage Preconditions to be valid.

## Fuzzers Implementation

All of the operations described above are akin to "lightmaps baking", we're preprocessing the smart contract paths to determine which paths are feasible.

### Union

Given the current concrete state, it is trivial to check which Storage Preconditions we satisfy.

Given the starting state, we can explore paths that will lead us to the Storage Preconditions, some of these are likely to timeout due to complexity.

Given the new states, after applying Corpus Transformation, if we don't recognize the current Concrete State in one of the "solved" Storage Preconditions, we know the fuzzer has found a path to a Storage Precondition for which we had timeouts.

After having reached this state, we can apply the same algorithm.

# Expected changes in fuzzers

I don't expect fuzzers to work by having a consistent corpus size and then mutating it.

I believe fuzzers will instead generate basic corpus that is based on the simplest to reach coverage classes.

More specifically we can take inspiration from LLM training to help the fuzzer reduce the amount of operations it wastes.

## SMT Solving [TODO: Exact inputs, Coverage Classes, Unlocking next coverage systematically)

SMT Solving with concrete bounds will help speed up the fuzzer.

Anytime a element belongs to the same Coverage Class, it may not be added to the corpus (I believe there are edge cases tied to optimization mode vs reaching full coverage here)

Using Coverage Classes instead of random inputs should also allow us to run the SMT solver on the given inputs and then store those results as part of the corpus.

This should be feasible because running SMT queries on concretized values is massively faster than providing the SMT solver with all the branches.

## Dependency List 

Even in lack of any sophisticated SMT Solving, the mere notion of Storage Preconditions allows us to devise a more appropriate algorithm for chosing starting points from which to build our corpus as well as a more nuanced algorithm for chosing which function the fuzzer should call.

After a first scan for overapproximated Coverage Classes for all functions.

We can run a very efficient AST analysis to identify the dependency list for storage slots.

A simple algorithm would be:
- For each function, for each storage slot
- Add to "depends on" if the function reads the storage slot
- Adds the "is precondition" if the function writes the storage slot

NOTE:
We can further refine this by also listing the relation (which could be as simple as: [from input, linear, complex]), this would result in a lot of interesting strategies to improve performance.

Off of our dependency list, given a specific property we'd:
- Determine the storage slots of the property we're trying to break
- Determine the functions that influence those storage slots (preconditions)
- Start from functions that are not influenced by anything
- Add based on storage preconditions, a set of functions that could be called, with their coverage classes

This massively reduces the function calls down to a more efficient subset.

## TODO: Coverage Independent Partitions 
Parts of the Coverage that are not dependent from each other -> Basically different systems

# Dependencies [TODO]

Below I list an intuition for certain relations that should over time be solved very quickly by fuzzers.

This is mostly WIP.

- Direct Dependency (Gates)

A boolean that needs to be true for something to happen can be viewed as a gate for a specific set of coverage classes

Those classes (or entire functions) can be ignored whenever we have a direct dependency, until the value is changed.

Very common examples:
- Paused
- Initialized
- Operation Disabled
- Operating on a Market ID

- Direct Linear Dependency (Paths)

Certain operation consume other values linearly, these are limited by the min([VALUES])

Very common examples:
- Allowances
- Balances

- Cyclic Linear Dependency (Roads)

A linear dependency may not be directly expressed through the relation of a function to another.

However a linear dependency can be emergent from the combination of multiple function calls.

A very commmon example: The balance of an ERC4626 would be tied to the balance of asset, allowance of asset to the vault, and the call to mint.

Therefore we can fairly easily see (easily for us, most likely not easy for a machine), that the `redeem` function is linearly dependent on the sequence of calls to [mint, approve, deposit)

And in lack of those calls, calling `redeem` is a waste of cpu cycles.

- Convex Dependency (can this be direct or non direct?)

A Convex Dependency is the general case of linear dependency

Where linear dependency is easy to plot and understand immediately, a Convex dependency represent some monotonic relation between a sequence of functions and a specific target function

Very common example:
- Reaching a liquidation threshold through a Utilization Based IRM

The matematical equation to express the utilization would be a sequence of polynomial, the interest formula would be an exponential formula, we may not necessarily be able to compute an exact root for the expression, however we would be able to know that over time the values increase following a geometric series, therefore we would be able to solve for many optimization issues.

(Note that an exact property may be harder to solve, however, because we should be able to express a Convex Dependency, we should be able to simply iterate over the values via Newton Raphson, and find an approximation that breaks the property or is close to that).

- Exact Dependency (cryptographically unfeasible vs dictionary based)

It would be dishonest for me to ignore Exact Dependencies, meaning the requirement of the fuzzer to generate an exact value.

Having the fuzzer generate an exact state with a specific value is non trivial and in many cases may not be computationally feasible.

Formal methods would help identify these, and when combined with the usage of dictionaries, this should give the fuzzer the best chance to identify a counter example.


---

All of the above leads us to 100% coverage.

I thought our discussion was about going above that.

So here we go.

In discussing the above we have discussed strategies to explore state space, and arguably we have found a way (or an intuition to cover all of it by covering their classes).

So now we have to go past 100%.

- The ratio of the values in a line / operation could be different
- The operation could cause precision loss
- The operation may not be idempotent
- The operation may be inversely related (opening and closing gives you more assets, instead of less).

I have a strong belief that we do not have "an infinite amount of options", however we have many
