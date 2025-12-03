# Rich Rejected PR Review Analysis

This document analyzes 376 rejected PRs from the Rich repository to identify
patterns, common issues, and extract actionable coding guidelines.

**Data source**: Maintainer comments from closed/rejected PRs
**Total rejected PRs**: 376
**PRs with feedback**: 202
**Total maintainer comments analyzed**: 293

---

## Table of Contents

1. [Rejected PRs with Extracted Rules](#rejected-prs-with-extracted-rules)
2. [Summary of Rules by Category](#summary-of-rules-by-category)

---

## Rejected PRs with Extracted Rules

### PR #3798: fixing with none rich colored text

**Comment:**
> Let Rich generate escape sequences. But don't try and use them as input. If
> you have escape sequences you want to preserve, wrap them in
> `Text.from_ansi(my_string)`.

**Rule:** Use `Text.from_ansi()` to preserve existing ANSI escape sequences;
don't mix raw escape sequences with Rich's markup system.

---

### PR #3759: Add option for custom animation frames when creating a Spinner

**Comment:**
> I'm afraid I'm not accepting new features, which could be implemented outside
> of the core library.

**Rule:** New features that can be implemented externally should not be added to
the core library. Rich is in maintenance mode.

---

### PR #3756: Fix: spacing on arc spinner

**Comment:**
> That makes it looks worse for me. Terminals don't always agree with wcswidth.
> Getting it to work nicely everywhere is likely impossible I'm afraid.

**Rule:** Character width calculations vary across terminals. Fixes that improve
one terminal may break another. Cross-platform testing is essential.

---

### PR #3755: Private random generator for style link

**Comment:**
> I will need to understand what problem this solves for you. I suspect you will
> be better off using a random generator in your code, than modifying libraries.

**Rule:** PRs must clearly explain the problem being solved. Solutions should be
in application code when possible, not library modifications.

---

### PR #3710: Feature/indeterminate progress with elapsed

**Comment:**
> There is way too much going on in this PR. There seems to be code unrelated to
> progress bars. At this point, I can't accept new features.

**Rule:** PRs must be focused on a single concern. Remove unrelated changes.
Keep PRs small and reviewable.

---

### PR #3688: Fix/console buffering

**Comment:**
> Nothing needs fixing here. Please, no LLM produced code, unless you fully
> understand what problem you are solving.

**Rule:** Don't submit AI/LLM-generated code unless you fully understand the
problem and solution. Verify that the problem actually exists before proposing
fixes.

---

### PR #3686: Fix table alignment with Unicode characters

**Comment:**
> Rich already handles double cell characters. Some emoji are never going to
> work because terminals render them at different widths, no matter what the
> unicodedata says. If you are using an LLM to write this code, you should know
> that they tend to produce garbage.

**Rule:** Understand existing functionality before proposing changes. Some emoji
width issues are terminal bugs, not Rich bugs. LLM code without understanding is
rejected.

---

### PR #3444: Line 660 in /rich/progress.py (small input validation bug)

**Comment:**
> Why would `bar_width` receive a string? It is typed as an optional int.

**Rule:** Don't add defensive type checks when types are already annotated.
Trust the type system.

---

### PR #3426: Add support for continuation indent and optimize divide_line

**Comment:**
> Sorry, I don't think I can accept this as a feature, unless I was sure there
> was enough support for it. Optimizations / refactor would be fine.

**Rule:** New features need demonstrated demand. Pure refactoring and
optimization PRs are welcome. Link to the "cake or puppies" blog post.

---

### PR #3418: Handle dataclasses with uninitialized fields in pretty printing

**Comment:**
> This probably isn't the fix. It can give the false impression that the
> attribute exists. Better to omit it entirely IMO.

**Rule:** When handling missing data, prefer omission over placeholder values
that could mislead users.

---

### PR #3394: Make it possible to override the representation function

**Comment:**
> Monkey patching isn't good API, but if you are comfortable with it, you could
> just do this... That said, I think the best approach right now would be to
> write a method that accepts your objects and returns a proxy with a
> `__rich_repr__`.

**Rule:** Monkey patching is not good API design. Use protocols like
`__rich_repr__` for customization instead.

---

### PR #3389: 2286 pep657 error code ranges in traceback

**Comment:**
> Great in principle. I think we can improve on the styles a bit. And some of
> the information present in the ascii version doesn't seem to be styled. This
> will also need to handle older Pythons.

**Rule:** New features must support all supported Python versions (3.8+).
Styling should be consistent across output modes.

---

### PR #3336: Remove margin from text lines in Jupyter HTML template

**Comment:**
> Every change to Jupyter support causes someone else to complain. I've
> practically given up on Jupyter formatting issues.

**Rule:** Jupyter changes require extensive cross-platform testing. The Jupyter
ecosystem is fragmented; fixes often break other platforms.

---

### PR #3313: Update markdown.py

**Comment:**
> So it is empty cells that is the issue? Could have used that in description!

**Rule:** PR descriptions must clearly explain the problem. Don't make
maintainers guess what issue is being solved.

---

### PR #3311: live: add "tail" vertical_overflow method

**Comment:**
> I'm afraid I don't think this is a good idea. Consider Textual for such
> functionality.

**Rule:** Complex UI features should use Textual, not Rich. Rich is for
rendering, not interactive UI.

---

### PR #3300: Bisect all the things!

**Comment:**
> Just discovered that bisect key was added in 3.10. However, I don't think we
> will need it. It will work with the tuples--no need to separate them.

**Rule:** Don't use features added after Python 3.8 (the minimum supported
version). Prefer simpler approaches that work with tuples/existing patterns.

---

### PR #3300: Bisect all the things! (continued)

**Comment:**
> Could we keep the pairs and use `key=itemgetter(0)` with bisect?

**Rule:** Use `operator.itemgetter` for key functions in sorting/searching.

---

### PR #3288: added terminal themes: solarized light/dark

**Comment:**
> I think these are best maintained in an independent package. Feel free to
> publish rich-terminal-themes.

**Rule:** Themes and extensions should be published as separate packages, not
added to core.

---

### PR #3270: use larger width/height defaults for non-interactive consoles

**Comment:**
> You can set the env var COLUMNS to your desired width when running
> non-interactively. This is a bad idea. You wouldn't want a rule or table to be
> 32767 characters long.

**Rule:** Use standard environment variables (COLUMNS) for configuration. Don't
add library-specific defaults that could cause extreme behavior.

---

### PR #3247: Added customization to the illegal_choice_message

**Comment:**
> The recommended way to do this would be to subclass your Prompt class.

**Rule:** Use subclassing for customization, not new parameters.

---

### PR #3241: Add new prompt classes for path inputs

**Comment:**
> Looks good, but it doesn't need to be part of the core library.

**Rule:** Specialized input handlers should be external packages, not core.

---

### PR #3236: Make the Console generic on the file type

**Comment:**
> I suspect this will cause typing errors for anyone who doesn't provide the
> generic type. And that would lead to a flood of issues. The solution is good,
> but for this reason I don't think I can merge.

**Rule:** Type changes must not break existing code. Consider the impact on
users who don't provide explicit type parameters.

---

### PR #3233: fix(BrokenPipeError)

**Comment:**
> It also seems disruptive for a library to explicitly exit the app. Maybe there
> should be a method on Console that is called when a broken pip is received.
> The default behavior could raise an exception, but the dev could implement
> different behavior if desired.

**Rule:** Libraries should not exit applications. Provide hooks for applications
to handle errors their own way.

---

### PR #3072: Modernize code

**Comment:**
> Most of these changes are fairly arbitrary. Some may be worthwhile, but some
> its not clear what the benefit of the change is. Without knowing the issue
> they are fixing, its just not worth merging new code. I'd suggest separating
> the cosmetic changes, from larger changes, and submitting them separately.

**Rule:** Each change must have a clear purpose. Separate cosmetic from
functional changes. Don't mix unrelated modifications.

---

### PR #3058: progress-bar-track

**Comment:**
> Examples should be fully-working, and use Python conventions (i.e.
> snake_case).

**Rule:** Examples must be complete and runnable. Use snake_case for Python
examples.

---

### PR #3041: Added emojis from Emoji Versions 13,0, 13.1, 14.0 and 15.0

**Comment:**
> Most terminals don't support or render these new emoji. I fear it would just
> disappoint most users.

**Rule:** Don't add features that won't work on most terminals. Terminal support
for new emoji is inconsistent.

---

### PR #3038: Make rich look better in legacy Windows Console

**Comment:**
> I can't accept this, because it changes the state of the terminal as a
> side-effect of importing. This may impact other users who weren't accepting
> it.

**Rule:** Never change global terminal state on import. Import side effects are
forbidden.

---

### PR #3025: Fix tag RegEx

**Comment:**
> You're not always going to be able to parse things perfectly. REPRs aren't a
> formal grammar. One thing to be aware of is performance. You might print a lot
> of text with Rich, and overly expensive regexes can be a problem.

**Rule:** Regex must be performant for high-volume text. Accept that some edge
cases are unfixable. Consider performance impact.

---

### PR #2996: Added `no_border` option to `console.print_exception`

**Comment:**
> There may be a need for a simpler output format, but no_border is too
> descriptive.

**Rule:** Parameter names should be general, not overly specific. Design for
future expansion.

---

### PR #2937: Tweaked SVG generation to be more readable

**Comment:**
> I'm afraid I can't justify a new dependency, or the breaking change, without a
> very good reason.

**Rule:** New dependencies require very strong justification. Breaking changes
need compelling reasons.

---

### PR #2905: Text.format() feature

**Comment:**
> Sorry, I'm afraid I can't merge this PR. I have a feeling the implementation
> is doing more work than it needs to be, and the overall code quality isn't
> quite high enough. The tests don't give me any confidence in the code. Tests
> should be treated almost like documentation, with a function for each aspect
> of the code being tested. It's not enough just to get full coverage.

**Rule:** Tests must be documentation-quality with one function per aspect.
Coverage alone is insufficient. Implementation should be minimal.

---

### PR #2905: Text.format() feature (continued)

**Comment:**
> The efficiency of this method could be improved. Readability could use some
> work. Better named variables etc. `str_index` says what it is, but doesn't say
> what it is used for. We have a no abbreviation and no single letter variables
> (with a few exceptions). A few well-placed comments would also help to follow
> the code.

**Rule:** Variable names must describe purpose, not just type. No abbreviations
or single-letter names. Add comments for non-obvious logic. Consider efficiency.

---

### PR #2811: Improve Traceback Frame Suppression

**Comment:**
> I'm not keen on the regex approach. The chances are too high that you end up
> excluding something accidentally. I'd consider callbacks or some other
> protocol to do this.

**Rule:** Avoid regex for filtering/matching. Prefer callbacks or protocols for
user-defined behavior.

---

### PR #2759: Let rich.pretty.install support ptpython

**Comment:**
> I don't think I can accept this. There needs to be a better mechanism that try
> all possible libraries until one works. Perhaps Rich is not the right place
> for this code.

**Rule:** Don't add try/except loops for library detection. Keep library-specific
code in those libraries.

---

### PR #2555: feat(_spinners): add more spinners

**Comment:**
> These are cool, but I suspect a number of these emojis will break on some
> terminals.

**Rule:** Emoji in spinners must work across all major terminals.

---

### PR #2554: Implement Text comparator methods

**Comment:**
> Text objects are mutable, which makes them unsuitable for use as dictionary
> keys. Without a clear bug to fix, this is a somewhat arbitrary change, more
> likely to break other projects.

**Rule:** Don't add comparison methods to mutable objects. Changes without clear
bugs risk breaking existing code.

---

### PR #2500: Add option to suppress provided types in tracebacks

**Comment:**
> It may be too difficult to satisfy everyone with an extra parameter. In that
> situation I usually prefer to expose hooks so that a user could implement
> their own particular requirements.

**Rule:** When needs vary widely, provide hooks rather than parameters.

---

### PR #2433: Prompt messages

**Comment:**
> Saves a line or two at the expense of supporting additional parameters. Don't
> think its worth it.

**Rule:** Don't add parameters just to save a line of code. The complexity isn't
worth it.

---

### PR #2414: Provide `exclude_locals` mechanism

**Comment:**
> This is trickier code than it might appear because Rich has to be extremely
> defensive. We can't risk the exception output generating an error of its own.

**Rule:** Exception handling code must be extremely defensive. Never let
exception rendering cause exceptions.

---

### PR #2405: Populate table using from_dict

**Comment:**
> Sorry, the interface for this `from_dict` doesn't gel well with the rest of
> the API. Good effort though. Perhaps you could make this a standalone project?

**Rule:** New APIs must be consistent with existing patterns. Inconsistent
interfaces should be separate packages.

---

### PR #2396: Add optional 'spacer' argument to Spinner

**Comment:**
> If your spacer argument makes it look a little better on your platform, at the
> expense of it breaking on another, then I wouldn't want to add it. If you can
> show me that it doesn't visually break on Mac, Windows, and Linux, I'll
> reconsider...

**Rule:** Visual changes must be tested on Mac, Windows, and Linux. Don't fix
one platform at the expense of others.

---

### PR #2364: logging render_message: Use message as-is if already Text

**Comment:**
> The hexdump is very cool, but I'm afraid this is an abuse of logging. Logging
> may be configured to go to multiple destinations. If you log a Text object it
> will break with anything other than RichHandler. Consider using `console.log`.

**Rule:** Logging handlers must output string-compatible data. Rich objects
should use `console.log`, not logging.

---

### PR #2314: Add `rqdm` and `rrange` utilities

**Comment:**
> Sorry, naming leaves a lot to be desired. Not sold on the functionality.
> Willing to be convinced.

**Rule:** Names must be clear and descriptive. Novel naming conventions are
rejected.

---

### PR #2270: Extract color-system-as-a-string to a type literal

**Comment:**
> I'm not convinced this is worthwhile. I'd suggest moving the burden back to
> Textual. Defining a literal there, but bear in mind it is for internal use.

**Rule:** Types for internal use don't need to be exported. Keep internal types
internal.

---

### PR #2232: Add level_width and level_format params to RichHandler

**Comment:**
> Sorry, the logging handler is too complex already. I can't accept new
> parameters. If you need this for a project I guess you wouldn't have much
> difficulty extending it in your own code.

**Rule:** Complex classes shouldn't gain more parameters. Extend via subclassing
instead.

---

### PR #2227: Return a Task object from add_task, not an id

**Comment:**
> Since this is a breaking change I couldn't accept it unless there was a very
> compelling reason to do so. Exposing the Task object is probably a bad idea
> because there are some internal attributes there, that a user might expect to
> modify and have a meaningful effect.

**Rule:** Breaking changes need compelling justification. Don't expose objects
with internal attributes that users might misuse.

---

### PR #2143: feat: waves spinner

**Comment:**
> Sorry, can't accept code which runs at import time. This may have to remain
> something you could add to your own project, if you want it. But probably
> doesn't belong in the core lib.

**Rule:** No code execution at import time. Initialization must be lazy.

---

### PR #2122: Allow default Console parameters via environment variables

**Comment:**
> I'm not keen on adding env vars that aren't already a standard. As a library,
> Rich shouldn't impose its own env vars on to applications. It should be the
> application author that decides how to handle env vars.

**Rule:** Libraries should not define custom environment variables. Use standard
env vars only (COLUMNS, LINES, NO_COLOR, etc.).

---

### PR #2084: fix: full static typing

**Comment:**
> There is little point in typechecking examples. They are for educational
> purposes, and I don't want to assume the viewer is familiar with type
> annotations. That includes `mypy:` comments which have the potential to
> confuse.

**Rule:** Examples should be beginner-friendly. Don't add type annotations or
mypy comments to examples.

---

### PR #2013: Add the `log_omit_repeated_times` option to Console

**Comment:**
> The constructor already has a large argument list. I'd prefer not to add to it
> without good reason.

**Rule:** Don't add parameters to already-complex constructors without strong
justification.

---

### PR #1985: Hy support during inspection

**Comment:**
> I'm afraid I couldn't advocate for adding code to inspect for specific object
> types. Especially something I am not familiar with.

**Rule:** Don't add support for specific third-party libraries. Keep Rich
agnostic.

---

### PR #1981: Pass Console parameters to logging.RichHandler

**Comment:**
> Sorry, not keen on this change. A union of a Console and a dict of Any is an
> invitation for typing errors.

**Rule:** Don't use Union types with very different types (object vs dict). Keep
parameter types focused.

---

### PR #1604: Many Miscellaneous Typing Fixes

**Comment:**
> I'm not willing to give up on the setter accepting a variety of types, but I
> don't want to move the burden to the caller.

**Rule:** Setters can accept flexible types for convenience, even if typing is
complex. Don't burden the user.

---

### PR #1551: Fix all reportPrivateImportUsage in pyright

**Comment:**
> I'm not overly keen on that workaround. The main sections are excluded from
> coverage, but I still like them to be typechecked. Could the individual lines
> be excluded in Pyright, rather than the entire block?

**Rule:** Prefer line-level type ignores over block-level exclusions.

---

### PR #1544: Correct typing of Iterables

**Comment:**
> I don't want to promise that the return type has a `__next__` method, all I
> want to promise is that I'm returning something that may be iterated over.
> This allows me to change the return type to some other iterable in the future.

**Rule:** Use `Iterable` over `Iterator` for return types to preserve
flexibility for future changes.

---

### PR #1382: Optimisation suggestions for Segment.divide()

**Comment:**
> The annotations are only evaluated at import time, so shouldn't have any
> impact. Good suggestions, but these micro-optimizations probably won't make a
> measurable impact. I'm looking for a more algorithmic improvement.

**Rule:** Micro-optimizations are generally not worthwhile. Focus on algorithmic
improvements for performance.

---

### PR #1290: GitHub Action to lint Python code

**Comment:**
> I already run black, mypy, and pytest. I suspect many of these linters will
> create pointless busy work. e.g. I use iSort in the editor, but frankly if an
> import is out of order, it doesn't matter if it gets checked in.

**Rule:** Don't add linting for trivial issues. Focus on meaningful code
quality, not pedantry.

---

### PR #1218: fix color output in spyder console

**Comment:**
> I can't accept a fudge for a specific IDE. Colorama is a headache for Rich as
> it makes global changes, whereas Rich can have different settings per-Console
> instance.

**Rule:** Don't add IDE-specific workarounds. Console instances should be
independent.

---

### PR #403: Allow level name of RichHandler to be customized

**Comment:**
> I prefer not to overload parameters like that, I find having an argument that
> may be two quite different types tends to make typing difficult and makes the
> code harder to reason about.

**Rule:** Don't overload parameters with multiple unrelated types. Keep
parameter types focused.

---

### PR #347: Make soft_wrap preference configurable per Console

**Comment:**
> If you print a third party renderable to a console with soft wrapping
> disabled, it may produce broken output. It doesn't really fix anything, just
> save a little typing, at the expense of potentially breaking things.

**Rule:** Configuration that could break third-party renderables is rejected.
Don't save typing at the expense of safety.

---

### PR #114: Removes unused imports

**Comment:**
> I've done the work in another branch, and also cleaned up the unused imports.
> Thanks for prompting me to do this, it was overdue.

**Note:** Sometimes PRs prompt maintainer to do the work differently. The
contribution is still valuable even if not merged directly.

---

### PR #107: enable emojis for rich logging handler

**Comment:**
> I'm not keen on enabling emoji code in logging. The emoji isn't going to be
> replaced in all handlers, and the function to do the replace is fast but not
> free in terms of cpu.

**Rule:** Logging handlers must be efficient. Don't add per-message processing
overhead that all logs would pay.

---

## Summary of Rules by Category

### Scope & Features

1. Rich is in maintenance mode - new features generally not accepted
2. Features that can be implemented externally should be separate packages
3. Use Textual for complex UI features
4. Themes, extensions, specialized handlers should be external packages

### Code Quality

1. No abbreviations in variable names
2. No single-letter variables (rare exceptions)
3. Variable names should describe purpose, not just type
4. Add comments for non-obvious logic
5. Consider efficiency in implementations
6. Tests should be documentation-quality, one function per aspect
7. Coverage alone is insufficient

### PR Quality

1. PRs must be focused on a single concern
2. Clearly explain the problem being solved
3. Separate cosmetic from functional changes
4. Don't mix unrelated modifications
5. Examples must be complete and runnable

### API Design

1. Use subclassing for customization, not new parameters
2. Provide hooks rather than parameters when needs vary
3. Don't overload parameters with multiple unrelated types
4. Don't expose objects with internal attributes users might misuse
5. Keep internal types internal
6. New APIs must be consistent with existing patterns

### Type Safety

1. Don't add defensive checks when types are already annotated
2. Type changes must not break existing users
3. Use `Iterable` over `Iterator` for return type flexibility
4. Prefer line-level type ignores over block-level exclusions

### Platform Compatibility

1. Must support Python 3.8+ (no features from later versions)
2. Visual changes must be tested on Mac, Windows, and Linux
3. Character width varies across terminals - cross-platform test
4. New emoji likely won't work on most terminals
5. Jupyter changes require extensive cross-platform testing

### Import & Initialization

1. Never change global terminal state on import
2. No code execution at import time
3. Libraries should not define custom environment variables
4. Use standard env vars only (COLUMNS, NO_COLOR, etc.)

### Error Handling

1. Libraries should not exit applications
2. Exception handling code must be extremely defensive
3. Never let exception rendering cause exceptions
4. Provide hooks for applications to handle errors

### Performance

1. Micro-optimizations are generally not worthwhile
2. Focus on algorithmic improvements
3. Regex must be performant for high-volume text
4. Logging handlers must be efficient

### LLM/AI Code

1. Don't submit AI-generated code without understanding
2. Verify problems exist before proposing fixes
3. Understand existing functionality before changing

### Breaking Changes

1. Breaking changes need very compelling justification
2. New dependencies require very strong justification
3. Changes without clear bugs risk breaking existing code

---

*This analysis was generated from maintainer feedback on rejected PRs to extract
patterns and guidelines for future contributions.*
