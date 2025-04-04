# SQL conventions

## Preface

### Database guidelines at eurodata

Be sure to read the database guidelines in the eurodata engineering handbook.
Whenever a Mizool Playbook rule reiterates, clarifies, extends, tweaks or changes eurodata guidelines, it references the respective rule like this: `ed-DB-321`.

!!! info "For non-eurodata developers"

    Unfortunately, the eurodata engineering handbook is not available to third parties.
    Therefore, when starting to work on a PR for an open source project that uses this Mizool Playbook, please ask the eurodata developers if there are any rules that you should know about.

### Applicability to legacy code

Legacy code often violates many of the rules given below.
Unless working on a modernization effort with an explicit strategy for dealing with (or intentionally breaking) backwards compatibility, the rules only apply to newly created elements.

In situations where old and new code meet (e.g. adding a new column to an old table), applying current conventions could lead to inconsistencies with neighboring elements.
If you encounter such a case, discuss it with other developers, then choose an approach to use and make a corresponding note on the surrounding element.

## Naming conventions

### General naming
 
- Use lower case for all identifiers in databases, and use `snake_case` for multi-word identifiers (see [Capitalization and format](naming.md#capitalization-and-format) and ed-DB-115).
- Avoid abbreviations and specifically making up your own abbreviations.
  For details, see [General rules](naming#general-rules) and ed-DB-118.
- Table names use singular (`book`, not `books`).
  See ed-DB-117.
- Do not add prefixes and suffixes to the name of a column based on its type or usage.
    - Specifically, do not add `fk_` to the name of a column that is (part of) a foreign key constraint (a practice suggested by ed-DB-119).
- Do not add a general prefix or suffix to the name of views to distinguish them from tables (a practice suggested by ed-DB-119).
    - A side effect of this rule is that you cannot have a table `foo_status` accompanied by a `v_foo_status` view, which we think would be confusing.
      If a view has some specific purpose, include it in the name instead, e.g. `foo_status_printable`.
      This way, it (mostly) becomes an implementation detail whether it's a view or a table.

### Names of keys, indexes and constraints

The following rules are conscious departures from ed-DB-119.

Names of keys, indexes and constraints follow the structure given below, with segments separated by double underscores (`__`). 

| Element                           | Name segments                                                    | Examples                                                                      |
|-----------------------------------|------------------------------------------------------------------|-------------------------------------------------------------------------------|
| Primary key                       | table name<br>`pk`                                               | `my_cool_table__pk`                                                           |
| Foreign key constraint            | owning table<br>`fk`<br>each included column of the owning table | `book__fk__author_id`<br>`complaint__fk__order_id__order_line_pos`            |
| Unique index<br>(full or partial) | table name<br>`u`<br>title / each included column                | `document__u__id__account_id`<br>`subscription__u__one_active_per_account_id` |
| Other index<br>(full or partial)  | table name<br>`i`<br>title / each included column                | `group__i__parent_id`<br>`orders__i__unbilled_order_numbers`                  |
| Check constraint                  | table name<br>`c`<br>title                                       | `account__c__name_length`<br>`contact__c__email_or_phone_set`                 |

!!! info "Name length"

    PostgreSQL's maximum name length is 63 characters, which should be sufficient for most elements, even foreign key constraints with multiple columns.

    In case a name constructed according to the rules above ends up being too long, abbreviate or truncate some segments.
    However, double check that it is still obvious which column each name segment refers to, and that there is no ambiguity.

## Tables and columns

- Define primary keys, foreign key constraints, check constraints and indexes (unique or otherwise) explicitly, not as part of a column definition.
- Always specify the name of keys, constraints and indexes.
  Letting the RDBMS generate names would prevent developers from documenting their intention and often makes code hard to comprehend. 
- Foreign key constraints
    - Each relation between tables must be covered by a foreign key constraint (see ed-DB-107).
    - Only specify the clauses that deviate from the default, e.g. include `on delete cascade` but don't add `on update no action`.
    - If the referenced column is the primary key, specify only the table name without a column name:
      ```sql
      constraint book__fk__author_id
          foreign key (author_id)
              references author   -- leave out `(id)` here
              on delete cascade
      ```
- Primary keys should always be a [surrogate key](https://en.wikipedia.org/wiki/Surrogate_key), also known as a synthetic key (see ed-DB-108)
    - Using [natural/business keys](https://en.wikipedia.org/wiki/Natural_key) (like email addresses, or an employee ID generated by another system) may feel more intuitive at first, but it often causes problems later on.
    - Instead, include the natural/business key as a data column, possibly with an index.
      However, be careful with unique indexes on such columns as you may not be able to trust the other system to enforce uniqueness at all times. 
    - This rule applies even if (or _especially if_) you cannot think of a scenario where the "obvious" natural/business key value of a row would change.
- Columns that hold ID/key values generated by another system should always unambiguously identify that system.
    - You may feel that it's obvious to everybody that `external_id` or `source_id` refers to the company's "SplineReticulator Directory" product.
      But even if that were true _at the time of writing_, that does not mean that the same is true for _future you_ or somebody else in charge of deciphering the code in 5 years.
    - Also, following this rule makes it easier to connect additional systems (each with their own column), or migrate from one to the other.
- As columns are nullable by default, omit the `null` keyword in column definitions.

## Syntax and data types

- Do not quote identifiers unless it is required, for example because a column name is a reserved word (e.g. `timestamp`). 
- Type casts
    - Prefer ANSI SQL syntax for casts over RDBMs specific syntax:
      ```sql
      cast(my_value as text)  -- good!
      
      my_value::text          -- bad, avoid PostgreSQL specific syntax where possible.
      ```
    - Avoid scattering type casts all over the place, it makes the code harder to read (and may even lead to back-and-forth casting which is confusing).
      Instead, try to centralize them using either of the following strategies:
        - near the original columns (e.g. the lowest levels of a multi-level Join) when working with raw data not usable otherwise (e.g. when dates or numbers are imported and stored as strings)
        - at the latest possible step (e.g. when required by a join)
