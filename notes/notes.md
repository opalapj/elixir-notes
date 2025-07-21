# Internals

`Erlang/OTP` is a complete development environment for concurrent programming.

`Erlang` is a programming language used to build massively scalable soft
real-time systems with requirements on high availability.

`OTP` is set of `Erlang` libraries and design principles providing middle-ware
to develop these systems. It includes its own distributed database, applications
to interface towards other languages, debugging and release handling tools.

`Erlang/OTP` is divided into a number of `OTP` applications.

`Elixir` programming language is broken into 6 applications: `Elixir`, `EEx`,
`ExUnit`, `IEx`, `Logger`, `Mix`. These applications follows `OTP` design
principles.

> That's right! `Elixir` (application) is `OTP` application itself. It is
core/standard library of whole `Elixir` programming language.

Applications normally contains modules, respectively `Erlang` modules and
`Elixir` modules.

More:
- https://www.erlang.org/doc/readme.html
- https://elixir-lang.org/docs.html

## Applications

Applications are a standard way of building:

- `libraries`,
- `supervised systems`.

The simplest applications do not have any processes, but consist of a collection
of functional modules. Such an application is called a `library` application.
An example of a `library` application is `Erlang` `STDLIB` or `Elixir` `EEx`.

More complex applications have a bunch of processes. Such an application is
called a `supervised system` application. An example of a `supervised system`
application is `Erlang` `Kernel` or `Elixir` `Mix`.

Check running applications:

```bash
# Using elixir command.
$ elixir -e "IO.inspect(Application.started_applications())"

# Using elixir shell.
iex> Application.started_applications()

# Using erlang shell.
erl> application:which_applications().
```

More:
- https://www.erlang.org/doc/system/design_principles.html#applications

## Compilation

`Source code` (`.erl` and `.ex` files) must be compiled (by `compiler`) to
`object code`. The current abstract machine, which runs the `object code`,
is called `BEAM`, therefore the object files get the suffix `.beam`.

More:
- https://www.erlang.org/doc/system/code_loading.html#compilation

## Code loading

The `object code` must be loaded into the `Erlang` runtime system. This is
handled by the `code server` (see `code` module in `Kernel` application).

`Code server` maintains a `code path`, consisting of a list of directories,
which it searches sequentially when trying to load a module. Initially
(and mostly), these directories are the root directories of the applications
delivered with `Erlang/OTP` and `Elixir`.

Initially, the `code path` consists of the current working directory and all
`object code` directories under `ROOT/lib`, where `ROOT` is the installation
directory of `Erlang`/`Elixir`. Directories can be named `Name[-Vsn]`, where the
`-Vsn` suffix is optional. By default, the `code server` chooses the directory
with the highest version number among those which have the same `Name`.
If an `ebin` directory exists under the `Name[-Vsn]` directory, this directory
is added to the `code path`.

`code path`:

```bash
# Using elixir command.
$ elixir -e "IO.inspect(:code.get_path())"

# Using elixir shell.
use Erlang api

# Using erlang shell.
erl> code:get_path().
```

`ROOT`:

```bash
# Using elixir command.
$ elixir -e "System.get_env("SCRIPT_PATH") |> Path.join("..") |> Path.expand"

# Using elixir shell.
iex> System.get_env("SCRIPT_PATH") |> Path.join("..") |> Path.expand

# Using erlang shell.
erl> code:root_dir().
```
 
More:
- https://www.erlang.org/doc/system/system_principles.html#code-loading-strategy
- https://www.erlang.org/doc/system/code_loading.html#code-loading
- https://www.erlang.org/doc/apps/kernel/code#module-code-path

## Module names

In `Elixir` environment:

- from `Elixir` e.g. alias `Code` (equal to atom `:"Elixir.Code"`)

    Aliases are commonly used as module names. They must be capitalized and
    written in `CamelCase`. Aliases are constructs that expand to atoms at
    compile-time. The alias `MyModule` expands to the atom `:"Elixir.MyModule"`.

- from `Erlang` e.g. atom `:code`

    In `Erlang` module names also correspond to atoms, practically to their
    simplest, unquoted form.

In `Erlang` environment:

- from `Erlang` e.g. atom `code`  

More:
- https://hexdocs.pm/elixir/1.18.3/alias-require-and-import.html#understanding-aliases
- https://hexdocs.pm/elixir/1.18.3/syntax-reference.html#aliases

## Standalone modules

Three possible ways to use standalone `Elixir` modules by `iex`:

- copy and paste modules definition directly into `iex`
- tell `iex` to interpret the file while starting (in-memory compilation):

    ```bash
    $ iex geometry.ex
    ```

- compile `.ex` files to `.beam` files and run `iex` in the same directory:

    ```bash
    $ elixirc geometry.ex
    $ iex
    ```

## Built-In Functions (`BIFs`)

`BIFs` are implemented in `C` code in the runtime system. `BIFs` do things that
are difficult or impossible to implement in `Erlang`.

> Erlang documentation provides inconsistent info.
>
> On the one hand:
>
> Most of the `BIFs` belong to module `erlang`, but there are also `BIFs`
> belonging to a few other modules, for example `lists` and `ets`.
>
> On the other hand:
>
> For a complete list of `BIFs`, their arguments and return values, see module
> `erlang` in `ERTS`.
>
> More:
> - https://www.erlang.org/doc/system/ref_man_functions#built-in-functions-bifs
> - https://www.erlang.org/doc/system/reference_manual.html#complete-list-of-bifs

## Auto-import

Auto-imported functions do not need to be prefixed with the module name.

In `Elixir`, auto-imported functions come from modules:

