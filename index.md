# Lean 3 For Hackers #

This is a guide to getting started using
[Lean](https://leanprover.github.io) 3 to write programs.  If you're a
curious sort that likes to explore new programming languages for fun
this guide is for you.  We will cover installing and setting up Lean,
the standard libraries, how to write programs using Lean, and where to
learn more.

## Installation ##

The community Lean project has great documentation for [getting
started](https://leanprover-community.github.io/get_started.html) on
all major platforms.

Another great way to get started, if you're on a Unix-like system, is
[elan](https://github.com/Kha/elan).  This is like your _nvm_,
_ghcup_, _rustup_, _rvm_, etc.  It's a light-weight package manager.
It can install project-specific versions of Lean and manage global
Lean installations for you.  This is my preferred method.

The two major editors for working with Lean are [VS
Code](https://code.visualstudio.com/) and
[Emacs](https://www.gnu.org/software/emacs/).  Both have supported
plugins that work well and offer full completion, jump to definition,
error hints, etc.

If you don't use one of those editors, `lean --server` speaks JSON
over `stdin`.  If you can write a plugin for your editor be sure to
share it.

That's all you need to get started.

## Hello, World ##

Copy the following program into a text file and name it `hello.lean`:

``` lean
import system.io

open io

def main : io unit :=
    put_str "Hello, world!\n"
```

You run this as a program on the command line by opening up your
favourite terminal and running:

``` shell
$ lean --run hello.lean
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
```

Running this in a shell from your terminal does what you'd expect:

``` shell
$ lean --run greet.lean
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

At the top level our special main function returns an `io` action so
the Lean VM will go ahead and evaluate our program for us.

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
a cool Lean feature called a _command_, `#check`:

``` lean
#check (++)
```

A command is something you can type into your editor which we can use
to query the Lean system.  This command allows us to inspect the type
of any expression that comes after it.

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

When we run our program, Lean runs through each action one at a time
and we get our expected behaviour.

### Recapping the Introduction ###

In order to get our feet wet we hackers need to know three things:

1. How to read input, display output
1. How to define values
1. How to apply functions

We broke down the canonical `"Hello, world!"` example and demonstrated
how you can:

1. Print strings to the console and read input from the user with `io`
1. Define functions with `def`
1. How to call functions and sequence together `io` actions

We also learned about the `#check` command for querying Lean about
types.

The next thing we need to learn are more data types, structures, and
libraries we have available to construct more interesting and useful
programs.

## Guessing Game ##

Let's write a program that is a little more interactive.  An easy
program that we should be able to confidently write in a new language
after, "hello, world," is a guessing game.  Our program will pick a
number between 1 and a 100.  The user will have a finite number of
chances to guess the number.

We'll have to learn a few more things: *loops*, how to get a random
number, and checking to see if the user made the correct guess.

### Random Numbers ###

Lean's `io` system is spartan.  Thankfully we have ways to read files,
get input, write output, and also get input from child processes.  We
also have a random number generator but there's no way to get access
to random input to seed it with in Lean.

If you're on a unix-like system you'll likely have access to a device
that generates random bytes for you.  If you're on Windows you'll have
to come up with something clever.  We can use the `io.cmd` function to
run a shell command to gather some bytes for us:

``` lean
import system.io

open char
open list
open io

def rand_range (lo : nat) (hi : nat) : io nat := do
  r ← io.cmd { cmd := "head", args := ["-c 80", "/dev/urandom"] },
  let seed         := foldl (+) 0 $ map to_nat r.to_list,
  let gen          := mk_std_gen seed,
  let (target, _)  := rand_nat gen lo hi,
  return target
```

You may have to modify this for your platform of choice.  `io.cmd`
returns back a string buffer which we map to `nat` and sum to get our
number.  We use this to seed the `mk_std_gen` function which creates
our generator.  We can then generate numbers!

If you come across a function you're not familiar with or need to know
the type of something be sure to use the `#check` command to ask Lean
about it.  We've added `foldl` for summing up our list of numbers and
we've used `map` to turn our `string` into a `list nat`.  There's also
the `mk_std_gen` function and `rand_nat`.

If any of these functions are unfamiliar to you look up a tutorial on
`foldl` or `map` in Haskell or some other language as they're quite
common.

You can also use your editor's _jump to definition_ facility to browse
what's available.  Jump to the `io` module to browse it or right to
`io.cmd`.  This can be useful for finding things without having to
search Github.

Once we can generate a random number we can start our main function
like so:

``` lean
def main : io unit := do
  put_str "Guess a number between 1 and 100 (inclusive)",
  target ← rand_range 1 100,
  put_str $ to_string target
```

Run the program and test that it's generating a random number.

Next we'll add the prompt to user and check if they won.  Replace
`main` with this:

``` lean
def main : io unit := do
  put_str "Guess a number between 1 and 100 (inclusive)",
  target ← rand_range 1 100,
  put_str "Guess a number: ",
  response ← get_line,
  let guess := string.to_nat response,
  if guess = target
  then put_str "You win!\n"
  else put_str "You lose!\n"
```

And run it.  It's not a very good game yet but one step at a time.
Let's extract the guessing into an action we can call:

``` lean
def get_guess : io nat := do
  put_str "Guess a number: ",
  response ← get_line,
  return $ string.to_nat response
```

And re-write `main` again:

``` lean
def main : io unit := do
  put_str "Guess a number between 1 and 100 (inclusive)",
  target ← rand_range 1 100,
  guess ← get_guess,
  if guess = target
  then put_str "You win!\n"
  else put_str "You lose!\n"
```

Here we've added a conditional expression: `if`.  Since this is an
expression, it has a value, and you can return it from a function.  It
also means there's no `else if`.  It's much like Haskell and other
functional programming languages in this regard.

Our last requirement is that the user get a number of guesses.  This
should make it more fun and game like.  We have a handy function in
`io` called, `iterate`.  Go ahead and use the `#check` command to see
it's type.  Then replace `main` with this:

``` lean
def main : io unit := do
  put_str "Guess a number between 1 and 100 (inclusive)",
  target ← rand_range 1 100,
  iterate 20 $ λ i,1
    if i > 0 then do {
      put_str $ "You have " ++ (to_string i) ++ " guesses.\n",
      guess ← get_guess,
      if guess = target then do {
        put_str "You win!\n", return none
      } else do {
        if guess > target
        then put_str "Too high!\n"
        else put_str "Too low!\n",
        return $ some (i - 1)
      }
    }
    else
      return none,
 return ()
```

That's a lot more code but hopefully it is easy enough to see the big
picture.  `iterate` is a function that iterates from an initial value
it passes to a function we give it.  Our function which we start with
the `λ i,` has one job: return an optional `nat`.  We see that in our
function body with those lines: `return $ some (i - 1)` and `return
none`.

The former `return` we use to decrement `i` and in that case `iterate`
will call our function again with the new value.  The latter `return`
will cease iterating.

When we run this we can play the game.  Let's just clean up the code a
bit after a few rounds, first add this to the top of the file:

``` lean
open ordering
```

then:

``` lean
def main : io unit := do
  put_str "Guess a number between 1 and 100 (inclusive)",
  target ← rand_range 1 100,
  iterate 20 $ λ i,
    if i > 0 then do {
      put_str $ "You have " ++ (to_string i) ++ " guesses.\n",
      guess ← get_guess,
      match cmp guess target with
      | eq := do { put_str "You won!\n", return none }
      | gt := do { put_str "Too high!\n", return $ some (i - 1) }
      | lt := do { put_str "Too low!\n", return $ some (i - 1) }
      end
    }
    else
      return none,
 return ()
```

Like our `if` statement we have `match` which is also an expression.
This is how Lean can do pattern matching on _inductive_ types.
Instead of nesting all of those if expressions and do blocks we can
regain some of our indentation with our `match`.  You can [read
more](https://leanprover.github.io/reference/declarations.html#match-expressions)
about there in the Lean reference documentation.

### Conclusion ###

We made a guessing game!  It wasn't too bad.  We had to work around
Lean's limited `io` library but it worked!  We also learned how to use
the `iterate` monadic action.  And how to pattern match on inductive
structures.

Play around with this game code a bit.  Use that `#check` command and
jump to definitions.  Next we'll learn about how to define our own
data structures.

## Data Structures ##

Lean is a _dependently typed_ functional programming language.  If
you've heard of Idris, Agda, or Coq it's much like that.  What this
means is that types can depend on data.  Or that we no longer need a
separate language for types and terms as we do in say, Haskell.  A
classic example is the type of vectors of a particular length.  In
Haskell this is difficult to express in its type language because
_data_ lives in terms... the expressions.  It's possible to extend the
type system of Haskell to include such types but it is built-in to
Lean and quite natural... and more powerful.

Let's not jump in to the deep end yet.  We will start with defining
data structures that you might be more familiar with first.  We'll add
dependent types later as we progress.

### Structures and Objects ###

Most programmers are familiar with structures, records, etc.  Lean has
some interesting features for you.  This is how you define a structure
for _two-dimensional points_ on the natural numbers:

    structure nat_point :=
      mk :: (x : nat) (y : nat)

The basic structure of this definition is:

    structure <name> := <constructor> :: <fields>

In order to create a point we need a constructor function.  We define
the constructor function in our definition.  Each of the fields is a
parameter to this function:

    #check point.mk 4 8

That's not the only function on `point` that Lean will generate for
us.  You will also get functions for accessing fields, a `sizeof`
function, and others which may come in handy later on.  To see them
all, try this command:

    #print prefix nat_point

If you want to be able to parameterize the type of `point` so that
users can create points with their own types we can add a `type
parameter` to our definition:

    structure point (a : Type) :=
      mk :: (x : a) (y : a)

Now the type of the `x` and `y` fields can be inferred by the types of
the values given to the constructor function (as long as they're of
the same type):

    #check point.mk 3 4

    def dx : int := -12
    def dy : int := 22

    #check point.mk dx dy

Once you have a point you can access its fields:

    def p : point nat := point 3 4

    #eval point.x p
    #eval point.y p

We can then write functions using our new type like so:

    def add (p q : point nat) :=
      point.mk ((point.x p) + (point.x q)) ((point.y p) + (point.y q))

That's a bit noisy so thankfully Lean will allow us to abbreviate our
accessor function on our type parameter:

    def add (p q : point nat) := point.mk (p.x + q.x) (p.y + q.y)

Much better.  What Lean is doing when it sees `p.x` or `q.x` is
inserting the `p` (the object to the left of the dot) as the implicit
first argument of the function `x` (the object named on the right side
of the dot) on the same type as `p`, which is in this case is `point`.
So `p.x` in this expression is the same as `point.x p`.  Since Lean
knows the type of `p` and the functions available on it we can shorten
it down a bit.

#### Objects ####

Next we have _objects_.  They're basically like anonymous constructors
for structures.  This is useful because so far when we use the
constructor of our structure we've had to pass parameters by position.
Objects allow us to pass them in by _name_ instead like so:

    let p1 : point nat := { y := 20, x := 10 }

For constructing large structures this can be the much more readable
option.  You can also do this anonymously like so:

    #check { point . y := 10, x := 20 }

We pass the name of the constructor along with one of the fields and
the rest of the structure is inferred.

#### Inheritance ####

One neat feature of structures is that you can _inherit_ new
structures from them.  This is similar to languages Common Lisp that
have similar features:

    inductive color
    | red
    | green
    | blue

    structure color_point (a : Type) extends point a :=
      mk :: (c : color)

And now we have a structure `color_point` that inherits the fields of
`point` and adds one of its own.  We construct one by passing in a
structure of the parent type and our fields:

    -- recall p1 : point nat
    #check color_point.mk p1 green

Nice!
