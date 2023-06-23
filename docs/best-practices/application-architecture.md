Our applications are divided into the following layers:

* REST
* Business
* Store

## REST layer

* Contains:
    * JAX-RS resource classes
    * DTO classes
    * corresponding converters
* Converters and DTO classes are only ever used inside the REST layer
    * These classes should use default visibility ("package level") whenever possible. For example, in a `dto` module
      used by server and client code, they will necessarily have to be public.
* Calls the business layer to do the actual work.

## Business layer

* Contains:
    * POJOs for the business objects (in `*-business` packages)
        * Unlike DTOs and record classes, these POJOs are passed around between layers.
    * Process classes (in `*-core` packages)
        * These classes implement business processes by orchestrating other classes inside this layer and the store
          layer.
        * Common process classes are `Creator`, `Reader`, `Updater`, `Deleter` and `Lister`.
        * Names are not dependent on other layers. For example, there usually is only one way to create a POJO, and the
          corresponding class is the `Creator`. Even if that class uses an `UpsertStore`, it is never called `Upserter`.

!!! note

    POJOs and process classes live in different packages (`business` and`core` respectively) to avoid package cycles.

## Store layer

### Record classes

* Database-oriented representation of business objects.
* Only ever used inside the store layer, so they should use default visibility ("package level").

### Store classes

Each **_operation_** is implemented in its own class, be it the standard operations listed below or custom ones like "confirm message".

| Operation         | Description                                                                                                                                                                                                                                                                                                   |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Create`        | write a new row; throw `ConflictingEntityException` if it already exists                                                                                                                                                                                                                                      |
| `Upsert`        | write a new row, overwriting any existing one                                                                                                                                                                                                                                                                 |
| `Update`        | overwrite an existing row; throw `ObjectNotFoundException` if it does not exist                                                                                                                                                                                                                               |
| `Delete`        | delete an existing row by primary key; throw `ObjectNotFoundException` if it does not exist                                                                                                                                                                                                                   |
| `DeleteBy\*`    | delete rows based on certain criteria that are not the primary key or only a subset of the primary key                                                                                                                                                                                                        |
| `Read`          | attempt to read a record and return an `Optional`                                                                                                                                                                                                                                                             |
| `FindBy\*`      | attempt to read one or multiple records based on certain criteria that are not the primary key or only a subset of the primary key, returning an Optional or a List                                                                                                                                           |
| `List`          | return a `List` of all records                                                                                                                                                                                                                                                                                |
| `AtomicGroup\*` | treat a group (`Set` or `List`) of rows and some object that identifies this group as one atomic element for the store operation.<br>For example, a `FooAtomicGroupUpsertStore` deletes existing `Foo` rows (if any) matching the given group key, then writes rows for the given group (which may be empty). |

* Interfaces naming scheme is`<Pojo><Operation>[Async]Store` (non-asynchronous stores being the rule)
* Classes use the interface name and add their platform as a prefix, e.g. `CassandraFooDeleteStore`.
* To the business layer, store operations are atomic.
    * For example, when implementing "Create" is only possible with a read before a write, those calls are
      encapsulated inside one store class.
* Internally, a store may call other stores as needed.
