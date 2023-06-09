## General

* Do not set the project `<name>`. It is redundant and results in additional maintenance when changing artifact IDs.
* If you really, definitively need to add information beyond what is contained in `<groupId>` and `<artifactId>`,
  use `<description>`.
* In multi-module projects, always specify the `<relativePath>` of the `<parent>`.

## Project structure

Or projects usually follow a common structure:

*   Maven modules belong to either of the following:
    *   an application/service the project provides ("bar")
    *   a business domain module ("foo") which is used by one or more applications
*   Applications and modules are further subdivided into business, core and other aspects.

For example, a project that deals with an ESB and provides AWS lambdas and REST APIs might use the following structure:

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

The folders in parallelogram boxes (▱) are aggregator modules, the other ones either result in a jar or war. 