- `Kernel` (`Elixir` app)
- `Kernel.SpecialForms` (`Elixir` app)

In `Erlang`, auto-imported functions come from modules:

- `erlang` (`ERTS` app)

## Inlining from `Erlang` to `Elixir`

In the `Elixir` some of the functions from `Kernel` module are inlined by the
`Elixir` compiler into their `Erlang BIFs` counterparts in the `erlang` module. 

> Erlang auto-imported functions are not auto-imported in `Elixir`, eg.
> `pid_to_list`
>
> In Erlang:
>
> ```bash
> erl> h(erlang, pid_to_list, 1).
> -spec pid_to_list(Pid) -> string() when Pid :: pid().
> erl> erlang:pid_to_list(self()) == pid_to_list(self()).
> true
> ```
>
> In `Elixir`:
>
> ```bash
> iex> h(pid_to_list)
> No documentation for Kernel.pid_to_list was found
> iex> h(:erlang.pid_to_list)
> @spec pid_to_list(pid) :: string() when pid: pid()
> ```

More:
- https://hexdocs.pm/elixir/1.18.3/Kernel.html#module-inlining

## AST

`Elixir` `AST` is converted to `Erlang` `AST`.

`Erlang VM` itself can run multiple languages like `Erlang`, `Elixir`, `Joxa`,
`Efene` and more.

More:
- https://youtu.be/IGmwiyines0?t=2396

## Default environment

```bash
$ elixir -e "IO.inspect(Application.started_applications())"
[
  {:logger, ~c"logger", ~c"1.17.3"},
  {:elixir, ~c"elixir", ~c"1.17.3"},
  {:compiler, ~c"ERTS  CXC 138 10", ~c"8.5.4"},
  {:stdlib, ~c"ERTS  CXC 138 10", ~c"6.2"},
  {:kernel, ~c"ERTS  CXC 138 10", ~c"10.2"}
]
```

- from `Elixir` system:

    - Elixir (`:elixir`) - standard library
    - Logger (`:logger`) - built-in Logger

- from `Erlang` system (minimal system based on `Erlang/OTP` consists of
`Kernel` and `STDLIB`):

    - Kernel (`:kernel`) - code necessary to run the `Erlang` runtime system,
    first application started
    - STDLIB (`:stdlib`) - `Erlang` standard libraries, contains no
    services
    - Compiler (`:compiler`) - interface to the standard `Erlang` compiler

# Application development

Applications are the idiomatic way to package software in `Erlang/OTP`. To get
the idea, they are similar to the "library" concept common in other programming
languages, but with some additional characteristics.

An application is a component implementing some specific functionality, with
a standardized directory structure, configuration, and life cycle. Applications
are loaded, started, and stopped. Each application also has its own environment,
which provides a unified API for configuring each application.

## `Mix`

`Mix` is a build tool that provides tasks for creating, compiling, and testing
`Elixir` projects, managing its dependencies, and more.

More:
- https://hexdocs.pm/mix/1.18.3/Mix.html
- https://hexdocs.pm/elixir/Application.html#module-tooling
- https://hexdocs.pm/elixir/introduction-to-mix.html
- https://hexdocs.pm/elixir/library-guidelines.html

## `mix.exs`

Its main responsibility is to configure our project.

The foundation of `Mix` is a project. A project can be defined by using
`Mix.Project` in a module, usually placed in a file named `mix.exs`.

When you call `use Mix.Project`, it notifies `Mix` that a new project has been
defined, so all `Mix` tasks use your module as a starting point. Once the
project is defined, a number of default `Mix` tasks can be run directly from
the command line:

- `mix compile` - compiles the current project
- `mix test` - runs tests for the given project
- `mix run` - runs a particular command inside the project

More:
- https://hexdocs.pm/mix/1.18.3/Mix.html
- https://hex.pm/docs/publish#example-mixexs-file

## Understanding `iex -S mix`

`iex` accepts all other options listed by `elixir`, e.g. `-S` what means "find
and execute the given script in $PATH". One of those scripts is `mix`. `mix`
runs the default task which is `mix run`. `mix run` starts the current
application dependencies and the application itself. The application will be
compiled if it has not been compiled yet or it is outdated.

So, `iex -S mix`:
- compiles source code to object code
- starts application
- starts interactive shell

More:
- `iex --help`
- `elixir --help`
- `mix --help`

## Code paths pruning

`Mix` caches and prunes load paths before compilation, ensuring your project
(and dependencies!) compile faster and in an environment closer to production.

To see differences:

```bash
$ elixir -e "IO.inspect(:code.get_path())"
$ mix run -e "IO.inspect(:code.get_path())"
```

More:
- https://hexdocs.pm/elixir/1.15.4/changelog.html#compile-and-boot-time-improvements

## Compilation

`mix compile` simply runs the compilers registered in your project and returns a
tuple with the compilation status and a list of diagnostics.

```bash
$ iex -S mix
iex> Mix.compilers()
[:erlang, :elixir, :app]
```

More:
- https://hexdocs.pm/mix/1.18.3/Mix.Tasks.Compile.html

## Starting applications

When an application starts, developers may configure a callback module that
executes custom code. Developers use this callback to start the application
supervision tree.

The first step to do so is to add a `:mod` key to the `application/0` definition
in your `mix.exs` file. It expects a tuple, with the application callback module
and start argument (commonly an empty list).

The callback module given to `:mod` needs to implement the `Application`
behaviour. This can be done by putting `use Application` in that module and
implementing the `start/2` callback.

More:
- https://hexdocs.pm/elixir/supervisor-and-application.html#the-application-callback
- https://hexdocs.pm/elixir/Application.html#module-the-application-callback-module

