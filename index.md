# Lean 3 For Hackers #

This is a guide to getting started using
[Lean](https://leanprover.github.io) 3 to write programs.  If you're a
curious sort that likes to explore new programming languages for fun
this guide is for you.  We will cover installing and setting up Lean,
the standard libraries, how to write programs using Lean, and where to
learn more.

## Installation ##

The best version of Lean available is the community fork at
<https://github.com/leanprover-community/lean>.  You can get binary
packages from the releases page or build from source.

I recommend building from source.  Make sure to add the built binaries
to your path and you're off to the races.

The two major editors for working with Lean are [VS
Code](https://code.visualstudio.com/) and
[Emacs](https://www.gnu.org/software/emacs/).  Both have supported
plugins that work well and offer full completion, jump to definition,
error hints, etc.

If you don't use one of those editors, Lean speaks
[LSP](https://microsoft.github.io/language-server-protocol/).  If you
can write a plugin for your editor be sure to share it.

That's all you need to get started.

## Hello, World ##

Copy the following program into a text file and name it `hello.lean`:

``` lean
import system.io

open io

def hello_world : io unit :=
    put_str "Hello, world!\n"

#eval hello_world
```

Lean is awesome and you should be able to see the result of the
`#eval` line in your editor.  This is because Lean is built to run and
evaluate programs interactively.  However you can also run this as a
program on the command line by opening up your favourite terminal and
running:

``` shell
$ lean hello.lean
Hello, world!
```

If you have installed everything right you should see the above
output.  If not check that you have the `lean` binary, at least, in
your system's path and that you're using Lean 3.

## Hello, ${name}! ##

Once we have our obligatory greeting out of the way the next step is
to ask the user to input their name and greet them properly.  Modify
`hello.lean` or paste the following program into a new file:

``` lean
import system.io

open io

def greet (s : string) : io unit :=
  put_str $ "Hello, " ++ s ++ "!\n"

def main : io unit := do
  put_str "What is your name? ",
  name ← get_line,
  greet name

#eval main
```

You may notice that you can no longer see the result of the `#eval`
line in your editor.  That's because the `get_line` function awaits
input from the terminal so there's nothing it can sensibly display.
However running this in a shell from your terminal does what you'd
expect:

``` shell
$ lean greet.lean
What is your name? Random J. Hacker
Hello, Random J. Hacker!
```

## Introducing Lean ##

Lean is a strictly-evaluated, pure, dependently-typed programming
language.  It has a powerful framework for proving mathematical
theorems built-in.  And libraries of theorems to get you started.

The primary focus of Lean is mathematical research and its intended
audience are mathematicians.  All of the documentation and tutorials
are built around this purpose and audience.

This guide fills the missing gap for the programmers.  If you've tried
Agda or Idris you will find Lean to be familiar.  If you're more
comfortable with Haskell, F#, Scala, or OCaml you will be able to sink
right into Lean.  This section provides an overview to get you up to
speed or familiarize you with the fundamentals of Lean you need to
know to write your first programs with it.

### Hello World, Revisited ###

The first things a hacker needs to know to start messing around with a
new language are:

1. How to read input, display output
1. How to declare values
1. How to apply functions

The first thing we do in `hello.lean` is import `system.io`.  `import`
is a keyword that includes a Lean module.  Modules in Lean contain
declarations.  This module includes a basic library for doing common
things with input and output such as reading files and writing to the
terminal.

You can declare something called a `namespace`. We `open` one on the
next line.  This handy feature prevents the common problem with module
systems such as Haskell's where declarations are imported into the
module's global name space by default.  This line adds the
declarations in the `io` name space to our module's global name space.

If we omit the `open io` line then we'd have to prefix our use of
`put_str` with the name space: `io.put_str`.  Go ahead and modify
`hello.lean` to see what I mean.

As the Python hackers say, _explicit is better than implicit_.

Next we have our first definition:

``` lean
def hello_world : io unit :=
put_str "Hello, world!\n"
```

As you may have guessed, `def` is our keyword for defining values.
Lean is a dependently typed functional programming language and so
functions are also values as much as strings and integers and anything
else you can define.

The colon can be read as, "has type."  This is the same syntax as
Purescript and is equivalent to Haskell's `::`. This definition has
type, `io unit` which is the same as Haskell's, `IO ()`.

Lean took a note out of the Haskell playbook and uses the _Monad_
abstraction to model computations that perform input and output.  If
you're not familiar with IO in Haskell it's enough to know that if you
want to read files or print things to the console your function will
return `io something` where `something` is the return type of the
computation.  In our example it's `unit`, a type with a single
inhabitant... sort of like `null`.  Since our "IO" _action_ doesn't
return anything, it merely prints to the console, we return `io unit`.

Finally we see the `#eval hello_world` line.  This is a _command_ in
Lean.  This command _evaluates_ a Lean expression.

Commands are most often used to _query_ Lean for information.  If
you're familiar with languages that have a REPL you might do this sort
of thing using that.  In Haskell this would look like, `:type
someFunc`.  Lean markets itself as an _interactive_ theorem prover and
it let's you do this in your source buffer without having to leave and
switch to a REPL.

Lean doesn't have any concept of an implicit entry-point into our
program so we explicitly use the `#eval` command to evaluate our
`hello_world` function.  This function evaluates to an IO action so
the Lean VM executes it for us and writes our greeting to the
terminal.

### Greetings, Revisited ###

The next thing we do is get input from the user and display a
personalized greeting.  We need to get input from the standard input
stream, ie: the user. And we need to print our greeting back out.

Let's talk about our `greet` function:

``` lean
def greet (s : string) : io unit :=
  put_str $ "Hello, " ++ s ++ "!\n"
```

This is a handy notation for defining a function of a fixed arity.
This function takes one parameter we name `s`.  It has the type of
`string`.  If we want to define a function with more parameters we
simply add another pair of parentheses before the colon separated by a
space:

``` lean
def example (s : string) (p : int) : io unit := ...
```

We also introduce a new function, `++`.  We use an infix _operator_
notation for it.  It's the `append` function.  You can tell this using
another handy command, `#check`:

``` lean
#check (++)
```

Just like in Haskell we have to put parentheses around operators.  If
your editor is able to talk to the Lean server this should pop up with
the type of `++` which is `append : ?M_1 → ?M_1 → ?M_1`.  The `?M_1`
is Lean's placeholder for a _polymorphic type variable_.  The arrows
are just like `->` in Haskell and other functional programming
languages and represent function types.  Since `(++)` is defined for
the `string` type we can use it to append the parts of our message
together.

Next we change the `main` function of our program.  Here we see that
Lean has taken another play from Haskell's book: the _do_ notation:

``` lean
def main : io unit := do
  put_str "What is your name? ",
  name ← get_line,
  greet name
```

Just as in Haskell this notation is syntactic sugar for chaining
together _bind_ operations in a _monadic context_.  That basically
means that we can extract values from functions that return `io
something` and use them to compute other things including other
functions returning other `io somethings`.  We're defining two such
actions and we use them here, one per line.

Notice the commas at the end of each line but the last in our function
definition.  These are sort of like semi-colons in a C-like imperative
language.  They make sure the indentation is not important unlike the
Haskell version.

There's also the left-arrow on the second line.  You can type that if
you're using VS Code or Emacs with the Lean plugin by typing `\l` in
the buffer and pressing space.  If you cannot, `<-` will also work.

Lean makes use of unicode symbols that mathematicians are used to.  So
these plugins make it easy to enter these symbols using a clever short
hand and some keyboard shortcuts.  They all start with `\` and some
letters.  Editor support varies but in Emacs it does a prefix search
on the letters you type and you can navigate the symbols with similar
names using shortcuts to get the one you mean.  This has a practical
reason beyond making mathematicians happy which we'll get into when we
start discussing the type system in more detail.

The left arrow in a `do` block tells Lean to resolve the computation
on the right-hand side of the arrow in our `io` context and bind it's
result to the identifier on the left side of the arrow.  The _action_
`get_line` awaits the user to input some characters into a buffer and
returns that buffer to us as a `string`.

Finally we evaluate our `greet` function in the last step which
returns an `io unit`, that matches our type for the definition of
`main` and thus is a valid final action for the block and will be the
type of our expression.

When we `#eval` this function and run our program, Lean runs through
each action one at a time and we get our expected behaviour.
