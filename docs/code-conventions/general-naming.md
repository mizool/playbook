# General naming conventions

## Capitalization and format

!!! note

    Names never include spaces unless specified otherwise.

| Area               | Element                                  | Capitalization style | Notes                                                                                                                      |
|--------------------|------------------------------------------|----------------------|----------------------------------------------------------------------------------------------------------------------------|
| Java               | Package                                  | `lowercase`          | no word delimiter, e.g. `com.example.myproject.somepackagename`                                                            |
| Java               | Class<br>Interface<br>Enum<br>Annotation | `UpperCamelCase`     | See [Class names](#class-names) below.                                                                                     |
| Java               | Method<br>Variable                       | `lowerCamelCase`     | See [Method names](#method-names) below.                                                                                   |
| Java               | Constant                                 | `UPPER_CASE`         | `static final`                                                                                                             |
| Java               | Enum value                               | `UPPER_CASE`         |                                                                                                                            |
| JavaScript         | Identifiers                              |                      | Try to mimic Java conventions as much as possible, e.g. treat modules injected via Require like Java classes.              |
| JavaScript         | Files                                    | `lowerCamelCase`     |                                                                                                                            |
| Maven              | group ID<br>artifact ID                  | `lowercase`          | Use '-' (minus sign) as a delimiter between certain elements, see Maven conventions.                                       |
| Database           | Tables<br>Columns<br>Indexes<br>…        | `lower_case`         | Use for all identifiers in the database, regardless of whether it's Cassandra, SQL, or something else.                     |
| General (Fallback) | File<br>Folder                           | `lowercase`          | If a word delimiter is required, prefer '-' (minus sign). If that character is not allowed, fall back to '_' (underscore). |

## General rules

* Avoid words like `Manager` that could mean mostly anything.
* When naming multiple related items, pay attention to established sets of terms like the ones given below. Do not
  replace terms with unusual ones, and do not mix terms of different groups (e.g. "add" and "delete").
    * source & target
    * origin & destination
    * add & remove
    * create & destroy/delete
    * start & end (exclusive upper bound)
    * first & last (inclusive upper bound)
* Abbreviations
    * Treat abbreviations as words, e.g. `ConfigurableFtpClient`. Not only is this easier to read
      than `ConfigurableFTPClient`, but it allows IDEs to find the class even if you only enter `cfc`.
    * Never make up abbreviations. Use only abbreviations that are widely known across the industry, such as `Io`
      or `Http`.
    * Do not shorten words, e.g. `Msg` for `Message`.
    * Only use the abbreviation if it is more common than the full term. For example, write `Database` instead of the
      abbreviation `Db`, but use the abbreviation `Html` instead of `HyperTextMarkupLanguage`.

## Class names

### Interfaces

Names of interfaces do not use a prefix or suffix, such as the `IWhatever` naming scheme used on the .NET platform.

If an interface exists to allow multiple pluggable implementations, the implementation class names use (parts of) the
interface as a suffix. For example, the `WirelessSplineReticulator` is an implementation of the `SplineReticulator`
interface. There might even be a `DefaultSplineReticulator` which is used in the standard configuration and/or as a
fallback. This name might also be used if - for now - there is only one implementation.

However, if the interface is not about pluggability, but exists for technical reasons only (e.g. to prevent cycles in
dependency injection, or to make unit testing easier), its only implementation uses the `Impl` suffix,
e.g. `SplineReticulatorImpl`.

### Unit tests

The test class for `SplineReticulator` is named `TestSplineReticulator`.

!!! note

    We do not use `SplineReticulatorTest` because that would appear in an IDE class lookup when entering `splinereticulator`.

### Utility classes

Classes that offer a collection of helpful methods do not use suffixes like `Util`, but the plural form of the "target".
For example, a utility class related to `Spline` objects is simply called `Splines`, analogous to the
JDK's `Collections` or Guava's `Objects` classes.

## Method names

`verify*`
:   Verifies that something is as expected, throws an exception if it isn't.

    Example: `verifyBasePackageSet`

`ensure*`
:   Ensures that something is as desired. Unlike `verify*()` methods, it silently fixes some or all violations, throwing
an exception for those it doesn't fix.

    Example: creation timestamps are write-once fields, so an UpdateStore only accepts the original value or `null`. In the latter case, it keeps the original value. The method implementing this could be called `ensureUnchangedCreationTimestamp`.

`validate*`
:   Verifies that something is as expected. Violations are recorded in some way, but unlike with the `verify*()`
methods, no exception is thrown.

    !!! note

        This naming scheme is intended for builders that mimic the behavior of Bean Validation, where violations are collected instead of being thrown as individual exceptions.

    Example: `validatePrimaryLocalePresent`

`obtain*`
:   Returns a non-null reference or throws an exception. The result may be a default value.<br><br>Use this in cases
where the calling site never wants to continue without the information. In contrast, `*ReadStore` classes treat the "
present" and "absent" situation equally,
see [OMG-13 - Store behavior when retrieving non-existent objects](https://incub8.atlassian.net/browse/OMG-13).

    Example: `obtainTenant`

`test*`
:   Unit test methods. Test methods can, but do not have to, incorporate names of methods of the class being tested.

    Examples: tests for the `fiddle()` method could be called `testFiddle` or `testFiddleRejectsNull`. A test of the serialization behavior, which does not invoke a method on the class to be
tested, could be called `testSerialization`.

`setUp` & `tearDown`
:   Unit test setup code.

## Variables

As our methods and classes tend to be very short, the reader often encounters only one variable of each
type anyway. Therefore, fields and local variables are usually named based on the class name. For example, a reference
to a `BoundStatement` is simply named `boundStatement` or even `statement`.

If the approach above is not feasible because there are several variables of the same type, we add just enough
information to the names to avoid ambiguities. The original class name could be kept as part of the name (for example if
the method deals with many different types of variables) or omitted (e.g. if the variables involved obviously use the
same type, and it's a very common one).
