# Protocols

tmock supports mocking `typing.Protocol` interfaces, letting you test code that depends on protocol-typed dependencies.

## Basic Usage

```python
from typing import Protocol

class Logger(Protocol):
    def log(self, message: str) -> None: ...

mock = tmock(Logger)
given().call(mock.log(any(str))).returns(None)

mock.log("test message")
verify().call(mock.log("test message")).once()
```

## Protocol Properties

Protocols can define properties, which are mocked like any other field:

```python
class DataStore(Protocol):
    def get_data(self, key: str) -> dict: ...

    @property
    def is_ready(self) -> bool: ...

mock = tmock(DataStore)
given().get(mock.is_ready).returns(True)
given().call(mock.get_data("key")).returns({"value": 42})
```

## Runtime Checkable Protocols

For `@runtime_checkable` protocols, mocks pass `isinstance` checks:

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Logger(Protocol):
    def log(self, message: str) -> None: ...

mock = tmock(Logger)
assert isinstance(mock, Logger)  # True
```

This is useful when the code under test uses `isinstance` to verify dependencies.

## Strictness

Protocol mocks are strict like class mocks—unstubbed calls raise `TMockUnexpectedCallError`:

```python
mock = tmock(Logger)
mock.log("unhandled")  # TMockUnexpectedCallError
```