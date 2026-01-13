# Fields & Properties

tmock supports mocking field access (getters and setters) on classes, dataclasses, Pydantic models, and properties.

## Supported Field Sources

tmock automatically discovers fields from:

- `@property` decorated methods
- `@dataclass` fields
- Pydantic `BaseModel` fields
- Class-level type annotations

For fields only assigned in `__init__` (not discoverable via introspection), use `extra_fields`. Prefer adding type annotations to your class when possible—they provide type checking and are automatically discovered:

```python
# Preferred: add annotations
class Person:
    name: str
    age: int

# Fallback: use extra_fields for legacy code
class LegacyPerson:
    def __init__(self):
        self.name = ""
        self.age = 0

mock = tmock(LegacyPerson, extra_fields=["name", "age"])
```

## Stubbing Getters

Use `given().get()` to stub field reads:

```python
mock = tmock(Person)
given().get(mock.name).returns("Alice")

assert mock.name == "Alice"
assert mock.name == "Alice"  # Can be read multiple times
```

## Stubbing Setters

Use `given().set()` to stub field writes. The second argument is the expected value:

```python
given().set(mock.name, "Alice").returns(None)

mock.name = "Alice"  # works
mock.name = "Bob"    # TMockUnexpectedCallError
```

Use [matchers](matchers.md) to accept any value:

```python
given().set(mock.name, any(str)).returns(None)

mock.name = "Alice"  # works
mock.name = "Bob"    # works
```

## Verifying Field Access

```python
given().get(mock.name).returns("Alice")
given().set(mock.name, any(str)).returns(None)

_ = mock.name
mock.name = "Bob"

verify().get(mock.name).once()
verify().set(mock.name, "Bob").once()
```

## Read-Only Fields

Attempting to stub a setter on a read-only property, frozen dataclass, or frozen Pydantic model raises `TMockStubbingError`:

```python
@dataclass(frozen=True)
class User:
    name: str

mock = tmock(User)
given().set(mock.name, "value").returns(None)  # TMockStubbingError: read-only
```

## Type Validation

Field stubs are type-checked against the field's annotation:

```python
class Person:
    name: str

mock = tmock(Person)
given().get(mock.name).returns(123)  # TMockStubbingError: expected str
```

Fields from `extra_fields` have `Any` type and skip validation.