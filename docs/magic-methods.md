# Magic Methods

tmock supports mocking Python's magic methods for context managers, iteration, containers, and more.

## Context Managers

Stub both `__enter__` and `__exit__` to use a mock as a context manager:

```python
class FileManager:
    def __enter__(self) -> Resource: ...
    def __exit__(self, exc_type, exc_val, exc_tb) -> bool | None: ...

mock = tmock(FileManager)
resource = tmock(Resource)

given().call(mock.__enter__()).returns(resource)
given().call(mock.__exit__(None, None, None)).returns(None)

with mock as res:
    assert res is resource
```

To handle exceptions inside the block, use [matchers](matchers.md) for the exception arguments:

```python
given().call(mock.__exit__(any(), any(), any())).returns(False)  # Don't suppress
```

Return `True` from `__exit__` to suppress exceptions.

## Async Context Managers

```python
class AsyncManager:
    async def __aenter__(self) -> Resource: ...
    async def __aexit__(self, exc_type, exc_val, exc_tb) -> bool | None: ...

mock = tmock(AsyncManager)
given().call(mock.__aenter__()).returns(resource)
given().call(mock.__aexit__(None, None, None)).returns(None)

async with mock as res:
    assert res is resource
```

## Callable Objects

```python
class Handler:
    def __call__(self, x: int) -> str: ...

mock = tmock(Handler)
given().call(mock(42)).returns("handled")

assert mock(42) == "handled"
```

## Container Methods

```python
class Container:
    def __len__(self) -> int: ...
    def __getitem__(self, key: str) -> int: ...
    def __contains__(self, item: str) -> bool: ...

mock = tmock(Container)
given().call(mock.__len__()).returns(3)
given().call(mock.__getitem__("key")).returns(42)
given().call(mock.__contains__("item")).returns(True)

assert len(mock) == 3
assert mock["key"] == 42
assert "item" in mock
```

## Iteration

```python
class Iterable:
    def __iter__(self) -> Iterator[str]: ...

mock = tmock(Iterable)
given().call(mock.__iter__()).returns(iter(["a", "b", "c"]))

assert list(mock) == ["a", "b", "c"]
```

## String Representations

```python
class Printable:
    def __str__(self) -> str: ...
    def __repr__(self) -> str: ...

mock = tmock(Printable)
given().call(mock.__str__()).returns("string form")
given().call(mock.__repr__()).returns("repr form")

assert str(mock) == "string form"
assert repr(mock) == "repr form"
```