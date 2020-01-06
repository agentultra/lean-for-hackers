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
