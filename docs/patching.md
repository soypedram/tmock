# Patching

Use `tpatch` when you need to replace something you don't control—a module function, a method on a class you didn't instantiate, or a module variable. It wraps `unittest.mock.patch` but provides tmock's type-safe DSL.

## When to Use

- **`tmock()`** — You control the dependency and pass it in (dependency injection)
- **`tpatch`** — You need to replace something used internally by the code under test

## Patch Methods

Each method is explicit about what it patches, with validation to catch mistakes:

**`tpatch.method(cls, name)`** — Instance methods

```python
with tpatch.method(Calculator, "add") as mock:
    given().call(mock(1, 2)).returns(42)
    assert Calculator().add(1, 2) == 42
```

**`tpatch.function(path)`** — Module-level functions

```python
with tpatch.function("myapp.utils.get_timestamp") as mock:
    given().call(mock()).returns(1234567890)
```

**`tpatch.staticmethod(cls, name)`** — Static methods

```python
with tpatch.staticmethod(IdGenerator, "generate") as mock:
    given().call(mock()).returns("mock-id")
```

**`tpatch.classmethod(cls, name)`** — Class methods

```python
with tpatch.classmethod(Config, "from_env") as mock:
    given().call(mock()).returns(Config(debug=True))
```

**`tpatch.field(cls, name)`** — Instance fields (properties, dataclass fields, Pydantic fields)

```python
with tpatch.field(Person, "name") as field:
    given().get(field).returns("Alice")
    given().set(field, any(str)).returns(None)
```

**`tpatch.class_var(cls, name)`** — Class variables

```python
with tpatch.class_var(Config, "MAX_RETRIES") as field:
    given().get(field).returns(1)
```

**`tpatch.module_var(path, name)`** — Module-level variables

```python
with tpatch.module_var("myapp.config", "DEBUG") as field:
    given().get(field).returns(True)
```

## Scope and Restoration

Patches are active only within the `with` block. The original is restored on exit:

```python
with tpatch.method(Calculator, "add") as mock:
    given().call(mock(1, 2)).returns(42)
    assert Calculator().add(1, 2) == 42

assert Calculator().add(1, 2) == 3  # Original restored
```

## Error Guidance

Using the wrong patch method gives helpful errors pointing to the correct one:

```python
with tpatch.method(MyClass, "static_func"):
    ...
# TMockPatchingError: 'static_func' is a staticmethod. Use tpatch.staticmethod().
```

## Async Methods

Async methods work the same way—tmock handles the coroutine wrapping:

```python
with tpatch.method(Service, "fetch_async") as mock:
    given().call(mock(1)).returns({"id": 1})
    result = await Service().fetch_async(1)
```