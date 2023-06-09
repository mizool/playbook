# Java conventions

## Naming

Be sure to read our [naming conventions](general-naming.md).

## Collections

* `null`
    * Inside the core layer, collections may only ever be empty, but never `null`. Other layers like REST and Store must
      turn `null` into empty collections before invoking/returning to the core layer.
    * Even outside the core layer, method return values should never be `null`.
* Immutability
    * Collections should be immutable whenever possible, using one of these approaches:
        * Guava's `ImmutableList` and other collections: if you can live with not storing `null` elements inside the
          collection, this is the preferred approach. In order of preference, instantiate them ...
            * if you have a stream: via Mizool's `GuavaCollectors.toImmutable*()` methods
            * via `of()`. This avoids the generics clutter and additional `build()` method call of the next option.
            * via a builder
                * if you need`addAll()`, or
                * if the elements to add are determined dynamically (e.g. in a loop)
        * Use `Collections.unmodifiable*()` methods to wrap a regular collection instance.
    * Note that this is **not** limited to "protect your instance state", but also covers collections for which no
      reference is kept in the code that creates it.
    * Exceptions to this rule are possible, but should be rare. Usually, calling code can simply filter the collection
      or create a new one.
* Arguments
    * When a method uses a collection argument only for iteration, it shall declare it as `Iterable<...>` to make its
      intention obvious.

        * Note that iterators allow removing elements, so this is only a hint. Safety is only provided by immutable
          collections.

## Handling `null` parameters

When a method or constructor parameter **should not accept `null`**, go through the following checklist:

1. If the code immediately accesses a member of the instance (e.g. `parameter.method()`, `parameter.field` or iterating
   the collection), the JVM will throw a `NullPointerException`. That's enough; do not add more code.
2. If the code stores the parameter for later usage, or if you feel the location where the `NullPointerException` would
   happen is "way over there" or even in another library, add a
   Lombok [`@NonNull` annotation](https://projectlombok.org/features/NonNull.html) to the parameter declaration.

Whatever you do, avoid explicit `if (parameter == null) throw new WhateverException()` code blocks. They are visual
clutter likely to be ignored by the reader, allowing bugs to hide.

When a method or constructor parameter **does accept null**, this is unusual, but okay. Some tips:

* Can you use overloading to make it more obvious the parameter is optional?
    * For example, `rotate(int degrees)` and `rotate(int degrees, int speed)` is better
      than `rotate(int degrees, Integer speed)` where _speed_ is optional.
    * Note that internally, those methods may of course delegate to a private method that _does_ use a nullable
      parameter.
* Write unit tests that ensure the `null` and non-`null` behaviors are kept.
* Consider writing a Javadoc comment mentioning that passing `null` for that parameter is allowed and explaining the
  effects of doing so.

## Constructors

### Regular constructors

Wherever possible, we use
Lombok's [`@RequiredArgsConstructor` annotation](https://projectlombok.org/features/Constructor.html) and friends to
generate constructors. An example where this is not possible is any class that needs to invoke a constructor of its
superclass.

### Constructors for `@Inject`

For managed CDI beans including producer classes, stateless session beans and other EJBs, we always use Lombok
annotations to generate `@Inject` constructors.

The access level for `@Inject` constructors should be set to package-level visibility (no modifier) to prevent
developers from accidentally invoking that constructor in production code. For injection and unit tests, that visibility
is no obstacle.

```java

@RequiredArgsConstructor(onConstructor_ = @Inject, access = AccessLevel.PACKAGE)
public class Foo
{
    private final Bar bar;
}
```

## Visibility

* Always choose the most restrictive visibility modifier you can get away with. Some examples:
    * Avoid making classes `public` unless they are required by other packages.
    * Even if a class has to be `public`, does its constructor really have to be public? See also the note above
      on [`@Inject` constructors](#constructors-for-inject).
* When a class has non-`public` visibility, do not bother setting all its `public` methods to the same visibility. This
  is unnecessary; a `public` method can be thought of as having "maximum visibility allowed by the class".
* Use `@VisibleForTesting` (from Guava) to flag cases where visibility was relaxed (or a method was added) to allow or
  improve tests.

## Final classes & immutability

Whenever possible, make classes `final`. This is especially useful for "protagonist" classes like `FooReader`
or `BarCreateStore` which are not intended for subclassing, but should instead use composition to share code. Note that
having unit tests does not preclude a class from being `final`
as [Mockito supports mocking such classes](https://github.com/mockito/mockito/wiki/What%27s-new-in-Mockito-2#mock-the-unmockable-opt-in-mocking-of-final-classesmethods).

For data classes, use the following Lombok annotations:

```java

@Value @Builder(toBuilder = true)
```

This results in final, immutable classes. With `MyClass.builder()` and `instance.toBuilder()`, they offer the same
convenience at call sites as mutable classes with all-args constructors and setters.

## Member ordering

Members are ordered within the following groups:

* Static members
    * Classes
    * Constants
    * Non-final static variables
    * Static initializer block (informally called "static constructor")
    * Static methods
* Non-static members
    * Classes
    * Member variables
    * Constructors
    * Methods

Methods are ordered in "reading order", i.e. a high-level method is placed above the low-level methods it calls. For
background and more details, refer to **Clean Code** by Robert C. Martin.

## Exception types

Mizool provides several standard exception types in
the [core.exception](https://javadoc.io/page/com.github.mizool/mizool-core/latest/com/github/mizool/core/exception/package-summary.html)
package. See the class comment for details on when to use each of them.

!!! info "Wrapping"

    For wrapping checked exceptions, these exceptions are almost always more appropriate than `RuntimeException`.

    As of Java 8, there is a specialized exception class for wrapping `IOException` called `java.io.UncheckedIOException`.

## Throwing exceptions

Do not write:

```java
if (something==okay)
{
    // Do work
}
else
{
    throw new SomethingWentWrongException("...");
}
```

Instead, use a so-called "watcher if":

```java
if (something!=okay)
{
    throw new SomethingWentWrongException("...");
}

// Do work. Look, Mum, no else block!
```

Also consider moving that watcher if to a `verify*()` method to increase readability:

```java
public doWork()
{
    verifyFluxCapacitorCharged();

    // Do work
}

private verifyFluxCapacitorCharged()
{
    if(!fluxCapacitor.isCharged())
    {
        throw new IllegalStateException("Flux capacitor must be charged");
    }
}
```

Using watcher ifs and `verify*()` methods has the advantage that adding more checks does not increase the indentation
level of the "good case" code.

## Leave out implied/unnecessary keywords

* `this`
    * Never use `this` for method calls.
    * For field access, only use `this` when there is a local variable of the same name.
* In interface method declarations, never use `public` or `abstract` as both are implied.
* Never use `final` on local variables (including method arguments) unless forced by the compiler.

## Utility classes

Sometimes, one has to write a utility class that only consists of static methods. These classes should be final and have
a private constructor.

To do this, simply use
Lombok's [@UtilityClass annotation](https://projectlombok.org/features/experimental/UtilityClass.html). Leave out the "
static" keyword.

## Unit testing

* Use TestNG, not JUnit.
* Use [AssertJ](https://joel-costigliola.github.io/assertj/) instead of the assertions provided by TestNG.
* Use Mockito.
* Use a `@BeforeMethod setUp()` to configure the object under test and any mocks it requires.

## Comments

* TODOs, either with or without issue keys, are discouraged as experience has shown that they tend to lose contact with
  reality and do not create much value.
