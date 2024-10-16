# Contributing to the Azure Python SDK

If you would like to become an active contributor to this project, please follow the instructions provided in [Microsoft Azure Projects Contribution Guidelines](https://opensource.microsoft.com/collaborate/).

## Generated Code

If you want to contribute to a file that is generated (header contains `Code generated by Microsoft (R) AutoRest Code Generator`), the best approach is to open a PR on the initial Swagger specification, as we cannot merge a PR on generated code (it would be replaced by the next generation). See [Azure REST API Specifications](https://github.com/Azure/azure-rest-api-specs/) for details.

## Tools Overview

We utilize a variety of tools to ensure smooth development, testing, and code quality for the Azure Python SDK. Below is a list of key tools and their purposes in the workflow:

- **Tox:** [Tox](https://tox.wiki/en/latest/) is our primary tool for managing test environments. It allows us to distribute tests to virtual environments, install dependencies, and maintain consistency between local and CI builds. Tox is configured to handle various testing scenarios, including linting, type checks, and running unit tests.

- **Virtualenv:** [Virtualenv](https://virtualenv.pypa.io/en/latest/) is leveraged by Tox to create isolated environments for each test suite, ensuring consistent dependencies and reducing conflicts.

- **Pytest:** [Pytest](https://docs.pytest.org/en/stable/) is the test framework we use for writing and running our unit tests. It supports fixtures, parameterized tests, and other features that make testing more powerful and flexible.

- **Pylint:** [Pylint](https://pylint.readthedocs.io/en/stable/) is used for code linting to enforce coding standards and catch potential issues early. Maintaining a consistent code style is important, and Pylint helps achieve that goal.

- **Mypy and Pyright:** Both tools are used for type checking. [Mypy](https://mypy.readthedocs.io/en/stable/) helps verify that type annotations are correct, while [Pyright](https://github.com/microsoft/pyright) provides additional support for type completeness and validation.

- **Sphinx:** [Sphinx](https://www.sphinx-doc.org/en/master/) is used for generating package documentation. This ensures that contributors and users alike have clear, accessible information about how to use the SDK.

- **Bandit:** [Bandit](https://bandit.readthedocs.io/en/latest/) is employed to find common security issues in Python code. It performs static analysis and helps us secure the codebase.

- **Azure DevOps:** Our CI/CD pipelines are managed using [Azure DevOps](https://azure.microsoft.com/en-us/products/devops/), ensuring that builds, tests, and deployments are executed consistently and reliably across all contributions.

## Building and Testing

The Azure SDK team's Python CI leverages the tool `tox` to distribute tests to virtual environments, handle test dependency installation, and coordinate tooling reporting during PR/CI builds. This means that a developer working locally can exactly reproduce what the build machine is doing.

[A Brief Overview of Tox](https://tox.wiki/en/latest/)

#### A Monorepo and Tox in Harmony

Traditionally, the `tox.ini` file for a package sits alongside the `setup.py` in the source code. The `azure-sdk-for-python` necessarily does not adhere to this policy. There are over one hundred packages contained herein. That's a lot of `tox.ini` files to maintain!

Instead, the CI system leverages the `--root` argument, which is new to `tox4`. The `--root` argument allows `tox` to act as if the `tox.ini` is located in whatever directory you specify!

#### Tox Environments

A given `tox.ini` works on the concept of **test environments**. A given test environment is a combination of:

1. An identifier (or multiple identifiers).
2. A targeted Python version.
    1. `tox` will default to the base Python executing the `tox` command if no Python environment is specified.
3. (Optionally) an OS platform.

Internally, `tox` leverages `virtualenv` to create each test environment's virtual environment.

This means that once the `tox` workflow is established, all tests will be executed within a virtual environment.

You can use the command `tox list` to list all the environments provided by a `tox.ini` file. You can either use that command in the same directory as the file itself or use the `--conf` argument to specify the path to it directly.

**Sample output of `tox list`:**

```
sdk-for-python/eng/tox> tox list
default environments:
whl              -> Builds a wheel and runs tests
sdist            -> Builds a source distribution and runs tests

additional environments:
pylint           -> Lints a package with a pinned version of pylint
next-pylint      -> Lints a package with pylint (version 2.15.8)
mypy             -> Typechecks a package with mypy (version 1.9.0)
next-mypy        -> Typechecks a package with the latest version of mypy
pyright          -> Typechecks a package with pyright (version 1.1.287)
next-pyright     -> Typechecks a package with the latest version of the static type-checker pyright
verifytypes      -> Verifies the "type completeness" of a package with pyright
whl_no_aio       -> Builds a wheel without aio and runs tests
develop          -> Tests a package
sphinx           -> Builds a package's documentation with Sphinx
depends          -> Ensures all modules in a target package can be successfully imported
verifywhl        -> Verifies directories included in the wheel and contents in the manifest file
verifysdist      -> Verifies directories included in sdist and contents in the manifest file. Also ensures that `py.typed` configuration is correct within the setup.py.
devtest          -> Tests a package against dependencies installed from a dev index
latestdependency -> Tests a package against the released, upper-bound versions of its Azure dependencies
mindependency    -> Tests a package against the released, lower-bound versions of its Azure dependencies
apistub          -> Generates an API stub of a package (for https://apiview.dev)
bandit           -> Runs Bandit, a tool to find common security issues, against a package
samples          -> Runs a package's samples
breaking         -> Runs the breaking changes checker against a package
```

### Example Usage of the Common Azure SDK for Python `tox.ini`

Basic usage of `tox` within this monorepo is:

1. `pip install "tox<5"`
2. Run `tox run -e ENV_NAME -c path/to/tox.ini --root path/to/python_package`
   - **Note:** You can use environment variables to provide defaults for tox config values.
     - With `TOX_CONFIG_FILE` set to the absolute path of `tox.ini`, you can avoid needing `-c path/to/tox.ini` in your tox invocations.
     - With `TOX_ROOT_DIR` set to the absolute path to your Python package, you can avoid needing `--root path/to/python_package`.

The common `tox.ini` location is `eng/tox/tox.ini` within the repository.

If at any time you want to blow away the tox-created virtual environments and start over, simply append `-r` to any tox invocation!

#### Example `azure-core` mypy

1. Run `tox run -e mypy -c ./eng/tox/tox.ini --root sdk/core/azure-core`

#### Example `azure-storage-blob` tests

2. Execute `tox run -c ./eng/tox/tox.ini --root sdk/storage/azure-storage-blob`

Note that we didn't provide an `environment` argument for this example. The reason here is that the default environment selected by our common `tox.ini` file is one that runs `pytest`.

#### `whl` Environment
Used for test execution across the spectrum of all the platforms we want to support. Maintained at a platform-specific level just in case we run into platform-specific bugs.

* Installs the wheel and runs tests using the wheel.

```
> tox run -e whl -c <path to tox.ini> --root <path to python package>
```

#### `sdist` Environment
Used for the local dev loop.

* Installs the package in editable mode.
* Runs tests using the editable mode installation, not the wheel.

```
> tox run -e sdist -c <path to tox.ini> --root <path to python package>
```

#### `pylint` Environment
Pylint install and run.

```
> tox run -e pylint -c <path to tox.ini> --root <path to python package>
```

#### `mypy` Environment
Mypy install and run.

```
> tox run -e mypy -c <path to tox.ini> --root <path to python package>
```

#### `sphinx` Environment
Generate Sphinx documentation for this package.

```
> tox run -e sphinx -c <path to tox.ini> --root <path to python package>
```

### Custom Pytest Arguments

`tox` supports custom arguments, and the defined pytest environments within the common `tox.ini` also allow these. Essentially, separate the arguments you want passed to `pytest` with a `--` in your tox invocation.

**Example:** Invoke tox, breaking into the debugger on failure.

```
tox run -e whl -c <path to tox.ini

> --root <path to python package> -- -p pdb
```

### Performance Testing

For full details on this framework, and how to write and run tests for an SDK, see the [perfstress tests documentation](https://github.com/Azure/azure-sdk-for-python/blob/main/sdk/metrics/tests/perfstress/README.md).

### More Reading

We maintain an [additional document](https://github.com/Azure/azure-sdk-for-python/blob/main/sdk/metrics/README.md) that contains detailed information regarding what is actually happening in these executions.

### Samples

The SDK's samples are intended to be executed when necessary to demonstrate the usage of an Azure SDK package; they should not be suggested to users as a best practice.

### Code of Conduct

This project's code of conduct is defined in the [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) file. By participating, you are expected to uphold this code.