## Project configuration (`project/0`)

In order to configure `Mix`, the module that calls `use Mix.Project` should
export a `project/0` function that returns a keyword list representing
configuration for the project.

- `:app` (required), `:version` (required)

    https://hexdocs.pm/mix/1.18.3/Mix.Tasks.Compile.App.html

- `:elixir`

    https://hexdocs.pm/mix/1.18.3/Mix.Tasks.Loadpaths.html#module-configuration

- `:start_permanent`

    https://hexdocs.pm/mix/1.18.3/Mix.Tasks.App.Start.html

- `:deps`

    https://hexdocs.pm/mix/1.18.3/Mix.Tasks.Deps.html

More:
- https://hexdocs.pm/mix/1.18.3/Mix.Project.html

## Application configuration (`application/0`)

`application/0` function is used to generate an application resource file, which
is a file called `APP_NAME.app`.

- `:extra_applications`, `:mod`

    https://hexdocs.pm/mix/Mix.Tasks.Compile.App.html

More:
- https://hexdocs.pm/elixir/introduction-to-mix.html#project-compilation
- https://hexdocs.pm/elixir/Application.html#module-the-application-environment

## Environments

More:
- https://hexdocs.pm/elixir/1.18.3/introduction-to-mix.html#environments
- https://hexdocs.pm/mix/1.18.3/Mix.html#module-environments

## Directory structure

More:
- https://www.erlang.org/doc/system/applications.html#directory-structure
- https://www.erlang.org/doc/apps/kernel/code#priv_dir/1
- https://www.erlang.org/doc/apps/kernel/code#lib_dir/2
- https://hexdocs.pm/mix/1.18.3/Mix.Tasks.Release.html#module-directory-structure

## `Hex`

`Hex` is the package manager for the `Erlang` ecosystem.

`Hex` is usable out of the box in `Elixir` with `Mix` and in `Erlang` with
`Rebar3`.

### Installation

`Mix` will automatically prompt you whenever there is a need to use `Hex`. In
case you want to manually install or update `hex`, simply run:

```bash
$ mix local.hex
```

## Dependencies

Dependencies must be specified in the `mix.exs` file.

```elixir
[
  {:plug, "~> 1.18"}
]
```

More:
- https://hexdocs.pm/mix/1.18.3/Mix.Tasks.Deps.html

### List

```bash
mix deps
```

### Fetch

```bash
mix deps.get
```

New
- `mix.lock`
- `deps/`
- `_build/dev/lib/APP_NAME/.mix/compile.lock`

### Compile

```bash
mix deps.compile
```

New
- `_build/dev/lib/DEP_NAME`

### Remove

```bash
mix deps.clean
```

## Versioning, requirements

https://hexdocs.pm/elixir/Version.html

## Useful tasks

- `mix new`
- `mix compile`
- `mix run`
- `mix clean`
- `mix deps`
- `mix format`

# `IEx` - interactive shell

https://hexdocs.pm/iex/1.18.3/IEx.html

## `h()`

https://hexdocs.pm/iex/1.18.3/IEx.html#module-helpers

## `.iex.exs`

https://hexdocs.pm/iex/1.18.3/IEx.html#module-the-iex-exs-file

## `#iex:break`

https://hexdocs.pm/iex/1.18.3/IEx.html#module-expressions-in-iex

## `IEx.Info.info/1`

https://hexdocs.pm/iex/1.18.3/IEx.Info.html

# Typespecs

There are 3 types of typespecs (actually module attributes):
- `@type`, defines a type to be used in `@spec`
- `@spec`, provides a specification for a function
- `@callback`, provides a specification for a behaviour callback

More:
- https://hexdocs.pm/elixir/1.18.3/typespecs.html
- https://hexdocs.pm/elixir/1.18.3/Module.html#module-typespec-attributes

## Getting typespecs

### Types

- list all module types (i.e. all `@type` attributes)

    ```bash
    iex> t GenServer
    ```

- get doc for specific type (i.e. `@typedoc` attribute prepended to specific
`@type` attribute)

    ```bash
    iex> t GenServer.name
    ```

### Functions

- get typespecs for specific function (i.e. `@spec` attribute prepended to
specific function definition)

    ```bash
    iex> h GenServer.whereis
    ```

### Callbacks

- list all module callbacks (i.e. all `@callback` attributes)

    ```bash
    iex> b GenServer
    ```

- get doc for specific callback (i.e. `@doc` attribute prepended to specific
`@callback` attribute)

    ```bash
    iex> b GenServer.init
    ```

## Short form

Most of the `built-in` types provided in `Erlang` (for example, `pid()`) are
expressed in the same way: `pid()` (or simply `pid`).

## Syntactical notation

Some types can also be declared using their syntactical notation, such as
`[type]` for lists, `{type1, type2, ...}` for tuples and `<<_ * _>>`
for binaries.

## Union type

The notation to represent the union of types is the pipe `|`. For example, the
typespec `type :: atom() | pid() | tuple()` creates a type that can be either an
`atom`, a `pid`, or a `tuple`.

## Empty collections

Note that the syntactic representation of `map()` is `%{optional(any) => any}`,
not `%{}`. The notation `%{}` specifies the singleton type for the empty `map`.

## Naming arguments

You can also name your arguments in a typespec using `arg_name :: arg_type`
syntax. This is particularly useful in documentation as a way to differentiate
multiple arguments of the same type (or multiple elements of the same type in a
type definition):

```elixir
@type color :: {red :: integer, green :: integer, blue :: integer}

@spec days_since_epoch(year :: integer, month :: integer, day :: integer) :: integer
def days_since_epoch(year, month, day) do
  ...
end
```

