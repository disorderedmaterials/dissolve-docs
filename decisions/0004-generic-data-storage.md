# Generic Data Storage

- Status: **Accepted**
- Deciders: TGAY
- Date: 24-03-2021

## Context and Problem Statement

Dissolve's modular architecture requires that data is available for sharing / re-use between modules, meaning that a central data store is required over local storage within, for example, module classes themselves. This data can be of any type (POD, STL class, custom class) leading to a heterogeneous list.

## Decision Drivers

The original implementation of the `GenericList` class solves the problem of heterogeneous data storage via the use of an inheritance hierarchy (via `GenericItem` and `GenericItemBase` classes) which served several purposes:

1. Force the implementation of suitable serialisation and deserialisation routines within any class inheriting from `GenericItemBase`, allowing read/write operations to be controlled from the central container and streamlining read/write of simulation data _en masse_.
2. Force the implementation of suitable broadcast and equality functions such that data can be broadcast / checked between MPI processes when running in parallel.
3. Provide search / retrieval / creation of (named) data of any type through a series of templated functions.

As discussed by AW in [#453](https://github.com/disorderedmaterials/dissolve/issues/453) this approach has several drawbacks and limitations:

1. The enforcement of all needed functionality on any class inheriting `GenericItemBase`, even if that functionality is not required in operation.
2. For POD classes there is significant boilerplate.
3. The inheritance hierarchy is relatively dense

Drivers for change are therefore:

- Since the codebase is moving away from MPI, only the imposed serialisation/deserialisation functions are relevant.
- There is little scope for extension the functionality of the `GenericList` container in its current form without adding new functions to the base class, further increasing boilerplate.
- Reduction of code bloat, particularly for POD classes.
- Removal of the custom `List` and `ListItem` classes, upon which `GenericList` is currently based.

## Considered Options

Assuming that the underlying `List` is to be replaced by a `std::map<std::string, X>` where the `std::string` is the name of the data, the heterogeneous data item `X` can be represented as:

1. `std::variant`
2. `std::any`
3. `boost::fusion`

## Decision Outcome

Chosen option: "`std::any`", because it requires no additional dependencies (unlike `boost::fusion`) and has a smaller memory footprint per-item (unlike `std::variant`). By registering templated `std::function`s specific to individual class types, the necessary functionality can provided on a per-class basis, either calling class functions or implementing the necessary code in the function body. This also allows good separation between the `GenericList` and the various operations - serialisation, deserialisation, etc. Since the desired `std::map` is to contain values that are heterogenous (or are, for instance, a `std::tuple` containing a member which is the heterogeneous value) the benefits of `boost::fusion` do not seem readily applicable.

### Positive Consequences

- Facile implementation of new classes of functors acting upon specific class types.
- No inheritance required
- Significant code reduction

### Negative Consequences

- The use of registered function pointers rather than a series of templated specialisations (as would be the case with `std::variant`) is perhaps less transparent, however judicious use of `typedef`s helps to retain readability of the code.

## Pros and Cons of the Options

### `std::variant`

- Good, because requires no additional dependencies
- Good, because permits easy traversal of all data in the `std::map` via the use of a type matching visitor or overloaded operators
- Bad, because requires all functions for all types to be defined within the templated `GenericList` function, potentially leading to executable bloat and slowdown from extensive header inclusion from the base `GenericList` class
- Bad, because any item takes on the memory footprint of the largest class in the `std::variant` (e.g. `bool` vs `MyLargeClass`)

### `std::any`

- Good, because requires no additional dependencies
- Good, because the size of the `std::any` is defined by the individual type stored
- Bad, because there is no equivalent of `std::visit`, so (e.g.) templated function registrations must be employed to provide functionality
- Good, because the use of templated function registrations can be made outside of the templated `GenericItem` class functions, removing direct include dependencies in the build chain
- Bad, because need to indicate contained types at runtime via `typeid`.  This could lead to type errors that aren't caught be the compiler, though this should only occur if someone tries to do something "clever".

### `boost::fusion`

- Good, because more performant
- Good, because provides natural heterogenous storage
- Bad, because requires additional dependencies

## Links

- [Link type] [Link to ADR] <!-- example: Refined by [ADR-0005](0005-example.md) -->
- … <!-- numbers of links can vary -->
