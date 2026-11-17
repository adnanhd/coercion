![](images/coercion.png)

# coercion
![coverage](images/coverage.svg)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

A fork of [kremrik/coercion](https://github.com/kremrik/coercion), version
0.1.1; the code is upstream's but for the annotation `schema_val: type` in
`_on_mismatch` and line breaks.

### Installation
Installable from source, or by pip installing from github:
```
python -m pip install git+https://github.com/adnanhd/coercion.git
```
The one dependency, `map_ops`, comes from
`git+https://github.com/kremrik/map-ops@master` (`requirements.txt`).

### What is it?
A dead-simple, dictionary-based, schema framework for shaping and _coercing_
arbitrarily complex data

### What is it _not_?
A schema validator. `coercion` makes every attempt to fill in the blanks and
cast your data to the appropriate types, but employs a "fail fast" approach to
exceptions and does not provide reporting.

### Why coercion?
Two reasons:
1. It's good to have a "gatekeeper" at the bounds of applications where data
may be missing fields, have messed up types, etc
1. `coercion` schemas are just Python - no DSL's, no custom objects, no serde;
if you know how to create a dictionary, you already know everything necessary

### Examples

##### Conform to a schema
```python
from coercion import coerce

# schemas are just dictionaries, with the types specified
# as regular python `type`s:
schema = {"foo": int, "bar": float}
record = {"foo": "1", "baz": "remove-me"}

coerce(schema, record)
{"foo": 1, "bar": None}

# you can alter the default value for missing keys too:
coerce(schema, record, {float: 0.0})
{"foo": 1, "bar": 0.0}
```

##### Operating on lists of dicts
```python
from coercion import coerce

schema = {
    "foo": [
        {"bar": str}
    ]
}
record = {
    "foo": [
        {"bar": 1},
        {"baz": "remove-me"},
        {"bar": "hi"}
    ]
}

coerce(schema, record)
{
    "foo": [
        {"bar": '1'},
        {"bar": None},
        {"bar": "hi"}
    ]
}
```

##### Operating with string representations
```python
from coercion import coerce

schema2 = {"foo": [float]}
record2 = {"foo": "[1, 2, 3]"}  # list is a string
coerce(schema2, record2)
{'foo': [1.0, 2.0, 3.0]}

schema3 = {"foo": {"bar": str}}
record3 = {"foo": '{"bar": 1}'}  # dict is a string
coerce(schema3, record3)
{'foo': {'bar': '1'}}
```

##### Error reporting
Debugging errors while parsing deeply-nested data structures can be a pain.
`coercion` makes this chore easier by providing a simple path on error:
```python
from coercion import coerce

schema = {"foo": {"bar": {"baz": float}}}
record = {"foo": {"bar": {"baz": "hi"}}}

coerce(schema, record)
...
ValueError: foo -> bar -> baz: could not convert string to float: 'hi'
```

### Development
`make check` runs the unit tests (`pytest`), the doctests of
`coercion/coercion.py`, coverage (writing `images/coverage.svg` with
`coverage-badge`), mypy, black and flake8, the last two with a 59-character
line limit that 12 lines of `coercion/coercion.py` exceed. The tools are in
`dev_requirements.txt`. `.githooks/pre-commit` runs `make check` and then
`git add .`; it is active only with `git config core.hooksPath .githooks`.
`make sphinx` expects a `sphinx/` directory, which the repository does not
have. `.github/workflows/python-package.yml` runs flake8, mypy and pytest on
Python 3.5 to 3.8.

### On this machine
The checkout is `~/src/personal/coercion`. Neither `coercion` nor `map_ops`
is installed, so the tests do not run here.

### License
MIT, see `LICENSE`.
