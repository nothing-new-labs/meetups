- Topic: The Cascades Optimizer Framework Paper Reading
- Date: 2022-07-20

# Paper Reading: The Cascades Framework for Query Optimization

This papre describes a new extensible query optimization framework that resolves many of the shortcomings of the *EXODUS* and *Volvano optimizer generators*.

## Introduction

Cascades project: A new project applying many of the lessons learned from the Volcano project on extensible query optimization, parallel query execution, and physical database design.

Compared to the Volcano design and implementation, the new Cascades optimizer has the following advantages:
- Abstract interface classes defining the DBI-optimizer interface and permitting DBI-defined subclass hierarchies
- Rules as objects
- Facilities for schema- and even query-specific rules
- Simple rules requiring minimal DBI support
- Rules with substitutes consisting of a complex expression
- Rules that map an input pattern to a DBI-supplied function
- Rules to place property enforcers such as sort operations
- Operators that may be both logical and physical, e.g., predicates
- Patterns that match an entire subtree, e.g., a predicate
- Optimization tasks as data structures
- Incremental enumeration of equivalent logical expressions
- Guided or exhaustive search
- Ordering of moves by promise
- Rule-specific guidance
- Incremental improvement of estimated logical properties

## Optimization Algorithm and Tasks

The optimization algorithm is broken into several parts, which we call ”tasks”. We chose to realize tasks as objects that, among other methods, have a "perform" method defined for them. All task objects are collected in a LIFO stack. 

There are six types of tasks that make up the optimizer's search algorithm: 1) Optimize Group; 2) Optimize Expression; 3) Explore Group; 4) Explore Expression; 5) Apply Rules; 6) Optimize Inputs.

- Optimizing a group means finding the best plan for any expression in the group and therefore applies rules. Whereas optimizing an expression starts with a single expression. The former is realized by invoking the latter for each expression. The latter results in transitive rule applications and therefore, if the rule set is complete, finds the best plan within the starting expression'a group. The task to optimize a group also implements dynamic programming and memorization.

- Exploring a group or an expression is an entirely new concept. A group is explored using transformation rules only on demand, and it is explored only to create all members of the group that match a given pattern.

- Applying a rule creates a new expression; It may roughly be broken into four components. 
  - First,all bindings for the rule’s pattern are derived and iterated over one by one.
  - Second, for each binding, the rule is used to create a new expression.
  - Third, the new expressions are integrated in the ”memo” structure.
  - Fourth, each expression that is not a duplicate of an earlier one is optimized or explored with the same goal and context that triggered the current rule application.

## Data Abstraction and the User Interface

There are three guidelines to design the interface of the Cascade optimizer:
- clean abstractions for support functions in order to enable an optimizer generator to create them from specification.
- rule mechanisms that permit the DBI to choose rules or functions to manipulate operator arguments.
- more concise and complete interface specifications, both in the code and in the written documentation.

Each of the classes that make up the interface between the Cascades optimizer and the DBI is designed to become the root of a subclass hierarchy.  The optimizer relies only on the method defined in this interface; the DBI is free to add additional methods when defining subclasses.

### Operators and Their Arguments

- The ”class OP-ARG” in the Cascades optimizer interface includes both logical and physical operators. For each operator, method called ”is-logical/physical” indicates whether or not an operator is a logical/physical operator. In fact, it is possible that an operator is neither logical or physical. 
- The definition of operators includes their arguments.
- There are two special operators, called ”LEAF-OP” and ”TREE-OP”.
  - The leaf operator can be used as leaf in any rule; during matching, it matches any subtree.
  - The tree operator is like the leaf operator except that the extracted expression contains an entire expression, independent of its size or complexity, down to the leaf operators in the logical algebra.
- All operators must provide a method ”opt- cutoff”. Given a set of moves during an optimization task, this method determines how many of those will be pursued.
- For logical operators, there are methods for matching and hashing and methods for finding and improving logical properties.
- For physical operators, there are methods for determining an operator’s (physical) output properties, methods for computing and inspecting costs and methods for mapping the optimization goal for expression.

### Logical and Physical Properties, Costs

1. "class COST" is for anticipated execution costs and has a comparison method.
2. "class SYNTH-LOG-PROP" is for the encapsulation of logical properties and has a hash function.
3. "class SYNTH-PHYS-PROP" is for the encapsulation for physical properties and has no methods at all.
4. "class REQD-PHYS-PROP" is for the required physical properties and has only one method associated with it, which determines whether a synthesized physical property instance covers the required physical properties.

### Expression Trees

In order to communicate expressions between the DBI and the optimizer, another abstract data type is part of the interface, call the ”class EXPR.” Each instance of this class is a node in a tree, consisting of an operator and to pointers to input nodes.

Methods on an expression node, beyond constructor, destructor, and printing, include methods to extract the operator or one of the inputs as well as a matching method, which recursively traverses two expression trees and invokes the matching method for each node’s operator.

### Search Guidance

In addition to pattern, cost limits, and required and excluded physical properties, rule application can also controlled by heuristics represented by instances of the ”class GUIDANCE.”

### Pattern Memory

The purpose of the pattern memory is to prevent that the same group is explored unnecessarily. There is one instance of a pattern memory associated with each group. 

Beyond checking whether a given pattern already exists in the memory, and saving it to detect a second exploration with the same pattern, the most complex method for pattern memories is to merge two pattern memories into one. 

### Rules

Next to operators, the other important class of objects in the Cascades optimizer are rules. Notice that rules are objects; thus, new ones can be created at run-time, they can be printed.

All rules are instances of the ”class RULE,” which provides for rule name, an antecedent (the ”before” pattern), and a consequent (the substitute). 

In the EXODUS and Volcano optimizer generators, an implementation rule’s
substitute could not consist of more than a single implementation operator; in the Cascades design, this restriction has been removed. The remaining restriction is that all but the substitute’s top operator must be logical operators.

For more sophisticated rules, two types of condition functions are supported. All of them consider not only the rule but also the current optimization goal.
- Before exploration starts, ”promise” functions informs the optimizer how useful the rule might be.
- A ”condition” function checks whether a rule is truly applicable after exploration is complete and a complete set of operators corresponding to the pattern in the rule is available.

If a rule’s substitute consists of only a leaf operator, the rule is a reduction rule. If a reduction rule is applicable, two groups in the search memory will be merged.

There is an important class of situations in which expansion rules are useful, namely the insertion of physical operators that enforce or guarantee desired physical properties. Such rules may also be called enforcer rules. 

In some situations, it is easier to write a function that directly transforms an expression than to design and control a rule set for the same transformation. For those cases, the Cascades optimizer supports a second class of rules, called the ”class FUNCTION-RULE.” In the extreme case, a set of function rules could perform all query transformations, although that would defeat some of the Cascades framework’s purpose.
