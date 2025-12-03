# Rich Coding Rules

These are the coding conventions and patterns for the Rich library that require
manual code review. Items handled automatically by the build system are listed
separately at the end.

---

## Table of Contents

1. [Python Version Compatibility](#python-version-compatibility)
2. [Naming Conventions](#naming-conventions)
3. [Type Annotation Patterns](#type-annotation-patterns)
4. [Docstring and Comment Guidelines](#docstring-and-comment-guidelines)
5. [Deprecation Patterns](#deprecation-patterns)
6. [Imports and Module Organization](#imports-and-module-organization)
7. [Error Handling](#error-handling)
8. [Class Design Patterns](#class-design-patterns)
9. [Performance Patterns](#performance-patterns)
10. [Rich Protocol Patterns](#rich-protocol-patterns)
11. [Automatically Enforced (Reference Only)](#automatically-enforced-reference-only)
12. [Open Questions](#open-questions)

---

## Python Version Compatibility

### Supported Versions
- Rich supports Python 3.8 through 3.14.
- Code must be compatible with Python 3.8+ syntax and features.

### Runtime Type Syntax Restrictions
- Do NOT use Python 3.9+ builtin generics at runtime: `list[str]`, `dict[str, int]`
- Do NOT use Python 3.10+ union syntax at runtime: `str | None`
- These are only allowed inside `TYPE_CHECKING` blocks or as string annotations.

```python
# Correct for Python 3.8+
from typing import List, Dict, Optional, Union

def example(items: List[str], mapping: Dict[str, int]) -> Optional[str]:
    ...
```

### Future Annotations
- Some modules use `from __future__ import annotations` for PEP 563.
- When used, forward references don't need quotes.
```python
from __future__ import annotations

def method(self) -> Style:  # No quotes needed
    ...
```

---

## Naming Conventions

### Variables and Functions
- Use `snake_case`. Avoid abbreviations.

### Classes
- Use `PascalCase`.

### Constants
- Use `UPPER_SNAKE_CASE` for module-level constants.
```python
WINDOWS = sys.platform == "win32"
NULL_STYLE = Style()
DEFAULT_JUSTIFY: "JustifyMethod" = "default"
```

### Private Members
- Prefix private attributes and methods with single underscore.
```python
self._spans: List[Span] = []

def _trim_spans(self) -> None: ...
```

### Internal Modules
- Internal utility modules start with underscore: `_loop.py`, `_pick.py`

### Type Aliases
- Type aliases use `PascalCase`.
```python
StyleType = Union[str, "Style"]
RenderResult = Iterable[Union[RenderableType, Segment]]
```

---

## Type Annotation Patterns

### Type Aliases
- Define type aliases for commonly used complex types at module level.
```python
StyleType = Union[str, "Style"]
TextType = Union[str, "Text"]
RenderableType = Union[ConsoleRenderable, RichCast, str]
```

### NewType for Type-Safe Aliases
- Use `NewType` for IDs and values that shouldn't be mixed.
```python
from typing import NewType

TaskID = NewType("TaskID", int)
```

### ClassVar for Class-Level Attributes
```python
from typing import ClassVar

class MarkdownElement:
    new_line: ClassVar[bool] = True
```

### Overload for Multiple Signatures
```python
from typing import overload, Union, List

@overload
def __getitem__(self, index: int) -> "Text": ...

@overload
def __getitem__(self, index: slice) -> List["Text"]: ...

def __getitem__(self, index: Union[slice, int]) -> Union["Text", List["Text"]]:
    return self._lines[index]
```

### Self Type
- Use `Self` from `typing_extensions` for methods returning self.
```python
if TYPE_CHECKING:
    from typing_extensions import Self  # pragma: no cover

def __enter__(self) -> "Self":
    self.start()
    return self
```

### Literal Types for Constrained Strings
```python
from typing import Literal

JustifyMethod = Literal["default", "left", "center", "right", "full"]
AlignMethod = Literal["left", "center", "right"]
```

### Protocol for Structural Typing
```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class ConsoleRenderable(Protocol):
    def __rich_console__(
        self, console: "Console", options: "ConsoleOptions"
    ) -> "RenderResult": ...
```

### mypy Ignore Comments
- Always include the specific error code.
```python
_console.__dict__ = new_console.__dict__  # type: ignore[assignment]
```

---

## Docstring and Comment Guidelines

### Docstring Format
- Google-style with Args/Returns sections.
- Opening triple quotes on their own line for multi-line docstrings.
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
- Document `__init__` parameters in the class docstring.
- Include usage examples when helpful.
```python
class Panel(JupyterMixin):
    """A console renderable that draws a border around its contents.

    Example:
        >>> console.print(Panel("Hello, World!"))

    Args:
        renderable (RenderableType): A console renderable object.
        box (Box): A Box instance that defines the look of the border.
        title (Optional[TextType], optional): Optional title. Defaults to None.
    """
```

### Field Docstrings
- Use inline docstrings after NamedTuple/dataclass fields.
```python
class Measurement(NamedTuple):
    minimum: int
    """Minimum number of cells required to render."""
    maximum: int
    """Maximum number of cells required to render."""
```

### Comment Guidelines
- Comments should explain "why", not "what".
- Avoid obvious comments.
```python
# Good - explains why
# Prevent potential infinite loop
rich_visited_set: Set[type] = set()

# Bad - states the obvious
if offset < 0:
    offset = len(self) + offset  # Convert negative to positive
```

### TODO Comments
- Use `# TODO:` prefix.

---

## Deprecation Patterns

### Docstring Deprecation Notice
- Use `Warn:` section.
```python
class VerticalCenter(JupyterMixin):
    """Vertically aligns a renderable.

    Warn:
        This class is deprecated and may be removed in a future version.
        Use Align class with `vertical="middle"`.
    """
```

### Runtime Deprecation Warnings
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

## Imports and Module Organization

### Relative Imports
- Use relative imports for internal module references.
```python
from .style import Style, StyleType
from .text import Text, TextType
from . import errors
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

### Lazy Imports
- Use to avoid circular dependencies.
```python
def get_console() -> "Console":
    global _console
    if _console is None:
        from .console import Console
        _console = Console()
    return _console
```

### Noqa for Re-exports
- Use `# noqa: F401` for intentional re-exports.
```python
from ._extension import load_ipython_extension  # noqa: F401
```

### __all__ Export List
- Define for public API modules.
```python
__all__ = ["get_console", "reconfigure", "print", "inspect", "print_json"]
```

### Internal Utility Modules
- Prefix with underscore: `_loop.py`, `_pick.py`
- Keep focused on single responsibility.
- Minimal dependencies on other Rich modules.

### Singletons
- Define at module level for common cases.
```python
NULL_STYLE = Style()
_null_highlighter = NullHighlighter()
```

### Sentinel Objects
- Use to distinguish "no value" from `None`.
```python
class NoChange:
    pass

NO_CHANGE = NoChange()

def update(self, value: Union[str, NoChange] = NO_CHANGE) -> None:
    if not isinstance(value, NoChange):
        self._value = value
```

### Module Demo Pattern
- Include `if __name__ == "__main__":` with `# pragma: no cover`.
```python
if __name__ == "__main__":  # pragma: no cover
    from rich.console import Console
    console = Console()
    ...
```

---

## Error Handling

### Custom Exception Hierarchy
- Define custom exceptions inheriting from base exceptions.
- Keep exception classes minimal.
```python
class ConsoleError(Exception):
    """An error in console operation."""

class StyleSyntaxError(ConsoleError):
    """Style was badly formatted."""
```

### Exception Messages
- Use descriptive messages with context.
- Use `from None` to suppress chained exceptions when appropriate.
```python
raise errors.StyleSyntaxError(
    f"unable to parse {word!r} as color; {error}"
) from None
```

### Input Validation
- Validate early in constructors.
- Use `!r` to show the actual invalid value.
```python
if align not in ("left", "center", "right"):
    raise ValueError(
        f'invalid value for align, expected "left", "center", or "right" (not {align!r})'
    )
```

### Assertions
- Use for internal invariants with helpful messages.
```python
assert len(character) == 1, "Character must be a string of length 1"
assert separator, "separator must not be empty"
```

---

## Class Design Patterns

### __slots__ for High-Frequency Objects
- Use `__slots__` for classes with many instances (e.g., `Style`, `Segment`, `Span`).
```python
class Style:
    __slots__ = [
        "_color",
        "_bgcolor",
        "_attributes",
        ...
    ]
```

### NamedTuple for Immutable Data
- Add inline docstrings for fields.
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
- Use for classes with many defaulted fields.
```python
@dataclass
class Column:
    header: "RenderableType" = ""
    """Renderable for the header."""

    width: Optional[int] = None
    """Width of the column."""
```

### Factory Methods
- Use `@classmethod` for alternative constructors.
- Name them descriptively: `from_*`, `parse`, `null`, `create`.
```python
@classmethod
def null(cls) -> "Style":
    """Create a 'null' style, equivalent to Style(), but more performant."""
    return NULL_STYLE

@classmethod
def from_markup(cls, text: str, ...) -> "Text": ...

@classmethod
def parse(cls, style_definition: str) -> "Style": ...
```

### Copy Methods
- Implement `copy()` for mutable classes that need cloning.

### Abstract Base Classes
```python
from abc import ABC, abstractmethod

class Highlighter(ABC):
    @abstractmethod
    def highlight(self, text: Text) -> None: ...
```

### Context Manager Protocol
```python
from types import TracebackType

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
- Use `RLock` for reentrant locking.
- Use `Event` for thread signaling.
- Create daemon threads (`daemon=True`).
```python
class _RefreshThread(Thread):
    def __init__(self, live: "Live", refresh_per_second: float) -> None:
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

## Performance Patterns

### Local Variable Caching
- Cache frequently-accessed attributes and class references in loops.
```python
def render(self, console: "Console", end: str = "") -> Iterable["Segment"]:
    _Segment = Segment  # Cache class reference
    text = self.plain
    ...
```

### Method Caching
- Cache bound methods in hot paths.
```python
append = output.append
for item in items:
    append(item)
```

### lru_cache
- Use for expensive pure functions.
- Specify appropriate `maxsize`.
```python
@classmethod
@lru_cache(maxsize=4096)
def parse(cls, style_definition: str) -> "Style": ...
```

### Functional Tools
- Use `operator` module functions.
```python
from operator import attrgetter, itemgetter

_hash_getter = attrgetter("_color", "_bgcolor", "_attributes")
max(measurements, key=itemgetter(0))
```

### String Building
- Build strings with lists and `"".join()`.
```python
parts: List[str] = []
append = parts.append
for item in items:
    append(str(item))
return "".join(parts)
```

### Generator vs List Comprehension
- Use generators for one-time iteration.
- Use list comprehensions when iterating multiple times.

---

## Rich Protocol Patterns

### __rich_console__ Protocol
- Yields `Segment` objects or other renderables.
```python
def __rich_console__(
    self, console: "Console", options: "ConsoleOptions"
) -> "RenderResult":
    yield Segment(self.text)
    yield Segment("\n")
```

### __rich_measure__ Protocol
- Returns a `Measurement` with minimum and maximum widths.
```python
def __rich_measure__(
    self, console: "Console", options: "ConsoleOptions"
) -> "Measurement":
    return Measurement(min_width, max_width)
```

### __rich__ Protocol
- Returns a Rich renderable object.
```python
def __rich__(self) -> "Text":
    return Text(str(self))
```

### __rich_repr__ Protocol
- Works with the `@rich_repr` decorator.
```python
from rich.repr import rich_repr, Result

@rich_repr
class Style:
    def __rich_repr__(self) -> Result:
        yield "color", self.color, None
        yield "bgcolor", self.bgcolor, None
```

### JupyterMixin
- Inherit to enable Jupyter notebook rendering.
```python
class Panel(JupyterMixin):
    ...
```

---

## Automatically Enforced (Reference Only)

The following are handled by the build system, formatters, or CI and do NOT
require manual code review:

### Formatting (Black)
- Line length (88 characters)
- Trailing commas
- Blank lines between definitions
- String quote style
- Indentation

### Import Ordering (isort)
- Standard library → third-party → local imports

### Type Checking (mypy)
- Type annotation correctness
- Missing return types
- Incompatible types

### Test Coverage (pytest-cov)
- Code coverage metrics

### Spelling (codespell)
- Common spelling errors in comments/docstrings

---

## Open Questions

Areas where conventions need clarification:

1. **slots vs dataclass**: When should `__slots__` be preferred? Currently,
   high-frequency objects use slots; configuration objects use dataclass.

2. **Protocol vs Duck Typing**: When should formal `Protocol` classes be used
   vs. relying on duck typing?

3. **Runtime Deprecation Warnings**: Should deprecated APIs emit
   `DeprecationWarning` at runtime, or just document the deprecation?

4. **Future Annotations Consistency**: Some modules use `from __future__ import
   annotations` and others don't. Should this be standardized?

5. **Exception Granularity**: When should a new custom exception be created vs.
   using an existing one?

---

*This document should be updated as the codebase evolves and new patterns emerge.*
