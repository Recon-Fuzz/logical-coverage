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

## Intuition on Coverage

Given a stateless (doesn't use storage variables), a function coverage is a direct result of the parameters it is provided.

Given a stateful function (which uses storage variables), we can view coverage as the result of the parameters it is provided plus the current state.

It's fairly easy to see that given setter(variable) and function(variable, params), where setter is the only way to alter variable, the coverage of function is directly determine by the paramers of setter as well as the parameters it receives.

We can as such argue that some functions' Coverage Classes are influenced by other functions' Coverage Classes.

We can extend this argument by looking at contracts for which multiple functions can alter the same storage variable in various ways.

These tend to be the most complex functions to analyze.

We can view Logical Coverage as a set of paths that uses Coverage Classes to achieve the desired coverage.

My goal with this document is to define how to explore these paths and help determine which paths can be solved, vs which paths cannot be solved.

## Motivating Examples


- Morpho fuzzing

Current fuzz
                   

- Strictly increasing properties


## TODO

Most importantly we want to emphasize the distinction between how a Coverage Guided Fuzzer works, and a Logical Coverage Guided Fuzzer would work

Our goal is to separate Input Classes, and Coverage Classes as to ensure all of those get into the corpus

Notice how inputs belong to discrete sets, but may not necessarily have any other property (such as contiguity)
