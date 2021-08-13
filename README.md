# PHP RFC: Never For Argument Types

## Introduction

Arguments in PHP are [contravariant](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)) to preserve Liskov substitution. This means that if class B extends class A, then redefines a function call, the entire type of that argument from class A must be present in the type of the argument in class B:

```php
<?php

abstract class A {

  abstract public function foo(int $arg);

}

class B extends A {
  
  public function foo(int|float $arg) {
    return round($arg) + 1;
  }
  
}
```

Thus, the more specific a type is for an argument in a base class, the more broad it can be in an extending class with the requirement that it must also include the type from the base class.

Since `never` is a [bottom type](https://en.wikipedia.org/wiki/Bottom_type) within the PHP engine, all other types contain it. This RFC proposes allowing `never` as a valid argument type for methods on interfaces and abstract functions.

## Use Cases

### Interfaces and Abstracts

With the `never` type available, interfaces and abstracts would be able to require that implementing classes provide a type without specifying any details about what that type must be. This would allow use cases such as the following:

```php
<?php

interface CollectionInterface {

  public function add(never $input): self;

}
```

Implementers of the `CollectionInterface` could then specify any type they want, but failing to specify a type would result in a Fatal Error. Functions which use collections could then type against the interface, allowing any of the variously typed implementations to be provided.

In this way, allowing the `never` type for interfaces and abstracts could be a method of providing minimal support for generics while avoiding the challenges that providing generics represents. It would also not prevent or make it more difficult to provide full generics in the future.

### Internal Classes and Interfaces

Providing internal interfaces that require a type but do not specify which type can be very beneficial. In fact, the motivation for exploring this concept originated in researching operator overloads and the interfaces that such a feature would provide.

## Design Considerations

### Using Never vs. A New Type

While it could be argued that the meaning of `never` was that code using this type would terminate, a new type offers several issues over using `never`:

- `never` already represents the concept of a bottom type in PHP due to its usage with return types. The engine already has the concept of this type built in, but is not currently exposing it for use with arguments. Having multiple bottom types not only doesn't make sense, but I cannot find a single instance of any programming language having multiple bottom types.
- `never` correctly indicates that the code which uses it for an argument type can never be called directly.
- This usage has precedence in other languages; see below.

### Other Languages

There are several languages which contain a bottom type, some of which use `never` as their bottom type. The behavior described in this RFC is in fact how `never` behaves and can be used [in TypeScript](https://blog.logrocket.com/when-to-use-never-and-unknown-in-typescript-5e4d6c5799ad/), which also uses `never` as its bottom type.

Scala also uses the bottom type to denote covariant parameter polymorphism, though the bottom type in Scala is `Nothing`.

## Proposal

Allow the use of `never` as a type for arguments in interfaces and classes. This would have the following semantics:

- `never` cannot be used in an intersection type or union type. Any intersection would reduce to `never`, and any union would reduce `never` out of the union, as `never` is the identity type of unions.
- Attempting to call code directly that uses the `never` type as an argument would result in a `TypeError`, as no zval will match this type.

## Backward Incompatible Changes

None

## Proposed PHP Version

This change is proposed for PHP 8.2
