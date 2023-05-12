# `Syspec` language guide

`Syspec` implements set-theory concepts in plaintext. Symbols are implemented using keywords (like all other programming languages). Because this language is intended for capturing design information it explicitly avoids adding specific programming constructs like conditional branching (`if`, `else`, `switch`, etc.).

## Comments

Only single line comments are supported. Each comment line must start with a `#` followed by one or more spaces and then the comment string.

```
# This is a comment
```

## Packages and Importing

When specifying large designs, it is a good practice to split them across files and packages.

### Packages

All files that are part of the same package must be present in a single directory/folder. Additionally the first line of the file must include `package <name>` to indicate the package that file is part of. Subpackages can be specified in sub-directories. Sub-package relationship is automatically inferred using the directory structure.

### Importing

Imports can used to import specifications from other packages using `import "<package-name>"`. Multiple packages can be imported using the syntax `import {"<package-1>", "package-2"}`.

## Built-in datatypes

Language provides certain built-in datatypes that can be used to build types specific to your design

* `{}` represents any set. It is also used to imply `null` set.
* `int` is the set of all positive and negative integers. It is equivalent to the set `Z` from set-theory.
* `nat` is the set of all positive integers. It is equivalent to the set `N` from set theory.
* `real` is the set of allr real numbers. IT is equivalent to the set `R` from set theory.
* `bool` is the boolean set. It only has two members `true` and `false`.
* `string` is an unbounded set of all strings, of all lengths. Each string is a set of `char` values terminated by a `\0` character. All strings are enclosed in double quotes. For example `"string"`.
* `char` is a set of all UTF-8 characters. All characters are enclosed in single quotes. For example `'c'`.
* `date` is a set of all dates represented as `YYYYMMDD`.
* `time` is a set of all timestamps represented as `hhmmss.SSS` (`SSS` represent milliseconds).
* `datetime` is a set of all timestamps represented as ISO-8601 string - `YYYY-MM-DDTHH:MM:SS±HH:MM`. For UTC `±HH:MM` is replaced by `Z`.
* `uuid` is the unbounded set of all UUIDs
* `url` is the unbounded set of all URLs as specified by IETF (https://url.spec.whatwg.org).

## Set specification

Sets can be specified in two ways either by 
* Listing its members - `S := {'a', 'b', 'c'}`
* By specifying its *form* - `S := {<shape>:<type-specification>;<predicate>}`. 
  * `<shape>` specifies the *shape* of the elements in that set. `<shape>` can also be a regex enclosed between back-ticks.
  * `<type-specification>` provides the source set (or from one of the built-in types) from which the elements are sourced. 
  * `<predicate>` provides the rules that must be satisfied by each element of the set. Predicates must be specified using the `syspec` syntax.

Examples - 
* `A = {B, C, S}` defines a set containing 3 elements (which can themselves be sets).
* `B = {n: n in nat; n >= 1 and n <= 100}` defines a set of first 100 natural numbers.
* `C = {x = n*n; n in nat; x >= 1 and x <= 100}` defines a set of squares between 1 and 100. In the shape specification - `x = n*n`, the left-hand-side of the equation represents the set element and right-hand-side represents what needs to be done to determine the element.
* `S := {}` is a null set.

> NOTE: When `:=` is used, it means RHS elaborates LHS. Wherever LHS appears, RHS is processed within the context where LHS appears and the result is inserted in place of LHS.
> When `=` is used, it means all occurances of LHS is replaced by RHS, verbatim. Here RHS is copied first and then the whole expression is processed.

Tuples are specified by enclosing members in square brackets. For example `[a, b, c]` is a 3-tuple.

## Relations

Relations can be specified in three ways -

* **Set specification** - A set specification preceeded by `relation` keyword is a relation. The `shape` part of the specification must have the form `<domain> -> <range>`. If not, it is a syntax error. Domain and range can be made of a single type or both can be tuples. Example - `relation S := {[a,b] -> [c,d]:a in int, b in nat, c in int, d in bool; whole(a) == b && c = fraction(a) && d = (c > 0)}`.
* **Shortcut** - A shorter syntax can be used to simplify specification - `relation <name>(<param1> <input-type1>, <param2> <input-type2>, ...) -> (<param3> <return-type1>, <param4> <return-type2>) := {<predicates>}`. For example, the `S` relation can be rewritten as `relation S(a int, b int) -> (c int, d bool) := {whole(a) == b && c = fraction(a) && d = (c > 0)}`.

When specifying the body of the relation, instead of using `&&` to link predicates, each predicate can be specified on a separate line. The example for `S` above can also be specified as -

```
relation S(a int, b int) -> (c int, d bool) := {
  whole(a) == b
  c = fraction(a)
  d = (c > 0)
}
```

## Functions

Functions are specified using the relation syntax except that the keyword would be `function` instead of `relation`. Functions have the additional constraint that they must be deterministic i.e., an element of domain should have a distinct element in range. 
Example - `function Square := {a -> b:a in int, b in int; b = a*a}`. The *shortcut* version would be `func Square(a int) -> (b int) := {b = a*a}`. Multiline predicates are also supported.

## Operators

Following are the list of pre-defined operators
* **Unary operators**
  - `!` is used to negate a boolean, 
  - A power set is defined by putting a `^` before a set identifier (for example `^A`).
* **Binary operators**
  - There are no binary operators for `bool`.
  - Comparison operators - `==`, `>`, `<`, `>=`, `<=` and `!=`, can accept sets or elements. 
    - When applied to elements, their classic meaning (of comparing values) is used.
    - When applied to sets they mean `same`, `proper superset`, `proper subset`, `superset`, `subset` and `not same` respectively.
  - `\` means set-difference
  - `*` means cartesian product of sets. `*` retains its classic meaning (i.e., *multiplication*) when applied to elements that use one of the base datatypes.
  - `+` and `-` mean addition and subtraction respectively for `int`, `nat`, `real`, `date`, `time` and `datetime`. For `string` only `+` is supported and it means "string concatenation" as well as for appending a `char` to a `string`. `+` is not supported between `char`s.
  - `|` and `&` are available only for sets.`|` and `&` are used for set union and intersection respectively and if these two operators appear before a set identifier (like `&A`) then it is considered as distributed-union or distributed-intersection of that power set. In case of the later, they are treated as unary operators.
  - `=>` means implies. Both sides of the operator must be booleans (boolean variables or functions that evaluate to `true`/`false`).
  > No operators are supported for `uuid` and `url`.
* When specifying a predicate with multiple members and operators, parentheses can be used to group them and also to provide precedence information. For example - `(A & B) & C` between 3 sets `A`, `B` and `C` is performed between `A` and `B` first and the result is then used with `C`.

When applying to custom set specifications, operators are treated as functions where the function name is one of the operator symbols described above. The domain can either be a single set or a 2-tuple. The range should always be a single set. They follow the same syntax (all 3 options) as functions. But the keyword used is `operator` instead of `function`. The host set to which this operator must apply is indicated by restricting the first parameter of the operator to the host set. For binary operators, the second parameter must satisfy the constraints applied to that operator in terms of the types that operator supports.

## Iterations / Loops

This language doesn't support looping constructs like `for`, `forall`, `foreach`, `while`, etc. Instead you can use the existing `x in S` notation. Example - `x in S: { <predicates to apply to each element 'x'> }`.
> The exact meaning of `x in S` depends on the context where it is used. In set specification, `x in S` is followed by a `;` and the predicate. For iterations, `x in S` is followed by `:` and then the predicate(s) that are part of the iteration.

## Composition

Relations and functions can be composed to form complex ones. For example `r1(r2(a))` represents the relation composition `r1 • r2` where `r1` is `A -> B` and `r2` is `B -> C`.
> As part of presenting design analysis, the report can use `(A -> B) -> C` as a shortcut to imply composition.

## Built-in functions

Built-in functions are functions that are available for built-in types and for any types defined in code. For types defined in code, these must be overloaded (similar to how it is done in C++ and similar languages).
- `domain(A)` on a relation `A` returns its domain set.
- `range(A)` on a relation `A` returns its range set.
- `inverse(A)` on a relation `A` returns another relation that is the inverse `A` i.e., if `A` is `B -> C` then `inverse(A)` would be `C -> B`. Here the mapping information from `A` is used to infer the map for the inverse.
- `count(S)` returns the cardinality of the set `S` i.e., the number of elements in the set.
- `type(v)` returns the type information i.e., the source set, of the supplied variable. When used with a tuple, it returns the ordered list of types used in that tuple.
- `bijective(A)` on a 2-tuple returns `true` if it is a bijective relation.
- `injective(A)` on a 2-tuple returns `true` if it is a injective relation.
- `surjective(A)` on a 2-tuple returns `true` if it is a surjective relation.
- `identity(A)` on a 2-tuple returns true if it is an identity relation.
- `homogeneous(A)` on a 2-tuple returns true if both domain and range in the relation `A` are from the same source set.
- `reflexive(A)` on a 2-tuple returns true if it is a reflexive relation.
- `symmetric(A)` on a 2-tuple returns true if it is a symmetric relation.
- `antisymmetric(A)` on a 2-tuple returns true if it is an antisymmetric relation.
- `transitive(A)` on a 2-tuple returns true if it is an transitive relation.
- `preorder(A)` on a 2-tuple returns true if it is both reflexive and transitive.
- `order(A)` on a 2-tuple returns true if it is reflexive, transitive and antisymmetric.
- `equivalent(A)` on a 2-tuple returns true if it is reflexive, transitive and symmetric.
- `classes(A)` returns a list of disjoint sets that are equivalence classes of a 2-tuple `A`.
- `partitions(A)` returns a power-set of all possible partitions of `A`. Each partition contains sets that are equivalence classes of `A`.
- `partition(B, A)` returns true if all members of `B` are equivalence classes of `A`.
- `paths(A)` returns a set of paths between any two elements of `A`.
- `path(B, A)` returns true if `B` contains elements that form a path using elements from the 2-tuple `A`.
- `cycles(A)` - returns a power set of all possible cycles using elements from 2-tuple `A`.
- `cycle(B, A)` returns true if elements of `B` form a cycle and are all sourced from the source set `A`.
- `cycle(B)` returns true if elements of `B` form a cycle.
- `closed(A, <function/operation>)` returns true if elements of `A` satisfy closure property under the provided function/operation.
- `associative(A, <function/operation>)` returns true if elements of `A` satisfy associative property under the provided function/operation.
- `commutative(A, <function/operation>)` returns true if elements of `A` satisfy commutative property under the provided function/operation.
- `distributive(A, <function/operation>, <function/operation>)` returns true if elements of `A` satisfy distributive property. 
  > Using distributive property's formal definition, `+` is represented by the first function/operation argument and `*` is represented by the second function/operation argument.

## Additional syntax for relations/functions/opeartors

A relation/function/operator can be scoped to a specific set / type by using the `on <type>` operator. Example -
```
S := {x: x in nat; P(x)}
function Func(i int, b bool) -> bool on S: {
  # List of predicates to apply.
}
```
In such cases, the host set's members can be part of domain and/or range (depending on the function). Additionally, all properties and constraints associated with the host set are automatically assigned to all parameters and local sets of this function. 

When specifying a relation, function or operation, if you want it to satisfy specific properties, you can add them as attributes of that relation/function/operation usind the `@`. Multiple properties can be added using `@[]`. Parser/Analyzer will flag an analysis error when that relation/function/operation doesn't satisfy all those properties. Examples -
```
@closure
function f(x int, y int) -> bool {...}
```
```
@[closure, commutative]
function f(x nat, y nat) -> (z int) {...}
```

## Relational Algebra

Relational algebra opearations, the most common operations used in database analysis, are supported.

* `projection(A, [])` returns the projection of a tuple `A` by picking attributes referred to by the indices in the supplied tuple.
* `select(A, <predicate>)` picks only those members of tuple `A` that satisfy `<predicate>`. 
* `join(A, B, <predicate>)` joins two tuples into a larger tuple based on `<predicate>`. 
* `semijoin(A, B, <predicate>)` picks only those members of A that satisfy `<predicate>` where the predicate itself is defined over members of `A` and `B`.

## Abstract algebra

Specific algebraic structures can be implied using the associated keyword - `magma`, `quasi` (for *quasi group*), `semigroup`, `monoid`, `group`, `abelian`, `ring`, `comring` (for *commutative ring*), `integral`, `field` and `loop`. For example `group A := {x: x in int: P(x)}` will define a group and will apply all properties a group must satify to the set `A`. 
Any additional functions defined for these structures, must satisfy all property constraints of the structure. This rule doesn't apply to morphisms that may map the structure to another set (that may/may-not be of the same type).

## Custom datatypes

New set types and structures can be built using existing ones. Custom types can either be a subset of a basic type/structure or a tuple made out of subsets.