## Guards in typespecs

Source:

- https://hexdocs.pm/elixir/1.18.3/typespecs.html#defining-a-specification

Explanation:

- https://bitsonthemind.com/posts/guards-in-typespecs/
- https://til.hashrocket.com/posts/mlioeeoyfd-guard-clause-on-elixir-typespec
- https://arunramgt.medium.com/elixir-basics-of-type-specifications-76bb21aa0ef0#2663

# Docs

`Elixir` documentation is written using `Markdown`.

There are 3 types of docs (actually module attributes):
- `@moduledoc`, used to add documentation to:
    - module - `defmodule`, see: `h GenServer`
    - protocol - `defprotocol`, see: `h Enumerable`
- `@doc`, used to add documentation to:
    - function - `def`, see: `h GenServer.whereis`
    - behaviour callback - `@callback`, see: `b GenServer.handle_info`
    - struct - `defstruct`, see: `h Task.__struct__`
- `@typedoc`, used to add documentation to:
    - types - `@type`, see: `t Task.t`

Accept one of these:

- a string (often a `heredoc`):

    ```elixir
    @doc "Hello string"

    @doc """
    Hello heredoc
    """
    ```

    > `Elixir` promotes the use of `Markdown` with `heredocs` to write readable
    > documentation. `Heredocs` are multi-line strings, they start and end
    > with triple double-quotes, keeping the formatting of the inner text.

- `false`, which will make the entity invisible to documentation-extraction
tools like `ExDoc`:

    ```elixir
    @doc false
    ```

    > Function with `@doc false` are not listed with tab completion in `iex`.
    > But there are still listed using `exports` helper.

- a keyword list, since `Elixir` 1.7.0:

    ```elixir
    @doc since: "1.1.0"
    ```

More:
- https://hexdocs.pm/elixir/1.18.3/writing-documentation.html
- https://hexdocs.pm/elixir/1.18.3/module-attributes.html
- https://hexdocs.pm/elixir/1.18.3/Module.html#module-moduledoc
- https://hexdocs.pm/elixir/1.18.3/Module.html#module-doc-and-typedoc

## Getting docs

### Modules

- get doc for specific module (i.e. `@moduledoc` attribute prepended to specific
module definition)

    ```bash
    iex> h GenServer
    ```

### Functions

- get doc for specific function (i.e. `@doc` attribute prepended to specific
function definition)

    ```bash
    iex> h GenServer.whereis
    ```

### Types

- get doc for specific type (i.e. `@typedoc` attribute prepended to specific
`@type` attribute)

    ```bash
    iex> t GenServer.name
    ```

## Function arguments & pattern matching

When documenting a function, argument names are inferred by the compiler.
Best practice is to put only the function head and argument names of function at
first and then next definitions according to various patterns:

```elixir
def filter_by_year(wines, year)
def filter_by_year([], _year), do: ...
def filter_by_year([{_, year, _} = wine | tail], year), do: ...
def filter_by_year([{_, _, _} | tail], year), do: ...
```

Pattern matching in third definition:

```elixir
def filter_by_year([{_, year, _} = wine | tail], year) do
  [wine | filter_by_year(tail, year)]
end
```

not only match first argument but also uses it (`wine`) in local scope.

Another interesting thing is using second argument `year` in pattern matching
for first argument. Local scope is already being created from name and
arguments.

More:
- https://hexdocs.pm/elixir/1.18.4/writing-documentation.html#function-arguments

## Docs metadata

Source:

- https://hexdocs.pm/elixir/1.18.3/Module.html#module-doc-and-typedoc

Explanation:

- https://github.com/elixir-lang/ex_doc/?tab=readme-ov-file#metadata

## `@doc` attribute persistance

```elixir
defmodule MyApp do
  IO.inspect(Module.get_attribute(__MODULE__, :doc), label: :bef_doc_declar)
  IO.inspect(Module.has_attribute?(__MODULE__, :doc), label: :bef_doc_declar)

  @doc "Doc for foo."

  IO.inspect(Module.get_attribute(__MODULE__, :doc), label: :aft_doc_declar)
  IO.inspect(Module.has_attribute?(__MODULE__, :doc), label: :aft_doc_declar)

  def foo, do: :nothing

  IO.inspect(Module.get_attribute(__MODULE__, :doc), label: :aft_foo_declar)
  IO.inspect(Module.has_attribute?(__MODULE__, :doc), label: :aft_foo_declar)

  @doc "Doc for bar."

  IO.inspect(Module.get_attribute(__MODULE__, :doc), label: :aft_doc_declar)
  IO.inspect(Module.has_attribute?(__MODULE__, :doc), label: :aft_doc_declar)

  def bar, do: :nothing

  IO.inspect(Module.get_attribute(__MODULE__, :doc), label: :aft_bar_declar)
  IO.inspect(Module.has_attribute?(__MODULE__, :doc), label: :aft_bar_declar)
end
```

```bash
$ iex -S mix
Erlang/OTP 27 [erts-15.2] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [jit:ns]

Compiling 1 file (.ex)
bef_doc_declar: nil
bef_doc_declar: false
aft_doc_declar: {5, "Doc for foo."}
aft_doc_declar: true
aft_foo_declar: nil
aft_foo_declar: false
aft_doc_declar: {15, "Doc for bar."}
aft_doc_declar: true
aft_bar_declar: nil
aft_bar_declar: false
Interactive Elixir (1.17.3) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> Code.fetch_docs(MyApp)
{:docs_v1, 1, :elixir, "text/markdown", :none, %{},
  [
    {{:function, :bar, 0}, 15, ["bar()"], %{"en" => "Doc for bar."}, %{}},
    {{:function, :foo, 0}, 5, ["foo()"], %{"en" => "Doc for foo."}, %{}}
  ]}
iex(2)>
```

