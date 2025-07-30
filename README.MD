# Logical Coverage

The goal of this document is to discuss Logical Coverage, introduce new terminology to help analyze it, and suggest an algorithm to enumerate all meaningful state in a smart contract system.

This research has 2 goals:
- Given a Chimera like Fuzzing Setup, helping fuzzers reach coverage with zero user input (AutoClamp)
- Giving manual reviewers a better defined checklist to help them determine if they have reviewed all possible meaningful states for their audit

## Definitions

Given a smart contract and it's function we define Coverage as the line coverage that can be achieve by some input.

Given the concept of Coverage, we can take the set of all possible coverage results for a function.

We use the term Coverage Classes for the set of all these coverage results.

# Types of Coverage Classes

We can define 3 types of Coverage Classes

- Non Reverting
- Reverting
- Assertion Breaking

Assertion Breaking Coverage Classes are the ones we want to reach

Reverting Coverage Classes can be useful to identify DOS, and technically contain `Assertion Breaking` CCs.

It's worth noting that due to how we write fuzz tests (assertions as the oracle), we are mostly interested in:

- Non Reverting -> For state space exploration / generation of preconditions
- Assertion Breaking -> To trigger an assertion

This because for any Reverting Coverage Class, we can write an assertion that triggers that call as a Try/Catch

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

## Motivating Examples

### ERC4626 Linear Code

Given the deposit function for a ERC4626 Vault, we need to perform the following:

- `mint` some `asset` to self
- `approve` the token for `deposit`
- Call `deposit` with `min(allowance, balanceOf)`

A manual reviewer with any meaningful experience intuitively understands this.

A fuzzer will waste (1/ handlers) possibly classing the `deposit` function, and would require millions of requests before performing a deposit.

Similar reasoning for `withdraw` / `redeem`, in lack of tokens, any call to these functions is completely unnecessary.

This type of linear relation is very simple to express and solve for solvers, implying that our fuzzers can only benefit by applying more formal methods.

### Morpho fuzzing

Fuzzers don't currently have an easy way to handle structs, this is arguably a limitation of how their corpus is generated.

However, in practice this limitation makes it so that even fuzzing Morpho (600 loc self contained contract), requires an unreasonably long amount of time.

The key issue with fuzzing Morpho is that the fuzzer doesn't seem to recognize that once it deploys a market, which results in a specific `marketId`, it's forced to continue fuzzing for that specific `marketId`, as all other markets won't be available to it.

### Convex Optimization Functions

Many optimization properties can be viewed as trying to maximize an unknown convex optimization function

Convex functions have the property of having only on global minima and have monotonicity around it.

These functions can be fairly easily solved by modern solvers, which leads us to expect that fuzzers should have a easier time in resolving them.


# Dependencies (TODO: NOT FINISHED)

- Direct Dependency (Gates)
- Direct Linear Dependency (Paths)
- Complex Linear Dependency (Bridges)
- Cyclic Linear Dependency

- Convex Dependency (can this be direct or non direct?)

- Exact Dependency (cryptographically unfeasible vs dictionary based)


# Branching Idea

Given some functions we can take all the possible branches in the code

Given those branches we can argue each branch that has no impact on other function is a terminal leaf in a decision tree (TODO: Is this the right data structure)

Given each function we can take each function that requires no particular state to work (meaning there exist some inputs for which the function won't revert), and we can consider these the start of our decision tree.

# Prerequisites for Coverage

Given a function that has no state nor external calls, we can say that it has no prerequisite to achive all possible coverage classes.

Given a function that has state and external calls, we need to analyze the possible storage values that can be achieve by the external calls and other functions in the contract.

## Storage preconditions

We can call those the storage preconditions

## Input Classes

Given a set of storage preconditions (finite by definition)

A corresponding set of Inputs can be identified for each Coverage Class that is feasible given a entry for the storage precondition

[Storage Preconditions, [Input Classes]]: Coverage Classes

The union of all [Storage Preconditions, [Input Classes]] will result in all feasible Coverage Classes

All Coverage Classes that don't belong to the set are unfeasible.


# First Level Approximations

Given a function, determine the relevant coverage classes.
Given storage, determine the bounds (approximate them by excess)
Generate [Storage Preconditions, [Input Classes]]
At each point during a Concrete Run, clamp inputs around [Input Classes] for the function as we have already determine that all calls that don't belong are wasted.

TODO: Benchmark efficacy of this, how many calls can we save?
This is probably a win, at all times.

-> Symbolic CFG? Symbolic Coverage Classes?


-> Code -> AST -> Coverage Classes -> Solved | Partial Concrete vs Fully Symbolic.

# Second Level Approximations

Separate properties by storage slot influence

Then apply first level approximation

This allows skipping functions and operations which are irrelevant for a given property

This also allows speeding up shrinking, as ultimately we already know there are no relevant operations


# TODO

- External Calls
- Storage Loops / Dependencies -> Path to get to Storage Coverage Class





