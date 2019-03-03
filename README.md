# Freedom from make, circleci, and dockerfile

mobash is an incremental rewiring tool that lets you cut tight linkages
to bad syntaxes and vendor lockin. It helps you develop and maintain low
maintenance high clarity bash shell function libraries while still
providing continuous compatibility with legacy make and CircleCI systems.
In this way it represents a credible incremental refactoring path for
any project to move away from strong dependence on technologies with
high liability footprints and developer learning burdens.

The main motivation behind this tool is to push back against bad variations
on shell script that lead to vendor or tool lockin and reduce developer
learning efficiency. It takes years to learn just one classic Unix shell
script language like bash alone and so it is not reasonable to spread our
learning out to become experts in minor variations that provide little help
for lots of work.

## Structured Modular Bash shell scripting using mobash

Shell scripts are popular in Unix style operating systems for many decades.
Unfortunately, so far no simple system for organizing shell functions into
libraries has yet become popular. Modern languages such as ES6, Python, and
Golang support modules that correspond to subdirectories or files within
a filesystem hierarchy. Mobash is a simple standard bash shell script header
of only a few lines (compressed) that can provide function library module
capabilities to help you organize and maintain your bash shell scripts with
less effort and higher quality.

## Why would I want to use structured programming with bash shell scripts?

Many tools are based on small modifications of shell syntax. For example,
`Makefile` from the `make` utility is a popular example for the last few
decades. Another popular example of a shell script like syntax is `Dockerfile`
from `docker`. A third example is `.circleci/config.yml` from the CircleCI
paid cloud testing provider.  In all of these cases, it's typically necessary
to use shell scripts and `bash` is a popular choice but there is a problem:
each of these variations is deficient compared to pure `bash` shell functions.

## Performance

When a bash shell function calls another shell function, there is no `fork`
and `exec` overhead. Neither is there shell environment startup cost.

When a `make` action calls another `make` target, it's done using the `make`
command and necessitates a fork and exec followed by another shell. A `make`
rule can efficiently call another `make` rule without an additional spawn of
the `make` command itself, but it still needs to spawn another shell to
execute any associated actions. Therefore, `make` targets are slower than
appropriately guarded shell functions.

## Developer experience

`make` provides a reasonable developer experience because at least a local
dev environment can invoke targets naturally from an interactive shell for
debugging. The problems come when we look into the `make` syntax to discover
its reliance on tabs and unfortunately limiting shell syntax. It is bash like
but requires double dollar signs, has different parameter expansion rules and
syntax, and a non-uniform calling mode. If you want to call a make target
from within a make rule line, everything seems fine. But if you call a make
target from within an action, you must precede the call with `make`. If
you want to call a `make` target from another `Makefile` from within a rule,
it can be done slowly using `make -C` but causes a loss of loose linkage
because one must know from which subdirectory the function comes defined.

When a CircleCI step is defined inline, it becomes impossible for developers
to call in a local development environment. The arbitrary divergence from
standard bash syntax means that to the degree the CircleCI steps or commands
do more than simply delegate to shell functions that are not inline is the
degree that the CircleCI testing environment is unique, non portable, and
non reproducible. This leads to greater maintenance costs as different
versions of the environment need to be made to support CircleCI instead of
the developer local or production modes even when the functionality is
otherwise the same. While it is easy to call a bash shell function from
another function, it is impossible to call a CircleCI step from a shell
function or a remote machine or from a developer interested in debugging.

## Proposed solution

The `mobash` utility solves two problems. It generates short boilerplate
top-level executable starter scripts that are meant to be modified by adding
lines. The scripts have two properties:

First, like `make`, it stabilizes the starting working directory to be the same
as the initially invoked script source directory. This means it doesn't matter
what directory was current before the script started.

The second feature is that it can scan upward through parent subdirectories
looking for files called `moba.sh` and sources them in order (using a stack)
from root to leaf enclosed directory. In this way a naturally organized
library of functions and environment variable or other settings is enabled
by simply defining them in `moba.sh` files in each subdirectory. The
`moba.sh` file corresponds to the `index.js` file in Javascript to define
an ES6 module.  The `moba.sh` file can define a few simple functions inline
but for larger modules it should just `source` other files in the same
directory. Since the working directory is guaranteed to be the same as
where the `moba.sh` is placed it is the simplest possible syntax to source
the other files in the directory to define the functions according to
natural groupings that are easier for developers to understand.

It is important to notice that while `moba.sh` are `source'd`, the top-level
shell scripts generated by `mobash` actually use the functions so can be
thought of as controllers, use-cases, work-flows, or configurations that
define work flows as sequences of function calls.

## Advantages

By using this utility to generate boilerplate bash shell scripts that suppport
convenient function sharing it becomes possible to use CircleCI and `make`
without sacrificing the DRY (Do Not Repeat Yourself) principle. You can
still use CircleCI steps but you must first checkout or clone the source code
and then only put simple one-liners that just invoke bash with the name of
a bash function with some parameters and environment variables. By making
sure that all your CircleCI lines are just 1-liners and simple it becomes
a pure delegation pattern and then you can access the same underlying
shell script functionality via CircleCI or from a bash shell in a developer
environment locally for testing and debugging.

The advantages do not stop there. Once you have moved the body of your
actions from CircleCI and Makefile into shell scripts, you can then have
the same functions callable from both top-level `.PHONY` Makefile targets
as well as CircleCI steps. Then you can run those shell functions anywhere
you want and are no longer locked into a strange non-standard syntax. Keeping
the Makefile and CircleCI actions down to just 1-line delegations ensures
that your dependence and lockin to old or paid-only lockin technologies is
minimized and your learning leads to more flexibility and freedom for you
in your code.

## Example

Clone this repository and try running the `./example_subdir/use_func` script.
It should print a few lines that show a couple different working directories
and a couple different functions being called, with one calling the other.

Those functions are defined in `moba.sh` in two places. The `echo` is only
meant to clarify the order of execution from root to leaf but in a real
system it is better to have the `moba.sh` be totally silent and only define
functions and environment variables or other environmental features such as
aliases.

Inside the example `dofun` script we show an example call of the `r_inside`
function. The `r_inside` function is defined within `example_subdir` but calls
another function called `outer_func` that is defined in the enclosing parent
dir `moba.sh`

```
~/src/mobash$ ./example_subdir/use_func 
in /home/pbs/src/mobash
in /home/pbs/src/mobash/example_subdir
in the r_inside func start here
pretty good, just testing function calls across files
in the r_inside func end   here
~/src/mobash$ 
```

## Getting Started
Once you are ready to start trying to use it yourself, you can just copy
and modify the `use_func` top-level shell script by keeping the top part and
modifying below the comment line. Or you can run the `mobash` script with a
filename to regenerate the header by itself. Simply put bash function
definitions in the `moba.sh` accoring to whatever directory tree makes
sense, then define one or more top-level executor scripts using the
compressed header. Use the available function libraries to eliminate
repeated code and allow for better portability and invocability across
a wider, more modern, less constrained and more convenient array of
environments.

In `docker`, `make`, or `CircleCI`, it is suggested to either run the
script directly with a command such as
```
./use_func
```

or if you are on a `no_exec` filesystem then use `bash` as a prefix word:

```
bash use_func
```

You may need to provide additional positional parameters or preceding
environmental parameters. Using this thin delegation means developers can
easily compose and invoke the shell functions themselves and therefore
read and debug them easier locally before they reach the repository.
