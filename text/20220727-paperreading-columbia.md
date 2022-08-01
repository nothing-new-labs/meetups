- Topic: The Columbia Optimizer Paper Reading
- Date: 2022-07-27

# EFFICIENCY IN THE COLUMBIA DATABASE QUERY OPTIMIZER

## Terminology

- Logical Operator
  - High-level operators that specify data transformations without specifying the physical execution algorithms to be used.
  - Each logical operator takes a fixed number of inputs and may have parameters that distinguish the variant of an operator.

- Query Tree
  - a query tree is represented as a tree of logical operators in which each node is a logical operator having zero or more logical operators as its inputs.

- Physical Operator
  - Physical Operators represent specific algorithms that implement particular database operations.
  - One or more physical execution algorithms can be used in a database for implementing a given query logical operator.

- Execution Plan
  - Replacing the logical operators in a query tree by the physical operators which can implement them gives rise to a tree of physical operators which is called an Execution Plan.
  - Each plan has an execution cost corresponding to the cost model and catalog information.

- Expression
  - A given query can be represented by one or another query tree that is logically equivalent.
  - Two query trees are logically equivalent if they output exactly the same result for any population of the database.
  - We can also use expressions to represent query trees and execution plans (or sub trees and sub plans).
  - An expression consists of an operator plus zero or more input expressions.
  - Given a logical expression, there are a number of logically equivalent logical and physical expressions.

- Group
  - A Group is a set of logically equivalent expressions.
  - In general, a group will contain all equivalent logical forms of an expression, plus all physical expressions derivable based on selecting allowable physical operators for the corresponding logical forms.

- Multi-expression
  - A Multi-expression consists of a logical or physical operator and takes
groups as inputs.
  - The advantage of multi-expressions is the great savings in
space because there will be fewer equivalent multi-expressions in a group.

- Logical Property
  - The Logical properties of a group are defined as the logical properties of the result, regardless of how the result is physically computed and organized.
  - These properties include the cardinality (number of tuples), the schema, and other properties.
  - Logical properties apply to all expressions in a group.

- Search Space
  - The search space represents logical query trees and physical plans for a given initial query.
  - In the initial search space, each group includes only one logical expression, which came from the initial query tree.

- Rules
  - A rule is a description of how to transform an expression to a logically equivalent expression.
  - A new expression is generated when a rule is applied to a given expression.
  - Each rule is defined as a pair of pattern and substitute. A pattern defines the structure of the logical expression that can be applied to the rule. A substitute defines the structure of the result after applying the rule.
