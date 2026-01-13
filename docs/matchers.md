# Argument Matchers

Matchers let you stub and verify calls without specifying exact argument values.

## `any()`

Match any value, regardless of type:

```python
given().call(mock.process(any())).returns("ok")

mock.process(42)        # matches
mock.process("hello")   # matches
mock.process([1, 2, 3]) # matches
```

## `any(Type)`

Match any value of a specific type:

```python
given().call(mock.save(any(dict))).returns(True)

mock.save({"key": "value"})  # matches
mock.save("not a dict")      # TMockUnexpectedCallError
```

## In Stubbing vs Verification

Matchers work the same in both contexts:

```python
# Stubbing: match any int
given().call(mock.get(any(int))).returns("found")

# Verification: count calls matching any int
verify().call(mock.get(any(int))).times(3)
```

You can also mix matchers with literal values:

```python
given().call(mock.fetch(any(int), "active")).returns(data)

mock.fetch(1, "active")      # matches
mock.fetch(999, "active")    # matches
mock.fetch(1, "inactive")    # TMockUnexpectedCallError
```

## Type Matching in Verification

When verifying, `any(Type)` only counts calls where the argument was that type:

```python
mock.process(42)

verify().call(mock.process(any(int))).once()  # passes
verify().call(mock.process(any(str))).never() # passes — 42 is not a str
```