## Hiding docs for private functions

```elixir
@doc """
Function that does nothing.
""" && false
defp do_nothing do
  nil
end
```

# Module structure

More:
- https://hexdocs.pm/elixir/1.18.3/Module.html#c:__info__/1
- https://hexdocs.pm/elixir/1.18.3/Module.html#module-generated-functions

# Struct

A struct is a tagged map that allows developers to provide default values for
keys, tags to be used in polymorphic dispatches and compile time assertions.

It is only possible to define a struct per module, as the struct is tied to the
module itself.

More:
- https://hexdocs.pm/elixir/1.18.4/structs.html
- https://hexdocs.pm/elixir/1.18.4/Kernel.html#defstruct/1
- https://hexdocs.pm/elixir/1.18.4/Module.html#module-struct-attributes
- https://hexdocs.pm/elixir/1.18.4/Kernel.SpecialForms.html#%25/2

## Deriving

More:
- https://hexdocs.pm/elixir/1.18.4/Protocol.html#c:__deriving__/2
- https://hexdocs.pm/elixir/1.18.4/Protocol.html#derive/3
- https://hexdocs.pm/elixir/1.18.4/Inspect.html#module-deriving
- https://hexdocs.pm/iex/1.18.4/IEx.Info.html

## Matching structs against maps

The `__struct__` field has an important consequence on pattern matching.
A struct pattern can’t match a plain map, but a plain map pattern can match
a struct:

```bash
iex> roksi = %Dog{name: "Roxana", age: 3}
%Dog{name: "Roxana", age: 3, breed: "Barker"}
iex> %{breed: rasa} = roksi
%Dog{name: "Roxana", age: 3, breed: "Barker"}
iex> rasa
"Barker"
```

Remember, all fields from the pattern must exist in the matched term.

## Full example of struct and protocol

```elixir
defmodule Dog do
  @doc """
  The Dog struct.

  Contains these fields:
  * `:name` - name of the dog
  * `:age` - age of the dog
  * `breed` - breed of the dog
  """
  @derive {Inspect, only: [:name]}
  @enforce_keys [:name]
  defstruct [:name, :age, breed: "Barker"]

  @typedoc """
  The Dog type.

  See [`%Dog{}`](`__struct__/0`) for information about each field of the structure.
  """
  @type t :: %__MODULE__{name: String.t(), age: non_neg_integer, breed: String.t()}
end

defimpl IEx.Info, for: Dog do
  def info(%module{name: name, age: age, breed: breed}) do
    [
      {"Data type", inspect(module)},
      {"Description", "My dog #{name} is #{age} year old and it is #{breed} breed."},
      {"Reference modules", inspect(module) <> ", Map"}
    ]
  end
end

defprotocol Animal do
  @moduledoc """
  The `Animal` protocol provides API to work with the most popular pets.
  """

  @doc """
  Presents the characteristic sound made by a particular pet.
  """
  @spec speak(t) :: String.t()
  def speak(animal)
end

defimpl Animal, for: Dog do
  def speak(_dog), do: "Woof"
end
```

# Protocols

Protocols are a mechanism to achieve polymorphism in `Elixir` where you want the
behavior to vary depending on the data type.

A protocol is defined with `Kernel.defprotocol/2` and its implementations with
`Kernel.defimpl/3`.

We define the protocol using `defprotocol/2` - its functions and specs may look
similar to interfaces or abstract base classes in other languages. We can add as
many implementations as we like using `defimpl/2`.

More:
- https://hexdocs.pm/elixir/1.17.3/protocols.html#content
- https://hexdocs.pm/elixir/1.18.4/Protocol.html

## Defining

```elixir
defprotocol Animal do
  @moduledoc """
  The `Animal` protocol provides API to work with the most popular pets.
  """

  @doc """
  Presents the characteristic sound made by a particular pet.
  """
  @spec speak(t) :: String.t()
  def speak(animal)
end
```

## Implementing

```elixir
defimpl Animal, for: Dog do
  def speak(_dog), do: "Woof"
end
```

## Implementing `Any`

Manually implementing protocols for all types can quickly become repetitive and
tedious. In such cases, `Elixir` provides two options:
- explicitly derive the protocol implementation for specific type
- implicitly derive the protocol implementation for all types

In both cases, we need to implement the protocol for `Any`.

Example definition and implementation:

```elixir
defprotocol Size do
  @doc "Calculates the size (and not the length!) of a data structure"
  def size(data)
end

defimpl Size, for: BitString do
  def size(string), do: byte_size(string)
end

defimpl Size, for: Map do
  def size(map), do: map_size(map)
end

defimpl Size, for: Tuple do
  def size(tuple), do: tuple_size(tuple)
end

defmodule User do
  defstruct [:name, :age]
end
```

### Add implementation for `Any`

```elixir
defimpl Size, for: Any do
  def size(term) do
    type =
      term
      |> IEx.Info.info()
      |> Enum.find_value(fn {x, y} -> if x == "Data type", do: y end)

    "Impossible to determine size of #{type}"
  end
end
```

### Explicitly derive the protocol implementation for specific type

Once the implementation for `Any` is defined, there are two ways it can be
derived:

- using the `@derive` module attribute by the time you define the struct:

    ```elixir
    defmodule User do
    @derive [Size]
    defstruct [:name, :age]
    end
    ```

