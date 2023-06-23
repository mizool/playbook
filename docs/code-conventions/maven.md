# Maven conventions

## General

* Do not set the project `<name>`. It is redundant and results in additional maintenance when changing artifact IDs.
* If you really, definitively need to add information beyond what is contained in `<groupId>` and `<artifactId>`, use `<description>`.
* In multi-module projects, always specify the `<relativePath>` of the `<parent>`.

## Project structure

Or projects usually follow a common structure:

* Maven modules belong to either of the following:
    * an application/service the project provides ("bar")
    * a business domain module ("foo") which is used by one or more applications
* Applications and modules are further subdivided into business, core and other aspects.

For example, a project that deals with an ESB and provides AWS lambdas and REST APIs might use the following directory structure:

<!-- TODO replace with plugin mentioned at https://github.com/squidfunk/mkdocs-material/discussions/3539 --> 
![](https://kroki.io/blockdiag/svg/eNqFUstugzAQvPMViF7dSknTqlJEfySqkG0MWHW8yDbKocq_1w-MgZBkD2DvzA7LzhIB9LfmuM3_stxGzRo8CFPpDvcsL_NGwIV2WJk3w9SZS2xAeaKEmlUXXpvOsnbv-5TsGG87Y7P7w0KyVTD0FQUByoLFS-Oj8BzdYzmpHb5SbhLbfWY-S7BmNVcngQkTZVGg3LdaUq6oYCj3IuXu48ezz1APgulT4KSf4bIfTKA0AE_gigyaS6b1-NF4TQQKio2gOyZAm4T4c4Bw3wtOseEgH_ZGsHoCP-7NETZ7816Mlkcil9biBlMWtVIiDnmziy09F0yT05Oye6UuuKRgF669AWAwLayBa3Z72lIW-ExqvEgpps2qOjwDl4I02M40rP2FEWveYhXz1--4Z2jh7HG-go5lN2J8TbahaX9QWpjjWn6u6u7WrshR432mGF1HK1tD19YWVxLHi6Z5ToqzEse0BYuPrcaCxpncKw905Md8zK7_WHVhpw==)
<!--
blockdiag {
    default_shape = flowchart.terminator
    node_width = 132
    node_height = 24
    default_group_color = "#ffffff"
    span_width = 48
    span_height = 16

    basedir[label="", shape=circle, width=15]
    modules[shape=flowchart.input]
    foo[shape=flowchart.input]
    foo_business[label=business]
    foo_core[label=core]
    foo_store[label=store]
    applications[shape=flowchart.input]
    bar[shape=flowchart.input]
    bar_business[label=business]
    bar_core[label=core]
    group {
        bar_interfaces[label=interfaces, shape=flowchart.input]
        group {
            esb[shape=flowchart.input]
            group {
                incoming
                outgoing
            }
        }
        group {
            lambda
            rest
        }
    }
    lambdacontainer
    webapp

    basedir -> modules, applications;
    modules -> foo -> foo_business, foo_core, foo_store;
    basedir -> applications -> bar;
    bar -> bar_business, bar_core, bar_interfaces

    esb -> incoming, outgoing;
    bar_interfaces -> esb;
    bar -> lambdacontainer, webapp;
    bar_interfaces -> lambda, rest;
}
-->

The directories in parallelogram boxes (▱) are aggregator modules, the other ones result in either a jar or war.

## IDs of artifacts and groups

The group ID of the POM in the project root directory represents the entire project (e.g. `com.example.fortytwo`). Its artifact ID is identical to the last segment of the group ID.

All other group IDs and artifact IDs are derived from the root group ID and the relative paths as follows:

* For each nested directory, the POM's group ID consists of the group ID of the parent plus the directory name. The artifact ID is identical to the last segment of the group ID.
* This process continues until a sensible cut-off point is reached. Usually, this would be the aggregator pom of a business domain module, service or application.
* Below the cut-off point, the group ID remains unchanged. For each nested directory, the POM's artifact ID consists of the parent's artifact ID plus the directory name.

For the directory structure shown above, these rules result in the following artifact coordinates:

| Group ID                                | Artifact ID                   |
|-----------------------------------------|-------------------------------|
| `com.example.fortytwo`                  | `fortytwo`                    |
| `com.example.fortytwo.modules`          | `modules`                     |
| `com.example.fortytwo.modules.foo`      | `foo`                         |
| `com.example.fortytwo.modules.foo`      | `foo-business`                |
| `com.example.fortytwo.modules.foo`      | `foo-core`                    |
| `com.example.fortytwo.modules.foo`      | `foo-store`                   |
| `com.example.fortytwo.applications`     | `applications`                |
| `com.example.fortytwo.applications.bar` | `bar`                         |
| `com.example.fortytwo.applications.bar` | `bar-business`                |
| `com.example.fortytwo.applications.bar` | `bar-core`                    |
| `com.example.fortytwo.applications.bar` | `bar-interfaces`              |
| `com.example.fortytwo.applications.bar` | `bar-interfaces-esb`          |
| `com.example.fortytwo.applications.bar` | `bar-interfaces-esb-incoming` |
| `com.example.fortytwo.applications.bar` | `bar-interfaces-esb-outgoing` |
| `com.example.fortytwo.applications.bar` | `bar-interfaces-lambda`       |
| `com.example.fortytwo.applications.bar` | `bar-interfaces-rest`         |
| `com.example.fortytwo.applications.bar` | `bar-lambdacontainer`         |
| `com.example.fortytwo.applications.bar` | `bar-webapp`                  |

## Versions

* Version numbers follow the [Semantic Versioning](http://semver.org/) specification, which defines three version segments: `MAJOR.MINOR.PATCH`.
    * All versions always include `MAJOR.MINOR`, for example `0.9`, `1.0` or `2.48`.
    * When `PATCH` is `0`, leave it out: For example, we write `2.7` (instead of `2.7.0`) and a later hotfix release would be called `2.7.1`.
* The version number of the project is the same for all its modules and is specified in the top-level module.
* Submodules specify their parent using group ID, artifact ID and version, without any `${...}` expressions.
* Submodules inherit the version and group ID of their parent; the latter may be overridden.
    * For example, the "applications" project above sets its group ID to `com.example.fortytwo.applications` - if it didn't, it would inherit the value `com.example.fortytwo` from its parent.
* Release version numbers (non-SNAPSHOTs) are never set manually. This is only done during the release process.
* To define the upcoming release version, set the related snapshot version manually.
    * For example, when **fortytwo** `2.7` was released, the project version was changed automatically to `2.8-SNAPSHOT`. Now the developers decide that the upcoming version should not be called `2.8`, but `3.0`. A developer therefore changes the project version manually from `2.8-SNAPSHOT` to `3.0-SNAPSHOT`.
    * The version number needs to be changed in all modules. To make this process easier, use `mvn versions:set -DgenerateBackupPoms=false` and enter the new version.

## Managing dependencies

* Keep things tidy. Do not declare anything that is not used anymore, or even worse, not yet used.
* Order entries alphabetically by group ID and artifact ID. In the rare case where order matters for compilation, add a comment explaining the situation. See [Sorting POMs](#sorting-poms) below.

### Dependencies
* Always declare dependencies explicitly, do not rely on transitive dependencies. `mvn dependency:analyze` helps with that.
* Never declare version numbers or scopes in dependencies, but only in the project's `<dependencyManagement>` block (see below).
* Declare the `<optional>` flag only on a dependency, never in `<dependencyManagement>`.
* Do not declare dependencies in POM modules, only in JAR/WAR modules. Even if multiple child modules have the same dependency, do not move up the dependency

    !!! info "Rationale"
        This rule avoids declared-unused-dependency warnings from `mvn dependency:analyze`.

### Dependency Management
* Use exactly one `<dependencyManagement>` block inside the top-level module of the project (the one that defines the project's version number and inherits from a corporate-level POM), not the child modules. In the example above, this would be the `com.example.fortytwo:fortytwo` POM.

    !!! info "Rationale"
        This way, when preparing a release, only one module needs to be checked for SNAPSHOT dependencies.

* In `<dependencyManagement>` entries for projects which are part of the aggregator (e.g. dependencies between **fortytwo** projects, see above), specify the version using the variable `${project.version}`.

    !!! warning
        Do not use `${project.groupId}` as the group ID may be different in the POM that has the real dependency.

* Do not add `<dependencyManagement>` entries for artifacts which inherit from the current one. For example, if the **fortytwo** project inherits from the corporate-level POM, that corporate POM should not have a `<dependencyManagement>` entry specifying a certain **fortytwo** version.

    !!! info "Rationale"
        Doing so would cause a chicken-and-egg problem when releasing each of the POMs the first time, because a release can only depend on other releases, not SNAPSHOTs, but there are no releases yet. Although introducing the entry later would work, it would confuse the hell out of everybody, so the rule is to avoid this altogether.


## Configuring plugins

* Specify the plugin `<version>` inside `<build><pluginManagement>`, never in `<build><plugins>`.
* Plugin `<executions>`, `<configuration>` and `<dependencies>` elements are also specified inside `<build><pluginManagement>`.

    !!! info "Rationale"
        Apart from consistency, this makes overriding configuration in inheriting projects easier.

* The rules above should result in `<build><plugins><plugin>` entries that only contain `<groupId>` and `<artifactId>`.

## Sorting POMs

Recent versions of the corporate POM configure the maven-sortpom-plugin. Just use `mvn sortpom:sort` on your project to have everything in order.

!!! warning

    Some test dependencies need to be in a certain order to override other dependencies. In this case, add an [Ignore Section](https://github.com/Ekryd/sortpom/wiki/IgnoringSections) to ensure everybody can always invoke `maven-sortpom-plugin` safely. Inside the section, always add a comment explaining the ordering details. See also rule OMG-10.

## Bill of Materials (BOM)

* BOMs only consist of a `<dependencyManagement>` section, they do not declare dependencies.
* Multi-module projects should offer a BOM listing their jar children.

    !!! info "Rationale"
        This can be [imported](http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Importing_Dependencies) into the `<dependencyManagement>` of other projects to ensure all modules used in that project are at the same version.

* BOMs can also be used to group external dependencies for one topic, e.g. `testing-bom` or `tools-bom`.

    !!! info "Rationale"
        This avoids littering our POM landscape with endless combinations of the version numbers of the individual artifacts.

* BOMs should only be used outside the project that they represent, not from within.

    !!! info "Rationale"
        Doing the latter can result in chicken-and-egg problems and is thus deemed too complicated.


