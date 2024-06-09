# Contributing to the Logfire SDK and docs

We'd love anyone interested to contribute to the Logfire SDK and documentation.

## How to contribute

1. Fork and clone the repository
2. [Install Rye](https://rye-up.com/guide/basics/)
3. Run `make install` to install dependencies
4. Run `make test` to run unit tests
5. To run tests in a specific directory:
   - `make test TEST_ARGS="-k <directory_path>"`
6. To run a specific test:
   - `make test TEST_ARGS="-k <test_name>"`
7. Run `make format` to format code
8. Run `make lint` to lint code
9. Run `make docs` to build docs and `make docs-serve` to serve docs locally

You're now set up to start contributing!