- using the `Protocol.derive/3` macro, if the struct has already been defined:

    ```elixir
    require Protocol
    Protocol.derive(Size, User)
    ```

### Implicitly derive the protocol implementation for all types

This can be achieved by setting `@fallback_to_any` to `true` in the protocol
definition:

```elixir
defprotocol Size do
  @fallback_to_any true
  @doc "Calculates the size (and not the length!) of a data structure"
  def size(data)
end
```

## Checking implemented protocols

```bash
iex> i %{}
Term
  %{}
Data type
  Map
Reference modules
  Map
Implemented protocols
  Collectable, Enumerable, IEx.Info, Inspect, Plug.Exception
```

## Extra

- Is the module a protocol?

    1. Using `IEx` helper, `Protocol` chapter

        ```bash
        iex> i Enumerable
        ```

    2. Presence of `__protocol__` callback

        ```bash
        Kernel.function_exported?(Enumerable, :__protocol__, 1)
        ```

- Does the type have any protocol implemented?

    1. Using `IEx` helper, `Implemented protocols` chapter

        ```bash
        iex> i [1, 2, 3]
        ```

    2. Using set of functions

        ```bash
        :code.get_path()
        |> Protocol.extract_protocols()
        |> Enum.uniq()
        |> Enum.filter(fn protocol -> protocol.impl_for([1, 2, 3]) != nil end)
        |> Enum.any?()
        ```

- Does the type have a specific protocol implemented?

    1. Using `IEx` helper, `Implemented protocols` chapter

        ```bash
        iex> i [1, 2, 3]
        ```

    2. Using set of functions

        ```bash
        :code.get_path()
        |> Protocol.extract_protocols()
        |> Enum.uniq()
        |> Enum.filter(fn protocol -> protocol.impl_for([1, 2, 3]) == Enumerable.List end)
        |> Enum.any?()
        ```

# Behaviours

Behaviours in `Elixir` (and `Erlang`) are a way to separate and abstract

- the generic part of a component (which becomes the `behaviour module`)

from

- the specific part (which becomes the `callback module`).

A `behaviour module` defines a set of functions and macros (referred to as
`callbacks`) that `callback modules` (implementing that behaviour) must export.

More:
- https://hexdocs.pm/elixir/1.18.3/typespecs.html#behaviours

## Defining callbacks

Example - `c:GenServer.init/1`

```elixir
@callback init(init_arg :: term) ::
            {:ok, state}
            | {:ok, state, timeout | :hibernate | {:continue, continue_arg :: term}}
            | :ignore
            | {:stop, reason :: any}
          when state: any
```

Defining a callback is a matter of defining a specification for that callback,
made of:

- the callback name (`init`)
- the arguments that the callback must accept (`init_arg :: term`)
- the expected type of the callback return value (`{:ok, state} | ...`)

## Implementing behaviours

```elixir
defmodule CallbackModule do
  @behaviour GenServer

  @impl GenServer
  def init(_init_arg) do
    {:ok, nil}
  end
end
```

If a module adopting a given behaviour (i.e. `callback module`) doesn't
implement one of the callbacks required by that behaviour, a compile-time
warning will be generated.

Furthermore, with `@impl` you can also make sure that you are implementing
the correct callbacks from the given behaviour in an explicit manner.

More:
- https://hexdocs.pm/elixir/1.18.3/typespecs.html#implementing-behaviours
- https://hexdocs.pm/elixir/1.18.3/Module.html#module-impl-since-v1-5-0

## Optional callbacks

https://hexdocs.pm/elixir/1.18.3/typespecs.html#optional-callbacks

## `@behaviour` enclosed in `__using__/1` (`use` use-cases)

`@behaviour` is most often set in definitions of `__using__/1` macro.
`__using__/1` macro is called by `use` macro.

Examples:

- https://hexdocs.pm/elixir/1.18.3/GenServer.html
- https://hexdocs.pm/elixir/1.18.3/Application.html

## Extra

- Is the module a behaviour?

    ```bash
    Kernel.function_exported?(GenServer, :behaviour_info, 1)
    ```

- Does the module adopt any behaviour?

    ```bash
    Keyword.has_key?(Dog.module_info(:attributes), :behaviour)
    ```

- Does the module adopt a specific behaviour?

    ```bash
    :attributes
    |> Dog.module_info()
    |> Keyword.get_values(:behaviour)
    |> List.flatten()
    |> Enum.member?(Animal)
    ```

# `use`

The `use` macro is frequently used as an extension point. This means that, when
you use a module `FooBar`, you allow that module to inject any code in the
current module, such as importing itself or other modules, defining new
functions, setting a module state, etc.

More:
- https://hexdocs.pm/elixir/1.18.3/alias-require-and-import.html#use
- https://hexdocs.pm/elixir/Kernel.html#use/2

# Metaprogramming

There are five `Elixir` literals that, when quoted, return themselves (and not
a tuple). They are:

```elixir
:sum         #=> Atoms
1.0          #=> Numbers
[1, 2]       #=> Lists
"strings"    #=> Strings
{key, value} #=> Tuples with two elements
```

## Quoting

Straightforward conversion from `Elixir` syntax to structure resembling a tree.
Many languages would call such representations an Abstract Syntax Tree (`AST`).
`Elixir` calls them `quoted expressions` (`quoted` for short).

```elixir
iex> quote do: sum(1, 2, 3)
{:sum, [], [1, 2, 3]}
```

`Elixir` syntax: `sum(1, 2, 3)`

`AST` / `qouted expression`: `{:sum, [], [1, 2, 3]}`

# OOP pillars vs functional programming

