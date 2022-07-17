# Paper Reading: The Volcano Optimizer Generator: Extensibility and Efficient Search

- A new optimizer generator: **Data model, logical algebra, physical algebra, and optimization rules** are translated by the optimizer generator into optimizer source code.
- The search engine is more extensible and powerful.
  - It provides effective support for non-trivial cost models and for physical properties such as sort order.
  - It is much more efficient as it combines dynamic programming.

## Introduction

Requirements:

1. extensible and modular
2. efficient: both in optimization time and in memory consumption for the search
3. provide effective, efficient, and extensible support for physical properties
4. permit use of heuristics and data model semantics to guide the search and to prune futile parts of the search space
5. support flexible cost models that permit generating dynamic plans for incompletely specified queries

## The Outside View of the Volcano Optimizer Generator

The optimizer generator paradigm:

```text
        Model Specification
                |
                | Optimizer Generator
                V
        Optimizer Source Code
                |
                | Compiler and Linker
                V
            Optimizer
Query --------------------> Plan
```

### Design Principles

1. use two algebras, called the logical and the physical algebras, and generate optimizer that map an expression of the logical algebra into an expression of the physical algebra. To do so, it uses transformations within the logical algebra and cost-based mapping of logical operators to algorithms.

2. rules have been identified as a general concept to specify knowledge about patterns in a concise and modular fashion, and knowledge of algebraic laws as required for equivalence transformations in query optimization can easily be expressed using patterns and rules.

3. the choices that the query optimizer can make to map a query into an optimal equivalent query evaluation plan are represented as algebraic equivalences in the Volcano optimizer generator's input.

4. rule compilation (vs. interpretation)

5. dynamic programming

### Optimizer Generator Input and Optimizer Operation

- an algrbra expression (tree) of logical operators
  - declared in the model specification
  - operators can have zero or more inputs; the number of inputs is not restricted
  - the output of the optimizer is a plan, which is an expression over the algrbra of algorithms

- The algebraic rules of expression equivalence are specified using *transformation rules*

- The possible mappings of operators to algorithms are specified using *implementation rules*

- The results of expressions are described using properties.
  - Logical properties can be derived from the logical algebra expression and inclued schema, expected size, etc.
  - Physical properties depend on algorithms, e.g. sort order, partitioning, etc.

- Enforcer: enforce physical properties in their inputs that are required for subsequent query processing algorithms.

To summarize this section, the optimizer implementor provides:

1. a set of logical operators
2. algebraic transformation rules, possibly with condition code
3. a set of algorithms and enforcers
4. implementation rules, possibly with condition code
5. and ADT "cost" with functions for basic arithmetic and comparsion
6. and ADT "logical properties"
7. and ADT "physical properties vector" including comparisons functions (equality and cover)
8. an applicability function for each algorithm and enforcer
9. a cost function for each algorithm and enforcer
10. a property function for each operator, algorithm, enforcer

## The Search Engine

- use dynamic programming and to be very goal-opiented, i.e., driven by needs rather than by possibilities.

- In order to prevent redundant optimization effort by detecting redundant (i.e., multiple equivalent) derivations of the same logical expressions and plans during optimization, expression and plans are captured in a hash table of expressions and equivalence classes.

- FindBestPlan
  - three sets of possible "moves" the optimizer can explore at any point
    - applicable transformations
    - algorithms that give the required physical properties
    - enforcers for required physical properties
  - After all possible moves have been generated and assessed, the most promising moves are pursued.
  - The cost limit is used to improve the search algorithm using branch-and-bound pruning.

## Comparsion with the EXODUS Optimizer Generator

### Functionality and Extensibility

- Volcano makes a distinction between logical expressions and physical expressions.
- The ability to specify required physical properties and let these properties, together with the logical expression, has contributed significantly to the efficiency of the Volcano optimizer generator search engine.
- The Volcano algorithm is driven top-down; subexpressions are optimized only if warranted.
- In Volcano, cost is an abstract data type for which all calculations and comparisons are performed by invoking functions provided by the optimizer implementor.
- The Volcano optimizer generator is more extensible than the EXODUS prototype, in particular with respect to the search strategy.