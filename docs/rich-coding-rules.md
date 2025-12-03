# Rich Coding Rules

These are the coding conventions, patterns, and best practices used in the Rich library.
Following these rules ensures consistency and maintains the high quality of the codebase.

---

## Table of Contents

1. [Python Version and Compatibility](#python-version-and-compatibility)
2. [Project Setup and Developer Workflows](#project-setup-and-developer-workflows)
3. [Code Formatting and Style](#code-formatting-and-style)
4. [Import Conventions](#import-conventions)
5. [Type Annotations](#type-annotations)
6. [Naming Conventions](#naming-conventions)
7. [Class Design Patterns](#class-design-patterns)
8. [Rich Protocol Patterns](#rich-protocol-patterns)
9. [Docstring and Comment Guidelines](#docstring-and-comment-guidelines)
10. [Error Handling](#error-handling)
11. [Performance Patterns](#performance-patterns)
12. [Testing Guidelines](#testing-guidelines)
13. [Module Organization](#module-organization)
14. [Deprecation Patterns](#deprecation-patterns)
15. [Open Questions](#open-questions)

---

## Python Version and Compatibility

### Supported Versions
- Rich supports Python 3.8 through 3.14.
- Code must be compatible with Python 3.8+ syntax and features.
- Do NOT use Python 3.9+ features like `dict | dict` union or `list[str]` builtin generics
  in runtime code (only allowed in `TYPE_CHECKING` blocks or string annotations).

### Type Annotation Compatibility
```python
# Use typing module generics for runtime compatibility
from typing import List, Dict, Optional, Union

# These are fine for Python 3.8+
def example(items: List[str], mapping: Dict[str, int]) -> Optional[str]:
    ...
```

### Future Annotations
- Some modules use `from __future__ import annotations` for PEP 563 postponed evaluation.
- This allows using string annotations without quotes and enables forward references.
```python
from __future__ import annotations

# With future annotations, no quotes needed for forward references
def method(self) -> Style:  # Instead of -> "Style"
    ...
```

---

## Project Setup and Developer Workflows

### Package Management
- Rich uses **Poetry** for packaging and dependency management.
- Install dependencies with `poetry install`.
- Enter the virtual environment with `poetry shell`.

### Development Commands
```shell
# Run tests with coverage
make test

# Run tests without coverage (faster)
make test-no-cov

# Check code formatting
make format-check

# Apply code formatting
make format

# Run type checking
make typecheck

# Build documentation
make docs
```

### Pre-commit Hooks
- Install pre-commit hooks for automatic checks: `pre-commit install`
- Hooks run Black formatting and other checks before each commit.

### Before Submitting Changes
1. Run `make test` and ensure all tests pass.
2. Run `make typecheck` and resolve any type errors.
3. Run `make format` to format code with Black.
4. Update `CHANGELOG.md` with a description of changes.
5. Add yourself to `CONTRIBUTORS.md` if this is your first contribution.

---

## Code Formatting and Style

### Black Formatter
- All code is formatted with Black.
- Use default Black settings (88 character line length).
- Set up your editor to format on save.

### Line Length
- Maximum line length is 88 characters (Black default).
- Use line continuation for long expressions.

### Blank Lines
- Two blank lines between top-level definitions (classes, functions).
- One blank line between methods within a class.
- No blank line after a function/method docstring before the code.

### String Quotes
- Use double quotes for strings by default.
- Single quotes are acceptable for nested strings.

### Trailing Commas
- Use trailing commas in multi-line collections and function calls.
```python
# Good
my_list = [
    "item1",
    "item2",
    "item3",
]

# Good
result = some_function(
    arg1="value1",
    arg2="value2",
)
```

### Disable Formatter When Needed
- Use `# fmt: off` and `# fmt: on` comments for special formatting needs.
- Common use case: Box character definitions that require exact alignment.
```python
# fmt: off
ASCII: Box = Box(
    "+--+\n"
    "| ||\n"
    "|-+|\n"
    ...
)
# fmt: on
```

---

## Import Conventions

### Import Order
Imports should be organized in this order (Black/isort handles this):
1. Standard library imports
2. Third-party imports
3. Local application imports

### Relative Imports
- Use relative imports for internal module references.
- This is the established pattern in Rich.
```python
# Good - relative imports within the package
from .style import Style, StyleType
from .text import Text, TextType
from .segment import Segment
from . import errors

# Also acceptable for longer paths
from ._loop import loop_first, loop_last
```

### TYPE_CHECKING Guard
- Use `TYPE_CHECKING` to avoid circular imports.
- Import types only needed for annotations inside the guard.
```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from .console import Console, ConsoleOptions, RenderResult
```

### Import Aliasing for Performance
- When a module/class is used heavily in loops, import it locally.
```python
def some_method(self) -> None:
    _Segment = Segment  # Local alias for performance
    for item in items:
        yield _Segment(...)
```

### Noqa Comments
- Use `# noqa: F401` for re-exports that appear unused.
```python
from ._extension import load_ipython_extension  # noqa: F401
```

---

## Type Annotations

### General Guidelines
- Type annotate all public functions and methods.
- Use type annotations from the `typing` module.
- Rich uses mypy with strict mode enabled.

### Union and Optional
- Use `Optional[X]` for values that can be `None`.
- Use `Union[X, Y]` for multiple possible types.
```python
from typing import Optional, Union

def example(
    color: Optional[str] = None,
    value: Union[int, str] = 0,
) -> Optional[str]:
    ...
```

### Type Aliases
- Define type aliases for commonly used complex types.
- Place type aliases near the top of the module.
```python
# Type aliases
StyleType = Union[str, "Style"]
TextType = Union[str, "Text"]
RenderableType = Union[ConsoleRenderable, RichCast, str]
```

### Forward References
- Use string annotations for forward references.
```python
def method(self) -> "Style":
    ...
```

### TypeVar for Generics
- Use `TypeVar` for generic type parameters.
```python
from typing import TypeVar

T = TypeVar("T")

def loop_first(values: Iterable[T]) -> Iterable[Tuple[bool, T]]:
    ...
```

### Return Type Annotations
- Always specify return types.
- Use `-> None` for functions that don't return a value.
- Use `-> "ClassName"` for factory methods returning self type.

### mypy Ignore Comments
- Use `# type: ignore[error-code]` when suppressing mypy errors.
- Always include the specific error code.
```python
_console.__dict__ = new_console.__dict__  # type: ignore[assignment]
```

### NewType for Type-Safe Aliases
- Use `NewType` to create distinct types for type checking.
- Useful for IDs and other values that shouldn't be mixed.
```python
from typing import NewType

TaskID = NewType("TaskID", int)

def get_task(task_id: TaskID) -> Task:
    ...
```

### ClassVar for Class-Level Attributes
- Use `ClassVar` to annotate class-level variables.
```python
from typing import ClassVar

class MarkdownElement:
    new_line: ClassVar[bool] = True
```

### Overload for Multiple Signatures
- Use `@overload` for methods with different return types based on input.
```python
from typing import overload, Union, List

@overload
def __getitem__(self, index: int) -> "Text":
    ...

@overload
def __getitem__(self, index: slice) -> List["Text"]:
    ...

def __getitem__(self, index: Union[slice, int]) -> Union["Text", List["Text"]]:
    return self._lines[index]
```

### Self Type from typing_extensions
- Use `Self` from `typing_extensions` for methods returning self.
- Required for Python 3.8-3.10 compatibility.
```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from typing_extensions import Self  # pragma: no cover

def __enter__(self) -> "Self":
    self.start()
    return self
```

### Literal Types for Constrained Strings
- Use `Literal` for string parameters with a fixed set of valid values.
```python
from typing import Literal

JustifyMethod = Literal["default", "left", "center", "right", "full"]
OverflowMethod = Literal["fold", "crop", "ellipsis", "ignore"]
AlignMethod = Literal["left", "center", "right"]

def render(self, justify: JustifyMethod = "left") -> Text:
    ...
```

### Protocol for Duck Typing
- Use `Protocol` with `@runtime_checkable` for structural typing.
```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class ConsoleRenderable(Protocol):
    """An object that may be rendered by the Console."""

    def __rich_console__(
        self, console: "Console", options: "ConsoleOptions"
    ) -> "RenderResult":
        ...
```

---

## Naming Conventions

### General Rules
- Use descriptive names; avoid abbreviations.
- Follow PEP 8 naming conventions.

### Variables and Functions
- Use `snake_case` for variables and functions.
```python
def get_style_at_offset(self, offset: int) -> Style:
    ...
```

### Classes
- Use `PascalCase` for class names.
```python
class ConsoleOptions:
    ...
```

### Constants
- Use `UPPER_SNAKE_CASE` for module-level constants.
```python
WINDOWS = sys.platform == "win32"
NULL_STYLE = Style()
DEFAULT_JUSTIFY: "JustifyMethod" = "default"
```

### Private Members
- Prefix private attributes and methods with single underscore.
- Internal/utility modules are prefixed with underscore.
```python
# Private attributes
self._spans: List[Span] = []
self._text: List[str] = [text]

# Private methods
def _trim_spans(self) -> None:
    ...
```

### Internal Modules
- Internal utility modules start with underscore.
- Examples: `_loop.py`, `_pick.py`, `_ratio.py`, `_wrap.py`

### Type Aliases (Module-level)
- Type aliases typically use `PascalCase`.
```python
StyleType = Union[str, "Style"]
RenderResult = Iterable[Union[RenderableType, Segment]]
```

---

## Class Design Patterns

### __slots__
- Use `__slots__` for classes that will have many instances.
- Improves memory efficiency and attribute access speed.
```python
class Style:
    __slots__ = [
        "_color",
        "_bgcolor",
        "_attributes",
        "_set_attributes",
        "_link",
        "_link_id",
        "_ansi",
        "_style_definition",
        "_hash",
        "_null",
        "_meta",
    ]
```

### NamedTuple for Immutable Data
- Use `NamedTuple` for simple immutable data structures.
- Add docstrings to describe the tuple and its fields.
```python
class Span(NamedTuple):
    """A marked up region in some text."""

    start: int
    """Span start index."""
    end: int
    """Span end index."""
    style: Union[str, Style]
    """Style associated with the span."""
```

### Dataclasses for Configuration
- Use `@dataclass` for configuration-like classes with defaults.
```python
@dataclass
class Column:
    """Defines a column within a Table."""

    header: "RenderableType" = ""
    """RenderableType: Renderable for the header."""

    footer: "RenderableType" = ""
    """RenderableType: Renderable for the footer."""

    width: Optional[int] = None
    """Optional[int]: Width of the column."""
```

### Factory Methods
- Use `@classmethod` for alternative constructors.
- Name them descriptively: `from_*`, `parse`, `null`, etc.
```python
@classmethod
def null(cls) -> "Style":
    """Create a 'null' style, equivalent to Style(), but more performant."""
    return NULL_STYLE

@classmethod
def from_markup(cls, text: str, ...) -> "Text":
    """Create Text instance from markup."""
    ...

@classmethod
def parse(cls, style_definition: str) -> "Style":
    """Parse a style definition."""
    ...
```

### Copy Methods
- Implement `copy()` methods for mutable classes that need cloning.
```python
def copy(self) -> "Style":
    """Get a copy of this style."""
    if self._null:
        return NULL_STYLE
    style: Style = self.__new__(Style)
    style._ansi = self._ansi
    # ... copy all attributes
    return style
```

### Abstract Base Classes
- Use `ABC` and `@abstractmethod` for abstract interfaces.
```python
from abc import ABC, abstractmethod

class Highlighter(ABC):
    """Abstract base class for highlighters."""

    @abstractmethod
    def highlight(self, text: Text) -> None:
        """Apply highlighting in place to text."""
```

### Context Manager Protocol
- Implement `__enter__` and `__exit__` for resource management.
- Use `types.TracebackType` for proper type hints.
```python
from types import TracebackType
from typing import Optional, Type

def __enter__(self) -> "Live":
    self.start()
    return self

def __exit__(
    self,
    exc_type: Optional[Type[BaseException]],
    exc_val: Optional[BaseException],
    exc_tb: Optional[TracebackType],
) -> None:
    self.stop()
```

### Threading Patterns
- Use `RLock` for reentrant locking when methods may call each other.
- Use `Event` for thread signaling.
- Create daemon threads that stop when the main program exits.
```python
from threading import Event, RLock, Thread

class _RefreshThread(Thread):
    """A thread that calls refresh() at regular intervals."""

    def __init__(self, live: "Live", refresh_per_second: float) -> None:
        self.live = live
        self.refresh_per_second = refresh_per_second
        self.done = Event()
        super().__init__(daemon=True)

    def stop(self) -> None:
        self.done.set()

    def run(self) -> None:
        while not self.done.wait(1 / self.refresh_per_second):
            with self.live._lock:
                if not self.done.is_set():
                    self.live.refresh()
```

---

## Rich Protocol Patterns

Rich defines several protocols that classes can implement to integrate with the
rendering system.

### __rich_console__ Protocol
- Implement to make a class renderable by Rich.
- Takes `console` and `options`, yields `Segment` objects or other renderables.
```python
def __rich_console__(
    self, console: "Console", options: "ConsoleOptions"
) -> "RenderResult":
    """Render this object to the console."""
    yield Segment(self.text)
    yield Segment("\n")
```

### __rich_measure__ Protocol
- Implement to provide width hints for layout calculations.
- Returns a `Measurement` with minimum and maximum widths.
```python
def __rich_measure__(
    self, console: "Console", options: "ConsoleOptions"
) -> "Measurement":
    """Calculate the measurement for this renderable."""
    return Measurement(min_width, max_width)
```

### __rich__ Protocol
- Implement for simple delegation to another renderable.
- Returns a Rich renderable object.
```python
def __rich__(self) -> "Text":
    """Return a Rich renderable."""
    return Text(str(self))
```

### __rich_repr__ Protocol
- Implement for custom repr formatting with Rich.
- Works with the `@rich_repr` decorator.
```python
from rich.repr import rich_repr, Result

@rich_repr
class Style:
    def __rich_repr__(self) -> Result:
        yield "color", self.color, None
        yield "bgcolor", self.bgcolor, None
        yield "bold", self.bold, None
```

### JupyterMixin
- Inherit from `JupyterMixin` to enable Jupyter notebook rendering.
```python
from rich.jupyter import JupyterMixin

class Panel(JupyterMixin):
    """A console renderable that draws a border around its contents."""
    ...
```

---

## Docstring and Comment Guidelines

### Docstring Format
- Use triple-quoted docstrings with the opening quotes on their own line.
- Follow Google-style docstring format with Args/Returns sections.
```python
def get_style_at_offset(self, console: "Console", offset: int) -> Style:
    """Get the style of a character at given offset.

    Args:
        console (~Console): Console where text will be rendered.
        offset (int): Offset into text (negative indexing supported).

    Returns:
        Style: A Style instance.
    """
```

### Class Docstrings
- Include comprehensive Args section for `__init__` parameters.
- Document the class purpose and usage.
```python
class Panel(JupyterMixin):
    """A console renderable that draws a border around its contents.

    Example:
        >>> console.print(Panel("Hello, World!"))

    Args:
        renderable (RenderableType): A console renderable object.
        box (Box): A Box instance that defines the look of the border.
        title (Optional[TextType], optional): Optional title. Defaults to None.
        ...
    """
```

### Docstring for NamedTuple/Dataclass Fields
- Use inline docstrings after field definitions.
```python
class Measurement(NamedTuple):
    """Stores the minimum and maximum widths required to render an object."""

    minimum: int
    """Minimum number of cells required to render."""
    maximum: int
    """Maximum number of cells required to render."""
```

### Comment Style
- Comments should explain "why", not "what".
- Keep comments concise and relevant.
```python
# Prevent potential infinite loop
rich_visited_set: Set[type] = set()

# Detect object which claim to have all the attributes
if hasattr(renderable, _GIBBERISH):
    return repr(renderable)
```

### TODO Comments
- Use `# TODO:` prefix for future work items.
```python
# TODO: This is a little inefficient, it is only used by full justify
```

### Avoid Obvious Comments
- Don't comment obvious code.
```python
# Bad - states the obvious
if offset < 0:
    offset = len(self) + offset  # Convert negative to positive

# Good - no comment needed, code is self-explanatory
if offset < 0:
    offset = len(self) + offset
```

---

## Error Handling

### Custom Exception Hierarchy
- Define custom exceptions that inherit from a base exception.
- Keep exception classes minimal with just docstrings.
```python
class ConsoleError(Exception):
    """An error in console operation."""


class StyleError(Exception):
    """An error in styles."""


class StyleSyntaxError(ConsoleError):
    """Style was badly formatted."""


class NotRenderableError(ConsoleError):
    """Object is not renderable."""
```

### Raising Exceptions
- Use descriptive error messages with context.
- Use `from None` to suppress chained exceptions when appropriate.
```python
raise errors.StyleSyntaxError(
    f"unable to parse {word!r} as color; {error}"
) from None
```

### Input Validation
- Validate inputs early in constructors and raise `ValueError` for invalid values.
- Use f-strings with `!r` to show the actual invalid value.
```python
def __init__(self, align: AlignMethod = "left") -> None:
    if align not in ("left", "center", "right"):
        raise ValueError(
            f'invalid value for align, expected "left", "center", or "right" (not {align!r})'
        )
```

### Exception Handling
- Be specific about which exceptions to catch.
- Handle exceptions at appropriate levels.
```python
try:
    Color.parse(word)
except ColorParseError as error:
    raise errors.StyleSyntaxError(
        f"unable to parse {word!r} as background color; {error}"
    ) from None
```

### Assertions
- Use assertions for internal invariants.
- Include helpful messages for debugging.
```python
assert len(character) == 1, "Character must be a string of length 1"
assert separator, "separator must not be empty"
```

---

## Performance Patterns

### Local Variable Caching
- Cache frequently-accessed attributes in local variables within loops.
- Cache class references to avoid repeated lookups.
```python
def render(self, console: "Console", end: str = "") -> Iterable["Segment"]:
    _Segment = Segment  # Cache class reference
    text = self.plain
    if not self._spans:
        yield Segment(text)
        if end:
            yield _Segment(end)
        return
```

### Method Caching
- Cache bound methods in local variables for hot paths.
```python
append = output.append  # Cache append method
for item in items:
    append(item)
```

### lru_cache Decorator
- Use `@lru_cache` for expensive pure functions.
- Specify `maxsize` based on expected usage patterns.
```python
from functools import lru_cache

@classmethod
@lru_cache(maxsize=4096)
def parse(cls, style_definition: str) -> "Style":
    """Parse a style definition."""
    ...

@lru_cache(maxsize=1024)
def get_html_style(self, theme: Optional[TerminalTheme] = None) -> str:
    """Get a CSS style rule."""
    ...
```

### Functional Tools
- Use `operator` module functions for simple operations.
```python
from operator import attrgetter, itemgetter

_hash_getter = attrgetter(
    "_color", "_bgcolor", "_attributes", "_set_attributes", "_link", "_meta"
)

max(measurements, key=itemgetter(0))
```

### Avoid Repeated String Concatenation
- Build strings with lists and `"".join()`.
```python
parts: List[str] = []
append = parts.append
for item in items:
    append(str(item))
return "".join(parts)
```

### Generator Expressions
- Prefer generator expressions over list comprehensions when iteration is one-time.
```python
# Good - generator expression for one-time iteration
return sum(segment.cell_length for segment in line)

# Use list comprehension when you need to iterate multiple times
measurements = [get_measurement(console, options, r) for r in renderables]
```

---

## Testing Guidelines

### Test File Organization
- Tests are in the `tests/` directory.
- Test files follow the pattern `test_<module_name>.py`.
- Each test file corresponds to a module in `rich/`.

### Test Function Naming
- Use descriptive names prefixed with `test_`.
- Name should describe what is being tested.
```python
def test_str():
    ...

def test_parse():
    ...

def test_style_stack():
    ...
```

### Test Structure
- Arrange-Act-Assert pattern.
- Keep tests focused on one behavior.
```python
def test_eq():
    assert Style(bold=True, color="red") == Style(bold=True, color="red")
    assert Style(bold=True, color="red") != Style(bold=True, color="green")
    assert Style().__eq__("foo") == NotImplemented
```

### pytest.raises for Exception Testing
```python
def test_parse():
    with pytest.raises(errors.StyleSyntaxError):
        Style.parse("on")
    with pytest.raises(errors.StyleSyntaxError):
        Style.parse("on nothing")
```

### Parameterized Tests
- Use `@pytest.mark.parametrize` for testing multiple inputs.
```python
@pytest.mark.parametrize(
    "is_windows,expected_size",
    [
        (True, (80, 25)),
        (False, (133, 24)),
    ],
)
def test_terminal_size(is_windows: bool, expected_size: Tuple[int, int]) -> None:
    ...
```

### Platform-Specific Tests
- Use `@pytest.mark.skipif` for platform-specific tests.
```python
@pytest.mark.skipif(sys.platform == "win32", reason="does not run on windows")
def test_16color_terminal() -> None:
    console = Console(
        force_terminal=True, _environ={"TERM": "xterm-16color"}, legacy_windows=False
    )
    assert console.color_system == "standard"
```

### Test Coverage
- Aim for high test coverage.
- Use `# pragma: no cover` for code that shouldn't be counted in coverage.
- Coverage report is generated with `make test`.

### Running Tests
```shell
# Run all tests with coverage
make test

# Run specific test file
pytest tests/test_style.py -v

# Run specific test function
pytest tests/test_style.py::test_parse -v
```

---

## Module Organization

### Module Demo Pattern
- Include `if __name__ == "__main__":` block with `# pragma: no cover`.
- Use this for demonstrating module functionality.
```python
if __name__ == "__main__":  # pragma: no cover
    from rich.console import Console

    console = Console()
    console.print(Style(color="red").render("Hello, World!"))
```

### __all__ Export List
- Define `__all__` in modules that are part of the public API.
```python
__all__ = ["get_console", "reconfigure", "print", "inspect", "print_json"]
```

### Module-Level Singletons
- Define singleton instances at module level for common cases.
```python
NULL_STYLE = Style()
_null_highlighter = NullHighlighter()
```

### Sentinel Objects
- Use sentinel classes to distinguish "no value" from `None`.
- This is useful when `None` is a valid input value.
```python
class NoChange:
    pass

NO_CHANGE = NoChange()

def update(self, value: Union[str, NoChange] = NO_CHANGE) -> None:
    if not isinstance(value, NoChange):
        self._value = value
```

### Internal Utility Modules
- Prefix with underscore: `_loop.py`, `_pick.py`, etc.
- Keep focused on single responsibility.
- Minimal dependencies on other Rich modules.

### Lazy Imports
- Use lazy imports to avoid circular dependencies.
- Import inside functions when needed only at runtime.
```python
def get_console() -> "Console":
    global _console
    if _console is None:
        from .console import Console
        _console = Console()
    return _console
```

---

## Deprecation Patterns

### Documenting Deprecation
- Use `Warn:` section in docstrings to note deprecated classes/functions.
```python
class VerticalCenter(JupyterMixin):
    """Vertically aligns a renderable.

    Warn:
        This class is deprecated and may be removed in a future version. Use Align class with
        `vertical="middle"`.
    """
```

### Runtime Deprecation Warnings
- Use `warnings.warn()` with `DeprecationWarning` for deprecated APIs.
```python
import warnings

def deprecated_function():
    warnings.warn(
        "deprecated_function is deprecated, use new_function instead",
        DeprecationWarning,
        stacklevel=2,
    )
```

---

## Open Questions

These are areas where the conventions could be clarified or where decisions
might need to be made:

1. **Python 3.10+ Union Syntax**: Should Rich consider using `X | Y` syntax in
   type annotations when dropping Python 3.9 support in the future?

2. **Absolute vs Relative Imports**: Rich uses relative imports consistently.
   Should this be documented as a hard requirement?

3. **slots vs dataclass**: When should `__slots__` be preferred over `@dataclass`?
   Currently, high-frequency objects like `Style`, `Text` use slots, while
   configuration objects like `Column` use dataclass.

4. **Protocol Classes Usage**: Rich uses both `Protocol` classes (like
   `ConsoleRenderable`) and duck typing. When should each approach be preferred?

5. **Runtime Deprecation Warnings**: Should deprecated APIs emit `DeprecationWarning`
   at runtime, or just document the deprecation?

6. **Async Support**: Are there plans for async rendering? If so, what patterns
   should be established?

7. **Type Stub Files**: Should Rich provide separate `.pyi` stub files, or
   continue with inline type annotations?

8. **Minimum Python Version Updates**: When Rich drops support for older Python
   versions, what modernization should be done (e.g., `|` union syntax,
   builtin generics)?

9. **Future Annotations Consistency**: Some modules use `from __future__ import
   annotations` and others don't. Should this be standardized?

10. **Exception Granularity**: When should a new custom exception be created vs.
    using an existing one?

---

*This document should be updated as the codebase evolves and new patterns emerge.*
