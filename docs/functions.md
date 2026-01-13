# Mocking Functions

`tmock()` can mock standalone functions, not just classes. This is useful for mocking callbacks or testing function-based code.

## Basic Usage

Pass a function to `tmock()` to create a callable mock:

```python
def add(x: int, y: int) -> int:
    return x + y

mock = tmock(add)

given().call(mock(1, 2)).returns(100)
assert mock(1, 2) == 100

verify().call(mock(1, 2)).once()
```

## With Matchers

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

mock = tmock(greet)
given().call(mock(any(str))).returns("Hi!")

assert mock("Alice") == "Hi!"
assert mock("Bob") == "Hi!"
```

## Async Functions

Async functions work the same way:

```python
async def fetch_data(id: int) -> dict:
    return {"id": id}

mock = tmock(fetch_data)
given().call(mock(1)).returns({"id": 1, "mocked": True})

result = await mock(1)
```

## As Callbacks

Function mocks can be passed where a callable is expected:

```python
def run_with_callback(callback):
    return callback(5, 5)

mock = tmock(add)
given().call(mock(5, 5)).returns(10)

result = run_with_callback(mock)
assert result == 10
```

## Type Validation

Function mocks validate argument and return types just like class mocks:

```python
mock = tmock(add)

given().call(mock("1", 2))            # TMockStubbingError: invalid arg type
given().call(mock(1, 2)).returns("x") # TMockStubbingError: invalid return type
```

## Limitations

`extra_fields` is not supported for function mocks—it only applies to classes.