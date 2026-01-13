# Async Support

tmock handles async methods transparently—you stub them the same way as sync methods, and tmock wraps the return value in a coroutine automatically.

## Stubbing Async Methods

```python
class AsyncService:
    async def fetch_data(self, id: int) -> str:
        return ""

mock = tmock(AsyncService)
given().call(mock.fetch_data(123)).returns("fetched")

result = await mock.fetch_data(123)
assert result == "fetched"
```

## Verification

Verification works the same as with sync methods:

```python
await mock.fetch_data(1)
await mock.fetch_data(2)

verify().call(mock.fetch_data(any(int))).times(2)
```

## Raising Exceptions

```python
given().call(mock.fetch_data(any(int))).raises(ValueError("not found"))

with pytest.raises(ValueError):
    await mock.fetch_data(123)
```

## Dynamic Responses

Use `.runs()` with a **sync** callback—async callbacks are not supported:

```python
given().call(mock.process(any(int))).runs(
    lambda args: args.get_by_name("value") * 2
)

result = await mock.process(21)
assert result == 42
```

## Mixed Sync/Async Classes

Classes with both sync and async methods work naturally:

```python
class MixedService:
    def sync_method(self, x: int) -> int: ...
    async def async_method(self, x: int) -> int: ...

mock = tmock(MixedService)
given().call(mock.sync_method(1)).returns(100)
given().call(mock.async_method(2)).returns(200)

sync_result = mock.sync_method(1)
async_result = await mock.async_method(2)
```

## Async Context Managers

See [Magic Methods](magic-methods.md) for mocking `__aenter__` and `__aexit__`.