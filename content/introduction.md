# Introduction

We often call Kraken a build system, but it would actually be more accurate to describe it as a "task orchestration
system". Kraken is not designed to replace optimized toolchain-specific build systems like CMake, Cargo, Poetry, Go,
etc. Rather, it is intended to operate one level above these tools to invoke these tools with the right configuration
and in the correct order.

Kraken is composed of a core API, the `kraken-core` package, which implements abstract representation of tasks, and
how to execute them. Tasks live in projects and may have dependencies on other tasks. It also provides a `kraken`
CLI that contains all of the functionality that directly interacts with the build graph, such as executing tasks or
querying information about the build.

Beyond the core API, there is the `kraken-std` package which contains the standard library for writing Kraken build
scripts to build projects for various types of projects (e.g. Rust, Python).

To execute Kraken, yuu should use the `krakenw` CLI provided by the `kraken-wrapper` package. It installs the
components of the Kraken build system per-project and allows you to generate a lock file of the build-time
dependencies, ensuring that builds stay fully reproducible.

## Installation

We recommend to install `kraken-wrapper` using [Pipx](https://pypa.github.io/pipx/):

    $ pipx install kraken-wrapper

!!! note

    We do not recommend to install `kraken-core` directly. While it would give you direct access to the `kraken`
    CLI (note the difference, no trailing `w`), it means that every project that you use the `kraken` CLI in will
    use the same version of `kraken-core` and `kraken-std` or any additional build-time dependencies that you may
    have installed.

    Worse, someone else that wants to build your project may have a different version of any of these dependencies
    installed. To ensure fully reproducible builds, use the `kraken-wrapper` CLI (`krakenw` command) and make it
    generate lock files (more on that below).

## Build scripts

Kraken build scripts can be written in pure Python or [BuildDSL](https://niklasrosenstein.github.io/python-builddsl/).
In order for `kraken-wrapper` to know the dependencies for your Kraken build environment, you need to call the
`buildscript()` function the top of your build script. It is only _after_ the `buildscript()` call that you should import any build script dependencies.

<table align="center"><tr><th>Python</th><th>BuildDSL</th></tr>
<tr><td>

```py
from kraken.common import buildscript

buildscript(
    requirements=["kraken-std>=0.4.16"],
)
```

</td><td>

```py
buildscript {
    requires "kraken-std>=0.4.16"
}
```

</td></tr></table>

An example build script for a Python project may look like this:

<table align="center"><tr><th>Python</th><th>BuildDSL</th></tr>
<tr><td>

```py
from kraken.common import buildscript

buildscript(
    requirements=["kraken-std"],
)

from kraken.std.v2.python import *

python(
    mypy={},
    pytest={
        "directory": "tests",
    }
)
```

</td><td>

```py
buildscript {
    requires "kraken-std"
}

from kraken.std.v2.python import *

python {
    mypy { ... }
    pytest {
        directory = "tests"
    }
}
```

</td></tr></table>


