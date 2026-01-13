# Verification

Verification asserts that specific interactions happened during the test. Use `verify()` after exercising the code under test.

## The `verify()` DSL

All verification starts with `verify()`, followed by the type of interaction:

- `verify().call(mock.method(...))` — verify a method was called
- `verify().get(mock.field)` — verify a field was read (see [Fields & Properties](fields.md))
- `verify().set(mock.field, value)` — verify a field was written (see [Fields & Properties](fields.md))

Each must end with an assertion method.

## Assertion Methods

**`.once()`** — Assert exactly one matching call.

**`.times(n)`** — Assert exactly n matching calls.

**`.never()`** — Assert no matching calls. Equivalent to `.times(0)`.

**`.called()`** — Assert at least one matching call.

**`.at_least(n)`** — Assert n or more matching calls.

**`.at_most(n)`** — Assert n or fewer matching calls.

## Using Matchers

Verification supports [argument matchers](matchers.md) to match a range of calls:

```python
mock.save(1)
mock.save(2)
mock.save(3)

verify().call(mock.save(any(int))).times(3)  # All three matched
verify().call(mock.save(1)).once()           # Just the first
```

## Argument Normalization

Positional and keyword arguments are treated equivalently—how you called the method doesn't have to match how you verify it:

```python
mock.add(1, 2)                            # Called with positional args
verify().call(mock.add(a=1, b=2)).once()  # Verified with kwargs — works
```

## Custom Error Messages

Add context to verification failures by passing `error_message` to any assertion method:

```python
verify().call(mock.action(1)).once(error_message="Setup should call action(1)")
```

The custom message appears before the original error, making test failures easier to understand.

## Incomplete Verifications

Forgetting to call `.once()` (or another assertion) is detected on the next mock operation and raises `TMockVerificationError`.