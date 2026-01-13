# Reset Functions

Reset functions clear mock state, useful when reusing mocks across test phases or in parameterized tests.

## `reset(mock)`

Clears both call history and stubs—the mock returns to its initial strict state:

```python
mock = tmock(MyClass)
given().call(mock.greet(any(str))).returns("Hello")
mock.greet("Alice")

reset(mock)

verify().call(mock.greet(any(str))).never()  # History cleared
mock.greet("Bob")  # TMockUnexpectedCallError — stubs cleared too
```

## `reset_interactions(mock)`

Clears only call history, keeping stubs intact:

```python
mock = tmock(MyClass)
given().call(mock.greet(any(str))).returns("Hello")
mock.greet("Alice")

reset_interactions(mock)

assert mock.greet("Bob") == "Hello"  # Stub still works
verify().call(mock.greet("Alice")).never()  # But Alice call is forgotten
verify().call(mock.greet("Bob")).once()
```

## `reset_behaviors(mock)`

Clears only stubs, keeping call history:

```python
mock = tmock(MyClass)
given().call(mock.greet(any(str))).returns("Hello")
mock.greet("Alice")

reset_behaviors(mock)

verify().call(mock.greet("Alice")).once()  # History preserved
mock.greet("Bob")  # TMockUnexpectedCallError — stub gone
```

## With Fields

Reset functions also clear field getter and setter state:

```python
mock = tmock(Person)
given().get(mock.name).returns("Alice")
given().set(mock.name, any(str)).returns(None)

_ = mock.name
mock.name = "Bob"

reset(mock)

verify().get(mock.name).never()
verify().set(mock.name, any(str)).never()
```