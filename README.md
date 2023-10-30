# Code review and Code style

- [Code review and Code style](#code-review-and-code-style)
  - [Aim of this document](#aim-of-this-document)
    - [Style guidelines in Docusaurus](#style-guidelines-in-docusaurus)
  - [Code-style: Python and Python repositories](#code-style-python-and-python-repositories)
    - [Comments and docstrings](#comments-and-docstrings)
      - [A note about markdown](#a-note-about-markdown)
    - [In-line comments](#in-line-comments)
      - [In-line TODOs](#in-line-todos)
    - [Logging](#logging)
      - [Lower-case messages](#lower-case-messages)
      - [Error-artefacts](#error-artefacts)
      - [Communicating with users](#communicating-with-users)
      - [Secrets](#secrets)
    - [Comments and communication](#comments-and-communication)
      - [Tips for communication outside of code](#tips-for-communication-outside-of-code)
    - [Variables and variable naming](#variables-and-variable-naming)
    - [Type-hinting](#type-hinting)
    - [Follow the happy path](#follow-the-happy-path)
    - [EAFP (Easy to ask forgiveness than permission)](#eafp-easy-to-ask-forgiveness-than-permission)
    - [Code-commits](#code-commits)
    - [Configuring run-times and "failing fast"](#configuring-run-times-and-failing-fast)
    - [Concatenation - strings and file-paths](#concatenation---strings-and-file-paths)
      - [File-paths](#file-paths)
      - [Strings](#strings)
    - [Tests](#tests)
      - [Tests are first-class citizens](#tests-are-first-class-citizens)
      - [Minimize the use of fixtures](#minimize-the-use-of-fixtures)

<!-- https://luciopaiva.com/markdown-toc/ -->

## Aim of this document

The goal of this document is to bring a cycle of pull-requests -- code-review --
merging into the Arkly API code-base. And begin to add this to our other
production repositories.

Another goal is to codify this for our other languages, be that JavaScript, Golang, Mendix, or anything else.

First we'll iterate on the thinking here, try it out, and when input is
incorporated, and it makes sense, we'll try and standardize it.

Iteration isn't described explicitly below, but it is important that we will
strike a balance between pragmatism and perfection, and learning from each
cycle we do this.

### Style guidelines in Docusaurus

The assumption is that we will build on these guidelines and add them to a
more general set of developer documentation. An example layout in Docusaurus
may look as follows.

```text
└── coding-style
    ├── mendix
    │   └── CODE_GUIDELINES.md
    ├── python
    │   └── CODE_GUIDELINES.md
    └── react
        └── CODE_GUIDELINES.md
```

## Code-style: Python and Python repositories

### Comments and docstrings

- Maximum 72 characters line-length. This is an old teletype standard but it
works well for seeing information quickly when scrolling through the code.
- Docstrings should have a title unless they are only sentence long. They can
encapsulate as much information as is needed to convey what they do.
- Docstrings will not list parameter or return types explicitly as they evolve
with the code and are more likely to change as function signatures and return
values change, as such, Python's type-hints should be preferred.

> *Example docstrings:*
>
> ```python
> def my_longer_function_docstring(sentence: str, count: int) -> str:
>   """Concatenates a string with an integer because at some point I am
>   going to log it to stderr.
>   """
> ```

> ```python
> def my_shorter_function_docstring(accumulative_value: int, new_value: int) -> int:
>   """Adds two integers together."""
> ```

#### A note about markdown

Markdown artifacts support the code-base. Where we are writing literal markdown
README.md for example, or other guides, we should also follow helpful standards.

- Maximum 79 characters line-length, Diffs reveal more information with greater
number of lines, and it also means less information appearing in a diff that
hasn't changed.
- Reference style hyperlinks when links or line-lengths cannot be split.

> *Example reference link:*

> ```markdown
> More information can be found [here][more-info-1]
>
> [more-info-1]: http;//example.com
> ```

### In-line comments

Code should be self-describing. That is, through a number of different
approaches described below, and clean-coding principles, comments should be
minimal. That being said in-line comments may still communicate something
important to the reader that isn't already in a docstring, or described by the
variable name. Communication is discussed below, and comments are an important
part of that.

Tips for determining if a comment is still required:

- The comment isn't a clue that a separate smaller and specific function should
be created for what it is describing, and would therefore benefit from the
function signature and docstring.
- The comment isn't repeating something that should be explicit in variable
naming, e.g. describing what the variable means, instead of the reader being
able to infer that from the variable name.
- The comment isn't redundant if the lines immediately following it are not
explicit enough in what they are doing, e.g. we may use a third-party library
that is overly abstracted and difficult to understand without commenting what we
do.
- The comment isn't dead-code, e.g. old code being preserved, or trial code that
isn't used.
- The comment isn't taking the place of logging.
- The comment isn't  a TODO, below.

Ultimately, comments are a communication artifact, and that means a lot as we
communicate with our future selves. They won't be viewed badly.

#### In-line TODOs

`TODOs` in production code should be avoided at all costs. A `TODO` should
prompt the creation of a new issue in the code repository, or in a Kanban board
such as Trello. A `TODO`worth documenting should be explicit to the project
team, i.e. not just those looking at code, and planned alongside the rest of the
work on the code-base. A TODO may also be surfaced in a pull-request, this way
it can be discussed as part of the same body of work.

### Logging

Logging should be configured consistently across a product family's
code-base unless there is a specific need for another specific style of
logging for a specific product.

A suggested configuration for Python logging is as follows:

```python
import logging
import time

# Set up consistent logging.
logging.basicConfig(
    format="%(asctime)-15s %(levelname)s :: %(filename)s:%(lineno)s:%(funcName)s() :: %(message)s",  # noqa: E501
    datefmt="%Y-%m-%d %H:%M:%S",
    level="INFO",
    handlers=[
        logging.StreamHandler(),
    ],
)

# Format logging using UTC time.
logging.Formatter.converter = time.gmtime


logger = logging.getLogger(__name__)
```

This logger does the following:

- Provides a log time in UTC. This is consistent wherever the application is
run.
- Provides detail on the log-level, e.g. INFO, WARNING, ERROR.
- Shares with the user the name of the source-code file where the log message is
output.
- The line number in the source code.
- The function name where the log message is output.
- The user controlled message.
- Uses double colons as delineation if the message needs to be filtered in
other ways using standard Linux tooling.

Example output looks as follows:

```log
1970-01-01 00:01:55 ERROR :: log_demo.py:24:sum() :: summing '1' + '"1"' failed: unsupported operand type(s) for +: 'int' and 'str'
```

For more information on Python's logging see:

- Logging [HOWTO][python-logging-howto].
- Logging [module documentation][python-logging-module].


[python-logging-howto]: https://docs.python.org/3/howto/logging.html
[python-logging-module]: https://docs.python.org/3/library/logging.html

#### Lower-case messages

Log messages should start lower-case. If they start to run into more than one
sentence, consider if the message is too complex, or if the function is
modular enough. Additional separators, e.g. ` :: ` in the case above can be
added for more complex lines, e.g. greater summary detail.

#### Error-artefacts

If the error artefact is available, then make sure to include it in the log
message, e.g.

```python
def sum(a: int, b: str):
   try:
      a + b
   except TypeError as err:
      logger.error("summing '%s' + '\"%s\"' failed: %s", a, b, err)
```

#### Communicating with users

Always keep in mind that logging is another communication with your users.

Your users might include support colleagues, devops, external users, external
support, and yourself. This means that logging should be professional, and
meaningful.

Ned Batchelder provides some useful tips about this: [here][ned-1]

[ned-1]: https://nedbatchelder.com/text/log-style-guide.html

As you make use of the application logs consider if they can be used for what
they are intended for, or if key information is missing that would otherwise
help an individual to debug a problem from the logs alone.

#### Secrets

Make sure that secrets are NEVER exposed through logging, e.g. when successfully
decrypting a value provided by another user.
### Comments and communication

Empathy driven development is a new development strategy promoting
communication:

- [What is empathy driven development][empathy-1] (warning, long article!).
- [EmDD website][empathy-2].

> Empathy driven development is a human centered approach to building software
that uses continuous communication to generate trust and resilience.

It is placed alongside four other initiatives, all of which are important in one
way or another to this project:

| Methodology                  | Item of focus | Goal                                                        | Descriptive Process                                                                   |
|------------------------------|---------------|-------------------------------------------------------------|---------------------------------------------------------------------------------------|
| Responsibility-driven design | Object        | Improve encapsulation in object oriented systems.           | What actions is this object responsible for? What information does this object share? |
| Test-driven development      | Test          | Reduce the risk of bugs and ship features faster.           | Red, Green, Refactor.                                                                 |
| Domain-driven design         | Domain model  | Use the language of the  domain in the code-base.           | Bounded contexts. Ubiquitous language.                                               |
| Behavior-driven development  | Scenario      | Make acceptance tests executable and easy to execute.       | As a, I want, so that. Given, when, then.                                             |
| Empathy-driven development   | Communication | Improve levels of trust and resilience in software systems. | Care, calm, consider, connect.                                                        |

If we consider all code eventually become legacy, we understand that we are
communicating with empathy for each other, today; and we are communicating to an
audience in the future, be that developers, project-managers, marketeers, or
other maintainers.

Our approach toward comments can push us toward empathy-driven design. It places
an importance on communication artifacts, some of which are:

- Comments and docstrings.
- Pull-requests.
- Commit messages.
- READMEs and documentation.

We shall try to use all of these artifacts in our projects to ensure that
information does not escape us, and is visible to all the folks that need to see
it.

#### Tips for communication outside of code

We have many audiences. Even in the early days of a project, we have the
developers, and then we have those selling or marketing the code. Those
audiences shrink and grow over time with diminishing or increasing relevance, or
greater push and pull.

Communication for developers in the core of our code-base (code, commits, PRs)
will often be `low-entropy`, including lots of acronyms and shortcuts to
describe why we're doing something. Communication outside of GitHub may quickly
become, and indeed, benefit from becoming `high-entropy` (documentation, issues
and feature requests) using greater but more plain-language detail, less
acronyms, and more explicitly explaining benefits and disadvantages and costs.

### Variables and variable naming

- Variables: `snake_case` this is not explicitly idiomatic, but it is a de-facto
standard for Python. It can also read better for more explicit variable naming.
- Explicit naming conventions, i.e. no single value variables `i` may become
`idx`, `x` and `y` may become `graph_x`, `graph_y` etc. Names should be
self-describing.
- Classes: Capitalized, `CamelCase`.

### Type-hinting

Type-hinting provides more explicit, dynamic, and actionable, documentation for
Python projects. They were introduced in Python 3.5 and continue to evolve.

Type-hinting should be used in function signatures to signal their intention.

Type-hints can be validated by tools like [mypy][mypy-1] and a configuration
is being developed that is optimized for our development environment, and
pragmatic, i.e. not too strict, and not too soft.

### Follow the happy path

The happy-path, or my rendering - keeping code left-aligned should be followed
where possible.

> *Example non-happy path (trivial):*
>
> ```python
> def some_function(authenticated: bool) -> dict:
>    """A brief example."""
>    if authenticated:
>       info = read_user_information()
>       # -> potentially do a lot with the information ...
>       return info
>    else:
>       return {"info": "not authenticated"}
> ```

> *Example happy-path (trivial):*
>
> ```python
> def some_function(authenticated: bool) -> int:
>    """A brief example."""
>    if not authenticated:
>       return {"info": "not authenticated"}
>    # We're authenticated: the happy path means less indenting and
>    # less use of else/elif constructs.
>    info = read_user_information()
>    # -> do as much as we want with the information ...
>    return info
> ```

The happy path becomes apparent quickly with a bit of practice. Tips to avoid:

- avoid else-returns,
- return early from code,
- split code into smaller functions keeps code readable, and left-aligned.
- Use EAFP (Easy to Ask Forgiveness than Permission) style coding (more below).

More on the happy path: [Mat Ryer @ Medium.com][happy-1].

### EAFP (Easy to ask forgiveness than permission)

EAFP style coding is less defensive than other styles. Python's approach to
try-except lends itself well to this.

Some more information about EAFP here: [EAFP at Geeks for Geeks][geek-1].

The primary difference is greater use of `try ... except` vs. `if ... then do
... else ... then do...`.

> *Example: ask permission:*
>
> ```python
> def some_function(data: dict) -> str:
>    """A brief example."""
>    if "value" in data:
>       return data["value"]
>    return ""
> ```

> *Example: Ask forgiveness:*

> ```python
> def some_function(data: dict) -> str:
>    """A brief example."""
>    try:
>       return data["value"]
>    except KeyError as err:
>       logging.info("value isn't in the data yet")
>    return ""

We begin to see the happy-path appear better through EAFP in less trivial
examples. One benefit we see in this example is that we become more explicit and
more deliberate about our error handling,

### Code-commits

Code commits are important communication artifacts. They communicate a lot to
the person maintaining this code in five-years time. They are usually made up of
a title and a body.

- Title: 50 characteers max.
- Title style: imperative mode of the form (If applied, this commit will ) `Add
vcrpy to the test coverage`.
- Body: 72 characters line-length.

Small or trivial commits need only use a title (50 chars max), e.g. `git commit
-m "add your message here"` style.

> NB. If GNU's nano is configured as your default Git commit editor, it can help
> to set the following in `/home/<user>/.nanorc` to see the column count:
>
>    ```text
>    set constantshow
>    ```
>
> Configuration of other editors to show column count will vary. If `.nanorc`
> doesn't currently exist on your machine it can be created with just the
> configuration above.

See Chris Beam's blog-post for more on commit messages:
[How to write a Git Commit Message][code-review-1] and their
[Seven rules for good commit messages][code-review-2].

Functionally, individual commits *should* work, but more importantly, they must
specific, i.e. solve one problem, not many.

> E.g. When working on a feature, you may come across spelling errors that need
fixing. These shouldn't be in numerous small commits. Rather, make them your
first or last change, and make them a single commit. As developers it requires
more strategic thinking when writing code, but it makes finding issues in
commits later easier, and is a bit easier to code-review, especially that of a
trusted contributor.

> E.g. Refactoring is always tempting. We don't discuss refactoring here, but
with good unit testing, it is something to be encouraged if something is found
while writing features. A commit, refactoring a small portion of the code,
should be kept separate from the problem solving part of the issue being looked
at, for example, and should be well communicated in the commit and pull-request.

> NB. Practically, when refactoring, if a unit-test doesn't exist for something,
the unit test should be created and committed first - then a second commit that
represents the changes from refactoring.

### Configuring run-times and "failing fast"

When configuring an application or service to run, it can be tempting to help
the user as much as possible, possibly, even correcting something that doesn't
work.

> E.g. an application needs an environment variable to be configured to run. The
user forgets to configure this, so code is written to prompt the user and offer
to create the variable for them. We've created an input for the user, and likely
code-paths that store the variable for a session, or permanently, or locally, or
globally.

This situation can be avoided by failing as soon as possible, e.g. checking for
the existence of the variable, and returning a non-zero exit code with a
message to the user, `please configure the $VARIABLE for this app to run`.

### Concatenation - strings and file-paths

Avoid concatenation with `+` at nearly all costs (I'm not sure what exceptions
there are, except maybe `list` types? Rely on library, functions to concatenate
for us, as they usually handle all the exceptions we would otherwise.

#### File-paths

- `pathlib`, ideal. More future-facing, object-oriented.

> *Example using pathlib:*

> ```python
> from pathlib import Path
> my_path = Path("this/is/a/path")
> my_path.mkdir(parents=True, exist_ok=True)
> my_path.exists()
> ```

> NB. While we normalize to use of the Linux separator `/` in our paths in this
particular example, two `Path(...) / Path` instances can also be concatenated as
such. Both examples instantiates a concrete path for the platform the code is
running on meaning it is platform independent.

More on pathlib [here][pathlib-1] via Trey Hunner.

- `os.path`-like functions, are okay too.

> *Example using os.path:*

> ```python
> from os import makedirs
> from os import path
> my_path = os.path.join("this", "is", "a", "path")
> try:
>    os.makedirs(my_path)
> except FileExistsError:
>    logging.info("path existS")
> ```

> NB. Ideally we'll move toward `pathlib` but there's a learning curve, and it's
not immediately obvious what benefits this object-orient style brings. We'll see
libraries like `pytest` make good use of it for temporary directories and so
forth.

#### Strings

F-strings take a lot of pain out of string concatenation and handle a lot of
type errors without needing to `str(cast)`. They're really simple, and can
include variables, as well as arbitrary expressions.

> *Example f-string:*

> ```python
> string_var = f"There are {len(my_list)} variables in: {my_list}}"
> ```

A reasonable guide on them can be found [here][f-strings-1] courtesy of Real
Python.

### Tests

Some things to keep in mind with tests.

#### Tests are first-class citizens

Maybe even more important than code, tests help us to make broad sweeping
changes when needed, e.g. performance optimizations, or refactoring, and still
have confidence in the result. We simply do the same, as above, but for our
tests 😉.

#### Minimize the use of fixtures

We have a lot of tools tha can help us to create files in-memory, as well as
work with databases, and other data structures in-memory too.

For pytest, take a look at
[how to create temporary files and folders in memory][pytest-1]. In-memory
creation of these objects help keep the code-repository clutter-free, and it
is easy to change these objects in-code and inspect the changes, vs. binary
objects which cannot be easily inspected.

[empathy-1]: https://corgibytes.com/blog/2021/01/12/empathy-driven-development/
[empathy-2]: https://www.empathy-driven-development.com/
[mypy-1]: http://www.mypy-lang.org/
[happy-1]: https://medium.com/@matryer/line-of-sight-in-code-186dd7cdea88
[geek-1]: https://www.geeksforgeeks.org/eafp-principle-in-python/
[code-review-1]: https://chris.beams.io/posts/git-commit
[code-review-2]: https://chris.beams.io/posts/git-commit/#seven-rules
[pathlib-1]: https://treyhunner.com/2018/12/why-you-should-be-using-pathlib
[f-strings-1]: https://realpython.com/python-f-strings/#f-strings-a-new-and-improved-way-to-format-strings-in-python
[pytest-1]: https://docs.pytest.org/en/7.1.x/how-to/tmp_path.html
