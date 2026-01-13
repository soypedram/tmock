# Stubbing

Stubbing defines what a mock returns when called. Unlike `unittest.mock`, tmock requires you to declare behavior upfront—calling an unstubbed method raises `TMockUnexpectedCallError`.

## The `given()` DSL

All stubbing starts with `given()`, followed by the type of interaction:

- `given().call(mock.method(...))` — stub a method call
- `given().get(mock.field)` — stub a field read (see [Fields & Properties](fields.md))
- `given().set(mock.field, value)` — stub a field write (see [Fields & Properties](fields.md))

Each must end with a response: `.returns()`, `.raises()`, or `.runs()`.

## Response Types

**`.returns(value)`** — Return a fixed value. This is the most common case.

```python
given().call(mock.get_user(123)).returns({"name": "Alice"})
```

**`.raises(exception)`** — Raise an exception. Useful for testing error handling paths.

```python
given().call(mock.connect()).raises(ConnectionError("Timeout"))
```

**`.runs(callback)`** — Execute a function to compute the return value dynamically. The callback receives a `CallArguments` object with access to the arguments via `get_by_name()`.

```python
given().call(mock.add(any(int), any(int))).runs(
    lambda args: args.get_by_name("a") + args.get_by_name("b")
)
```

You can also use `.runs()` to capture calls for assertions that `verify()` can't express, like call order:

```python
call_order: list[str] = []
given().call(mock.log(any(str))).runs(
    lambda args: call_order.append(args.get_by_name("level"))
)

mock.log("info")
mock.log("warning")
mock.log("error")

assert call_order == ["info", "warning", "error"]
```

## Using Matchers

Stubs can use [argument matchers](matchers.md) like `any()` to match a range of inputs:

```python
given().call(mock.save(any(dict))).returns(True)  # Matches any dict
```

## Stub Priority

When multiple stubs could match a call, the **last defined stub wins**. This means order matters—define general fallbacks first, then specific overrides:

```python
given().call(mock.foo(any(int))).returns("fallback")
given().call(mock.foo(1)).returns("specific")

mock.foo(1)   # "specific" — later stub wins
mock.foo(99)  # "fallback"
```

Be careful: if you reverse the order, the `any(int)` stub would match everything, including `1`.

## Returning Mocks

You can return a mock from a stubbed method to test chained dependencies:

```python
session = tmock(Session)
given().call(factory.create_session()).returns(session)
given().call(session.execute(any(str))).returns([{"id": 1}])
```

## Type Validation

tmock validates argument and return types at stub time. If your callback in `.runs()` returns the wrong type, the error is raised when the mock is called.

## Incomplete Stubs

Forgetting to call `.returns()` (or another response method) is detected on the next mock operation and raises `TMockStubbingError` with a helpful message showing what was left incomplete.