In a typical OOP, the basic abstraction building blocks are classes and objects.
For example, there may be a `String` class available that implements various
string operations. Each string is then an instance of that class that can be
manipulated by calling methods, as the following `Python` snippet illustrates:

```python
"a string".upper()
```

This approach generally isn’t used in functional programming. E.g. `Elixir`
promotes decoupling of data from the code. Instead of classes, you use modules,
which are collection of functions. Instead of calling methods on objects, you
explicitly call module functions and provide input data via arguments. The
following snippet shows the `Elixir` way of uppercasing the string:

```elixir
String.upcase("a string")
```

OOP (e.g. `Python`):
- classes
- methods
- attributes
- properties

Functional programming (e.g. `Elixir`):
- modules
- functions

## Inheritance

Inheritance is one of the fundamental concepts of object oriented programming
and expresses the fundamental relationships between classes: superclasses
(parents) and their subclasses (descendants). Inheritance creates a class
hierarchy. Any object bound to a specific level of class hierarchy inherits all
the traits (methods and attributes) defined inside any of the superclasses.

This means that inheritance is a way of building a new class, not from scratch,
but by using an already defined repertoire of traits. The new class inherits
(and this is the key) all the already existing equipment, but is able to add
some new features if needed.

Each subclass is more specialized (or more specific) than its superclass.
Conversely, each superclass is more general (more abstract) than any of its
subclasses.

### Python

```python
class Animal:
    def move(self):
        return "I can move"

class Dog(Animal):
    def bark(self):
        return "Woof!"

dog = Dog()
print(dog.move())  # Output: I can move
print(dog.bark())  # Output: Woof!
```

### Elixir

use

defdelegate

## Polymorphism

Polymorphism is the provision of a single interface to objects of different
types. In other words, it is the ability to create abstract methods from
specific types in order to treat those types in a uniform way.

Imagine that you have to print a `string` or an `integer` — it is more
convenient when a function is called simply `print`, not `print_string`
or `print_integer`.

However, the `string` must be handled differently than the `integer`, so there
will be two implementations of the function that lead to printing, but naming
them with a common name creates a convenient abstract interface independent of
the type of value to be printed.

Summary:
- polymorphism is used when different class objects share conceptually similar
methods (but are not always inherited)
- polymorphism leverages clarity and expressiveness of the application design
and development
- when polymorphism is assumed, it is wise to handle exceptions that could pop
up

Methods to achieve polymorphism:

- duck typing

    Duck typing is a fancy name for the term describing an application of the
    duck test: "If it walks like a duck and it quacks like a duck, then it must
    be a duck", which determines whether an object can be used for a particular
    purpose. An object's suitability is determined by the presence of certain
    attributes, rather than by the type of the object itself.

    > In duck typing, we believe that objects own the methods that are called.
    > If they do not own them, then we should be prepared to handle exceptions.

    ### Python

    ```python
    class Cat:
        def speak(self):
            return "Meow"

    class Dog:
        def speak(self):
            return "Woof"

    def animal_sound(animal):
        return animal.speak()  # Doesn't care about the class, only the method

    print(animal_sound(Cat()))  # Output: Meow
    print(animal_sound(Dog()))  # Output: Woof
    ```

- inheritance

    One way to carry out polymorphism is inheritance, when subclasses make use
    of base class methods, or override them. You can use inheritance to create
    polymorphic behavior, and usually that's what you do, but that's not what
    polymorphism is about.

    > In inheritance, all subclasses are equipped with methods named the same
    > way as the methods present in the superclass.

    ### Python

    ```python
    class Animal:
        def speak(self):
            return "Some sound"

    class Dog(Animal):
        def speak(self):
            return "Woof"

    class Cat(Animal):
        def speak(self):
            return "Meow"

    def animal_sound(animal: Animal):
        return animal.speak()

    print(animal_sound(Cat()))  # Output: Meow
    print(animal_sound(Dog()))  # Output: Woof
    ```

- abstraction

    If we want our code to be polymorphic, all subclasses have to deliver a set
    of their own method implementations in order to call them by using common
    method names.

    > Abstraction is the most effective way to achieve polymorphism. It forces
    > the provision of a common interface in newly implemented classes.

    ### Python

    ```python
    from abc import ABC, abstractmethod

    class Animal(ABC):
        @abstractmethod
        def speak(self):
            pass

    class Dog(Animal):
        def speak(self):
            return "Woof"

    class Cat(Animal):
        def speak(self):
            return "Meow"

    def animal_sound(animal: Animal):
        return animal.speak()

    print(animal_sound(Cat()))  # Output: Meow
    print(animal_sound(Dog()))  # Output: Woof
    ```

### Elixir

protocol
behaviour

## Abstraction

Abstraction is a way of providing a common interface to other newly implemented
classes.

Abstraction focuses on hiding implementation details and exposing only the
essential functionality of an object. By enforcing a consistent interface,
abstraction simplifies interactions with objects, allowing developers to focus
on what an object does rather than how it achieves its functionality.

### Python

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def make_sound(self):
        pass

class Dog(Animal):
    def make_sound(self):
        return "Woof!"

dog = Dog()
print(dog.make_sound())  # Output: Woof!
```

### Elixir

behaviour

## Encapsulation

Encapsulation describes the idea of bundling attributes and methods that work on
those attributes within a class.

Encapsulation is used to hide the attributes inside a class like in a capsule,
preventing unauthorized parties direct access to them. Publicly accessible
methods are provided in the class to access the values, and other objects call
those methods to retrieve and modify the values within the object. This can be a
way to enforce a certain amount of privacy for the attributes.

### Python

```python
class BankAccount:
    def __init__(self):
        self.__balance = 0  # private attribute

    def deposit(self, amount):
        self.__balance += amount

    def get_balance(self):
        return self.__balance

