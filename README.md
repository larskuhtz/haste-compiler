Haste
=====

A compiler to generate Javascript code from Haskell.

It even has a [website](http://haste-lang.org) and a
[mailing list](https://groups.google.com/d/forum/haste-compiler).

Features
--------

* Seamless, type-safe single program framework for client-server communication
* Support for modern web technologies such as WebSockets, WebStorage and Canvas
* Simple Javascript interoperability
* Generates small, fast programs
* Supports all GHC extensions except Template Haskell
* Uses standard Haskell libraries
* Cabal integration
* Simple, one-step build; no need for error prone Rube Goldberg machines of
  Vagrant, VirtualBox, GHC sources and other black magic
* Concurrency and MVars with Haste.Concurrent
* Unboxed arrays, ByteArrays, StableNames and other low level features
* Low-level DOM base library
* Easy integration with Google's Closure compiler
* Works on Windows, GNU/Linux and Mac OS X


Installation
------------

You have two options for getting Haste: installing from Hackage or from
Github. In both cases, you need to add add Cabal's bin directory, usually
`~/.cabal/bin`, to your `$PATH` if you haven't already done so.

Then, installing the latest stable-ish version from cabal is easy:

    $ cabal install haste-compiler
    $ haste-boot

Building from Github source is equally easy. After checking out the source,
`cd` to the source tree and run:

    $ cabal install
    $ haste-boot --force --local

You should probably run the test suite first though, to verify that everything
is working. To do that, execute `./runtests.sh` in the Haste root directory.
You may also run only a particular test by executing `./runtests.sh NameOfTest`.
The test suite uses the `nodejs` interpreter by default, but this may be
modified by setting the `JS` environment variable as such:
`JS=other-js-interpreter ./runtests.sh`. Other JavaScript interpreters may or
may not work.

Haste has been tested to work on Windows and OSX platforms, but is primarily
developed on GNU/Linux. As such, running on a GNU/Linux platform will likely
get you less bugs.


Portable installation
---------------------

It is possible to install Haste along with its runtime system into a portable
directory. Each user still has their own package database, which makes this
handy for global installations. To do this, check out the source and run:

    $ cabal configure -f portable-compiler
    $ cabal build


Usage
-----

To compile your Haskell program to a Javascript blob ready to be included in an
HTML document or run using a command line interpreter:

    $ hastec myprog.hs

This is equivalent to calling ghc --make myprog.hs; Main.main will be called
as soon as the JS blob has finished loading.

You can pass the same flags to hastec as you'd normally pass to GHC:

    $ hastec -O2 -fglasgow-exts myprog.hs

Haste also has its own set of command line arguments. Invoke it with --help to
read more about them. In particular --opt-all, --opt-google-closure and
--with-js should be fairly interesting.

If you want your package to compile with both Haste and, say, GHC, you might
want to use the CPP extension for conditional compilation. Haste defines the
preprocessor symbol `__HASTE__` in all modules it compiles.

Haste also comes with wrappers for cabal and ghc-pkg, named haste-inst and
haste-pkg respectively. You can use them to install packages just as you would
with vanilla GHC and cabal:

    $ haste-inst install mtl

This will only work for libraries, however, as installing Javascript
"executables" on your system doesn't make much sense. You can still use
`haste-inst build` to build your "executables" locally, however.

Finally, you can interact with Javascript code using the FFI. See
`doc/js-externals.txt` for more information about that.

For more information on how Haste works, see
[the Haste Report](http://haste-lang.org/hastereport.pdf "Haste Report"),
though beware that parts of Haste may have changed quite a bit.

You should also have a look at the documentation and/or source code for
`haste-lib`, which resides in the `libraries/haste-lib` directory, and the
small programs in the `examples` directory, to get started.


Interfacing with Javascript
---------------------------

When writing programs you will probably want to use some native Javascript
in your program; bindings to native libraries, for instance. There are two ways
of doing this. You can either use the GHC FFI as described in
`doc/js-externals.txt`, or you can use the Fay-like `ffi` function:

    addTwo :: Int -> Int -> IO Int
    addTwo = ffi "(function(x, y) {return x + y;})"

The `ffi` function is a little bit safer than the GHC FFI in that it enforces
some type invariants on values returned from JS, and is more convenient. It is,
however, quite a bit slower due to its dynamic nature.

If you do not feel comfortable throwing out your entire legacy Javascript
code base, you can export selected functions from your Haste program and call
them from Javascript:

fun.hs:

    import Haste.Foreign
    
    fun :: Int -> String -> IO String
    fun n s = return $ "The number is " ++ show n ++ " and the string is " ++ s
    
    main = do
      export "fun" fun

fun.js:

    function mymain() {
      console.log(Haste.fun(42, "hello"));
    }

...then compile with:

    $ hastec '--start=%%(); mymain();' --with-js=fun.js fun.hs

`fun.hs` will export the function `fun` when its `main` function is run.
Our Javascript obviously needs to run after that, so we create our "real" main
function in `fun.js`. Finally, we tell the compiler to start the program by
first executing Haste's `main` function (the `%%` gets replaced by whatever
name the compiler chooses for the Haste `main`) and then executing our own
`mymain`.


Effortless type-safe client-server communication
------------------------------------------------

Using the framework from the `Haste.App` module hierarchy, you can easily write
web applications that communicate with a server without having to write a
single line of AJAX/WebSockets/whatever. Best of all: it's completely type
safe.

In essence, you write your web application as a single program - no more forced
separation of your client and server code. You then compile your program once
using Haste and once using GHC, and the two compilers will magically generate
client and server code respectively.

You will need to have the same libraries installed with both Haste and vanilla
GHC (unless you use conditional compilation to get around this).
`haste-compiler` comes bundled with all of `fursuit` and `haste-lib`, so you
only need to concern yourself with this if you're using third party libraries.

Examples of Haste.App in action is available in `examples/haste-app` and
`examples/chatbox`.

For more information about how exactly this works, see this
[draft paper](http://haste-lang.org/icfp14.pdf).


Base library and documentation
------------------------------

You can build your own set of docs for haste-lib and fursuit by running
`cabal haddock` in the Haste base directory as with any other package.

Or you could just look at
[the online docs](http://hackage.haskell.org/package/haste-compiler).


Reactive web EDSL
-----------------

Haste comes with a basic, environment for writing client side web applications
in a reactive fashion. See Fursuit for more information.

As the reactive library relies heavily on Applicative, you may find the idiom
brackets of the Strathclyde Haskell Enhancement
(https://personal.cis.strath.ac.uk/~conor/pub/she/) quite useful.


A note about security
---------------------

As described in https://github.com/haskell/cabal/issues/936,
Cabal is not entirely secure, and as haste-boot uses Cabal this obviously extends
to Haste as well. If this troubles you, you can take the following steps in order
to obtain a trusted Haste installation:

* Install Haste from GitHub (don't forget to use HTTPS!) and run `haste-boot`
  as usual.
* Install `deepseq`, `containers`, `monads-tf` and `transformers` from a source
  you trust, in that order, forcing reinstalls as necessary.
* Manually reinstall `fursuit` and `haste-lib` from the same source tree you
  installed Haste from, in that order.

That said, if you're comfortable trusting random Internet people
(me, for instance), trusting Cabal shouldn't really be a big deal.


Libraries
---------

Haste is able to use standard Haskell libraries. However, some primitive
operations are still not implemented which means that any code making use 
of them will give you a compiler warning, then die at runtime with an angry
error. Some libraries also depend on external C code - if you wish to use such
a library, you will need to port the C bits to Javascript yourself (perhaps
using Escripten) and link them into your program using `--with-js`.


Why yet another Haskell to Javascript compiler?
-----------------------------------------------

Existing implementations either produce huge code, require a fair amount of
work to get going, or both. With Haste, the idea is to give you a drop-in
replacement for GHC that generates relatively lean code.


Known issues
------------

* Not all GHC primops are implemented; if you encounter an unimplemented
  primop, please report it together with a small test case that demonstrates
  the problem.