acc = BankAccount()
acc.deposit(100)
print(acc.get_balance())  # Output: 100
```

# Anonymous functions (`fn`)

More:
- https://hexdocs.pm/elixir/1.18.4/anonymous-functions.html
- https://hexdocs.pm/elixir/1.18.4/Function.html
- https://hexdocs.pm/elixir/1.18.4/Kernel.SpecialForms.html#fn/1
- https://hexdocs.pm/elixir/1.18.4/Kernel.SpecialForms.html#&/1

## Capture operator (`&`) and placeholders (`&1, &2, ...`)

Capture operator is used:

- to capture public module functions and pass them around as if they were
anonymous functions

    ```elixir
    add = &Kernel.+/2
    ```

- as a shortcut for creating functions that wrap existing functions

    ```elixir
    add_two = &Kernel.+(&1 + 2)
    ```

- as a shortcut for creating anonymous functions

    ```elixir
    add = &(&1 + &2)

    instead of

    add = fn a, b -> a + b end
    ```

## Omitting brackets (`&(...)`)

In some cases brackets can be ommited, e.g.:

- strings

    ```elixir
    fun = &("Hello #{&1}")
    fun = &"Hello #{&1}"
    ```

- tuples

    ```elixir
    fun = &({&1, &2})
    fun = &{&1, &2}
    ```

- lists

    ```elixir
    fun = &([&1 | &2])
    fun = &[&1 | &2]
    ```

## Restrictions of using capture operator

- at least one placeholder must be present:

    ```elixir
    &(:foo)
    ```

- block expressions are not supported:

    ```elixir
    &(&1; &2)
    ```

# String

## Interpolation

https://hexdocs.pm/elixir/1.18.3/String.html#module-interpolation

# Keyword Lists vs Maps

## Keyword Lists

- keys must be atoms
- keys are ordered, as specified by the developer
- keys can be given more than once
- pattern matching does require the number of items and their order to match
- dynamic-only access:

    ```elixir
    map = [name: "john", age: 42]
    # dynamic
    map[:name]
    ```

## Maps

- keys can be any value
- keys have their own internal ordering
- keys can not be given more than once
- pattern matching does not require the number of items and their order to match
- dynamic and static access:

    ```elixir
    map = %{name: "john", age: 42}
    # dynamic
    map[:name]
    # static
    map.name
    ```

## Summary

- use keyword lists for passing optional values to functions
- use maps for general key-value data structures
- use maps when working with data that has a predefined set of keys
- use maps for pattern matching

# Truthy and falsy values

In `Elixir`, all datatypes evaluate to a `truthy` or `falsy` value when they are
encountered in a boolean context (like an `if` expression). All data is
considered `truthy` except for `false` and `nil`. In particular, empty strings,
the integer 0, and empty lists are all considered truthy in `Elixir`.

More:
- https://hexdocs.pm/elixir/1.18.3/Kernel.html#module-truthy-and-falsy-values
- https://hexdocs.pm/elixir/1.18.3/basic-types.html#booleans-and-nil

# Miscellaneous

## Scripting mode (`.ex` vs `.exs`)

https://hexdocs.pm/elixir/1.18.4/modules-and-functions.html#scripting-mode

## Matching struct name (`%_{}`)

```elixir
defmodule User do
  defstruct [:email, name: "John", age: 27]
end
```

```bash
iex> %module_name{} = %User{name: "Piotr", age: 30, email: "opalap@op.pl"}
%User{email: "opalap@op.pl", name: "Piotr", age: 30}

iex> module_name
User

iex> %_{} = %User{name: "Piotr", age: 30, email: "opalap@op.pl"}
%User{email: "opalap@op.pl", name: "Piotr", age: 30}
```

## Matching non-empty list (`[]` vs `[_|_]`)

```elixir
defmodule TestList do
  def empty?([]), do: true
  def empty?([_|_]), do: false
end
```

## Matching-all patterns (`_` vs `:_`)

`_` - match-all **variable/pattern** in matching with match operator `=`

`:_` - match-all **value/expression** in matching with functions e.g.
`Registry.match/3`

More:
- https://hexdocs.pm/elixir/1.18.3/pattern-matching.html
- https://hexdocs.pm/elixir/1.18.3/Registry.html#match/3

## `binding`

https://hexdocs.pm/elixir/Kernel.html#binding/1

## `:sys.get_state`

https://www.erlang.org/doc/apps/stdlib/sys.html#get_state/1

## "Let it crash"

At the end of the day, "fail fast" / "let it crash" is a way of saying that,
when something unexpected happens, it is best to start from scratch within a new
process, freshly started by a supervisor, rather than blindly trying to rescue
all possible error cases without the full context of when and how they can
happen.

More:
- https://hexdocs.pm/elixir/1.17.3/try-catch-and-rescue.html#fail-fast-let-it-crash

## `nil`

```bash
iex> nil[:some_key]
nil
```

# Acronyms

- BEAM - Bogdan's Erlang Abstract Machine
- OTP - Open Telecom Platform
- ETS - Erlang Term Storage
- ERTS - Erlang Runtime System Application

# Dictionary

- abstract - wyodrębniać, wydobywać, wychwycać
- abstract sth from sth - wyodrębniać coś z czegoś
- common - powszechne / wspólne
- generic - ogólny
- inheritance - dziedziczenie
- provision - dostarczanie (czegoś), zaopatrywanie (w coś)
- specific - konkrektny
- suitability - stosowność, odpowiedniość, nadawanie się (do czegoś)
