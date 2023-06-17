# Adventures in the Land of Lisp

I've been experimenting with lisps for about 6 month now, clojure, common lisp,
emacs lisp, and a little racket. I recently finished my biggest code so far,
solving all the [Protohackers](https://protohackers.com/) challenges in [Common
Lisp](https://github.com/bo-tato/protohackers-cl), so I thought I'd share my
thoughts. I also contributed a little to "real software" with bug fixes and
small features in projects but for the most part I don't have any real
experience yet, this is just my first impressions, not the words of someone
experienced in these languages. The TLDR is: don't be scared off by the
parenthesis, and don't assume languages are better because they're more
"modern". 
If you're tired of discussion of old topics (parenthesis, editors, languages),
then skip ahead to the protohackers section for dissection of some tricky
concurrency bugs.

## (Parenthesis)

Why dedicate the first section to a superficial discusion of syntax? Because I'm
superficial. I like learning different languages, and I like dynamically typed
languages and languages that make functional programming ergonomic but don't
strictly enforce it. That sounds like a perfect fit for lisp, but I'd never
taken a serious look at it until now, 90% because I thought all the parenthesis
were ugly (the other 10% because I thought lisp is a functional language, and
I'd already done some elixir, ocaml, and haskell, so I thought I wouldn't really
be learning much new ideas, which was another big misconception).

It really is just a question of what we have most exposure to, english speakers
will think arabic or hebrew are "unreadable" with their different alphabet and
text right-to-left. They will think english is "unreadable" at first. Lisp is
visually different enough from other languages to be harder to read at first, if
we have many years experience with them and none with lisp. My progression has been more or less:  
Starting out: this is ugly and harder to read  
After a few weeks: this is not so bad after all  
After a few months: maybe I'm actually coming to prefer this

It seems to happen to everyone, and is said the be the reason no alternative
syntax has taken off: by the time programmers have enough experience with lisp
to design an alternative syntax based on whitespace or something rather than
parenthesis, they no longer want to. Is it just lisper's stockholm syndrome? It
is objectively a simple and consistent syntax and there are various benefits
that come from that:

1. Macros

How many rust programers are comfortable writing procedural macros? How many
ocaml programmers are comfortable with ppx or haskellers with template haskell?
All lisp programmers are comfortable writing macros, because it is almost like
writing any ordinary function, as the AST you are manipulating is simple and is
the code you see on the page.

2. Structural Editing

Lisp has this since forever with paredit, and I thought treesitter would bring
this to other languages. I tried adding some paredit style keybindings for
python based on treesitter and now I no longer think so. For the same reason
macros are more natural in lisp (the AST is the code you see on the page), I
think structural editing will always be more natural in lisp.

3. Extensibility

As code and data are both just lists, it is easy to embed one in the other, and
to add new constructs to the language. For example for protohackers I used the
[lisp-binary](https://github.com/j3pic/lisp-binary) library which provides a
declarative DSL for specifying binary formats, and uses macros to turn that into
efficient code to parse and write it. The equivalent for common languages is
Kaitai Struct, which uses code generation instead of macros, and embeds their
own [expression
language](https://doc.kaitai.io/user_guide.html#_expression_language) to handle
parsing complex formats, whereas lisp-binary just embeds lisp. With lisp
implementing such a library from scratch is [one
chapter](https://gigamonkeys.com/book/practical-parsing-binary-files.html) of an
introductory programming book. For other languages it is a much more complex
task.

Another example is GUIX, which is still in development and not really ready to
replace all DevOps yet but has the potential to replace what is currently done
as shell scripts in yaml lists, a mix of templating languages that each have
their own syntax for conditionals and loops and inheritance, and a long list of
DSL's that slowly accrue the features of general purpose programming languages,
with just guile scheme.

It seems the lisp approach of using the same language and extending it for
different purposes, but with a consistent syntax, with a full general purpose
language available, and without having to reimplement syntax highlighting,
debuggers, compilers and the rest of the tooling for each bespoke language, is a
much more sane approach than the current mess of config languages, templating
formats and DSLs.

## Emacs

Like parenthesis, editors are another topic already discussed to death, but I
think as the largest and longest-running open source lisp project, and the best
free development environment for lisps, it is worth a mention. Before I thought
I already know an editor well (vim) that I've been using forever, and I thought
the people reading email and doing everything inside emacs must be crazy, why
would you want your editor to do all that? And why do would they spend so much
time tweaking their emacs config?

I haven't customized it much, I mostly just use the defaults of [Doom
Emacs](https://github.com/doomemacs/doomemacs) and they work well, but I now
understand the people who do. They're not just editing a config file, they're
coding lisp and extending their environment. Presumably you also enjoy coding as
a hobby or you wouldn't be reading these ramblings. And I'm not reading email in
emacs but I understand those who do. I got two huge productivity gains from
using emacs:
1. using magit replaced my use of git cli
2. org mode replaced my loosely organized plaintext files for note taking, todo
   lists, and calendar

For email I already have a good setup I know so I just keep using it but if I
didn't then it'd make sense to do in emacs also. Emacs isn't an editor but a
lisp runtime and platform for text-oriented applications, one of which is a
great implementation of vim (evil-mode). For any given task (git interface, note
taking software, todo list, text editor, etc) there is probably some more
polished software than emacs, but in the same way lisp can unite various DSLs in
a single host language, emacs unites many different programs into sharing the
same UI patterns, the same configuration and extension language, and being able
to easily interoperate with each other. I don't have time to become a power user
of the best tool for everything, but I do have enough time to learn just emacs
and get a decent tool for everything.

It is also a good example I think of the productivity of lisp that people talk
about but also say has never been demonstrated. Despite all the warts of emacs'
lisp dialect (slow, lack of any namespacing, no good way to write asynchronous
code besides callbacks, no lexical scoping until relatively recently), it has
managed, with 10% of the users vim has, and a tiny fraction compared to VSCode
and the rest, to produce a code editor and a whole set of text-oriented
applications with an impressive depth of functionality.

## Clojure

I liked the language and would pick it over the others for anything where
access to JVM libraries is important, or anything to do with web development in
general where it seems to have a lot more mature ecosystem than Common Lisp (CL)
or racket. But there are already endless comments on how CL is old and crufty,
and clojure or racket are newer better lisps so when I decided to learn a lisp I
thought I should just look at clojure and not pay attention to CL. So to balance
it out some here I'll just write some disadvantages I found. Again I quite
liked clojure but there is already plenty of writing on how clojure is a nicer
cleaned up lisp, so I'll just write some things I haven't really seen discussion
of. Or well I'm sure these are known to people that use the language, but from
the perspective of me 6 months ago or anyone else interested in learning some
lisp for the first time these tradeoffs will be unknown.

1. It feels very "loose"

Everything is super generic and there is not much error checking. For example in
one protohackers challenge I was first using an `agent` and then realized I'd
better use an `atom`, so I had to change all uses of `send` to `swap!` but I
missed one. If you call `send` on an atom it doesn't give any error, it just
silently doesn't send anything. It didn't take me too long to find the cause as
it was a quite small program but I imagine in large software such things could
be a real pain. When destructuring in clojure if you have too few values the
extra variables get nil, you can also give too many. In racket or CL or python
that would raise an error. In clojure you don't use structs or classes, just
maps, and accessing a key that doesn't exist is nil, while in CL accessing an
unbound slot (an uninitialized field of a class) raises an error. It seems often
the case that functions in clojure, when passed input or types they don't
expect, will simply return nil or sometimes even produce a garbage answer,
rather than raising an error closer to the source of the problem. Also CL for
example with SBCL provides a level of compile time warnings comparable to mypy
and flake8 with python. Clojure has clj-kondo that warns of some things but it
isn't really comparable.

2. Startup Time

A lot of my coding is scripts and CLI utilities. Clojure on the JVM has quite a
big startup time. One option is graalvm native compilation, but it doesn't work
or is hard to get working with many libraries, and is slow to compile.
[Babashka](https://babashka.org/) offers an easier option for quick scripting,
but again you lose access to a lot of libraries, and to a lot of clojure's
development tooling. It's a quite new project though and cider's nrepl
middleware and other tools could surely be made compatible with babashka, they
just haven't yet. So the situation with babashka and graalvm will both improve
in the future.

## Racket

Again like clojure it seems a great language, but I had heard that for learning
lisp nowadays racket or clojure are more modern and better so I didn't consider
CL at first. So I'll just list some ways it wasn't better for me.

1. Startup Time

A lot of my coding is scripts and CLI utilities, racket has enough of a startup
time to be annoying for that. There is a minimal dialect
[Zuo](https://docs.racket-lang.org/zuo/index.html) with instant startup, but
with CL with SBCL I can have compiled binaries with instant startup and use the
full language and all of it's libraries.

2. Lack of interactive development

Racket breaks from the lisp tradition in having a level of interactive
development worse than ipython with autoreload enabled. CL has many details that
went into it's design and implementation to make it possible to develop large
programs without restarting the program and losing it's state, which is not the
case for the REPL's of racket, python, and most other languages. For example
what happens to existing instances of a class or struct when you add or remove
some field to it's definition, or what happens if you passed a handler function
as a parameter to some tcp or http server, and then update that function. Or when
you get an error does it crash and stop the whole program or do you enter a
debugger and have the option to edit and reload code and then continue? In terms
of productivity I think interactive development does make a real difference for
large complex software with a lot of state like emacs, but for most stuff I work
on the REPL of python or racket would be plenty enough. But the interactivity
CL provides is more fun to me.

3. Tooling and ecosystem

Emacs with Sly or Slime for CL, or CIDER for clojure, seems a much nicer
development experience than racket-mode or Dr. racket. And while they are all
niche languages with tiny ecosystems, clojure and CL have been used much
more in production software and consequently have more mature libraries.

## Protohackers

I found it fun and a great set of problems for learning a new language,
especially if you sometimes do lower-level networking and binary parsing. You
get exposed to some real libraries (json, networking, concurrency, parsing), and
the problems are complex enough that you run into the trickier areas of the
language like concurrency bugs.

### Bugs

Maybe the most interesting part is the tricky bugs and what could have avoided
them. I think it's interesting that none of the non-trivial bugs I ran into
would be helped by static type-checking.

1. concurrency bugs in gloss (binary parsing library for clojure)

The first bug I ran into was that gloss was using a thread-unsafe java type
(ByteBuffer), which had already been noticed and
[fixed](https://github.com/clj-commons/gloss/commit/bbe57eae3a27d6c2d8083c04219652f97601b7d4),
just it hadn't made it into the latest published release yet. The second bug
which was the real tricky one had thankfully [already been
identified](https://github.com/clj-commons/gloss/pull/53), since as a new
clojure programmer I might never have found the cause in a reasonable amount of
time otherwise, but fixing it turned out to be quite tricky. Gloss has a
`decode-stream` function that takes a parser, and a
[manifold](https://github.com/clj-commons/manifold "manifold") stream of bytes,
and returns a manifold stream of parsed objects:
``` clojure
    (s/connect-via src f dst {:downstream? false})
    (s/on-drained src #(do (f []) (s/close! dst)))
```
    
src is the src stream of bytebuffers, dst is the output stream of parsed
objects, and f is the decoding function. The decoder needs to know when the
input is over, as for example you could have a parser that accepts the string
"abc" or "abcdef". After you call it with "abc" it can't return "abc" right
away, but needs to wait to see if it's followed by "def", but if the stream ends
at "abc" it should return "abc". That's what the `on-drained` callback with `f
[]` does, flush out partial results like "abc" when the input is done. But it
turns out this on-drained callback can actually run before f has finished
decoding the last element of src. My first idea to fix it was to simply append
`[]` to src, now f will be called on all elements of src and then `[]`
sequentially, we eliminate that unneeded concurrency, and eliminate the bug. But
this lead to [very confusing test
failures](https://github.com/clj-commons/gloss/issues/62#issuecomment-1462551568)
that only failed on the ci setup. I'm still not sure why it only failed on the
ci setup, but I think the root cause was that in the f function we have the code:
``` clojure
(binding [complete? (s/drained? src)]
```
Before with the on-drained callback, the call to `f []` would happen with
`(s/drained? src)` as true. Now that I appended `[]` to src, `s/drained?` is
still false there. I [edited it
again](https://github.com/clj-commons/gloss/pull/65) so that `src` would refer to the
original src and not the new src with `[]` appended.

In CL with [lisp-binary](https://github.com/j3pic/lisp-binary), everything just
worked and I had no bugs related to parsing. But to be fair to gloss, this is
because lisp-binary was operating in a simpler manner, just everything as
blocking sequential code, with me spawning one thread per client connection. If
gloss was used the same way operating on a BufferedReader with one thread per
client, these concurrency bugs wouldn't happen. Outside of gloss in my own code
some of the bugs I had were cause dealing with deferred values and callbacks is
harder than simple blocking code with one thread per client. Especially for a
language like clojure where data is immutable by default, I think when project
loom (lightweight threads for jvm) makes it into the stable jvm and the clojure
ecosystem adopts it it could be very nice.

2. Capturing a loop variable

None of the bugs I ran into with CL were nearly as hard, I spent days on gloss
whereas no bug in CL took more than 30 minutes to figure out, of which this was
probably the hardest. I had code like:

``` common-lisp
  (spawn "main server thread"
    (loop for client = (socket-accept *socket*)
          do (spawn "client thread"
               (with-open-stream (stream (socket-stream client))
```

`spawn` is a simple macro that expands into:
``` common-lisp
(make-thread (lambda ()
   ...)
   :name "name")
```
It's entirely unnecessary and probably not worth introducing a macro for, I just
wanted to try writing at least one macro and this is the only one I wrote for
protohackers. The bug is the same as a common footgun in Go and presumably
any other language with first-class functions that capture lexical variables,
and mutable loop-scoped variables. If two clients connect in very rapid
succession, client will already be the socket of the next client, before the
first client thread runs `(socket-stream client)`, the threads for both clients
using the stream of the second client. I later realized `usocket` the de facto
networking library of common-lisp, has a `socket-serve` function that does the
work of spawning a thread per connection and passing the stream to a handler
function, so my buggy code was unnecessary. This is one area of concurrency where
clojure is certainly safer, as it couldn't happen for two reasons:
 - the clojure version doesn't use looping but rather map or reduce or other
functions over streams
 - clojure is immutable by default so you don't have these issues of multiple
lambda capturing the same mutable variable.

3. [another bug sharing sockets between threads](https://www.reddit.com/r/lisp/comments/130o9w8/networking_and_threads/) 

See the forum post for a fuller description. Basically I had two threads
operating on the same stream, but I assumed it was safe as one only wrote to it,
and one only read from it. And it seems the underlying posix thread and socket
calls are safe to use this way, but in sbcl's layer on top it is not thread safe
and if you call shutdown on a socket while the other thread is blocking on a
call to read to the socket, the call to read will get stuck in a busy loop at
100% cpu forever. I'm still not entirely sure whether I should report this to
sbcl or if their socket code is simply not intended to be safe to use by multiple
threads like this.

### Yak shaving

No hacking in lisp would be complete without a detour hacking on the tools we
use to hack. This was also the moment where I realize there's something special
in lisp, when I'm hacking on the code instrumentation of cider's clojure
debugger (something I would consider too advanced for me in any other language
but for the same reasons macros are much easier in lisp, code instrumentation is
also), and I'm modifying the code of emacs cider mode and cider-nrepl which is
what is used to connect to your running clojure program. Interactive development
modifying an external program without restarting is one thing, but when you're
modifying live the very tools you are developing them with, it's like inception,
and somehow everything just works.

#### [break on exception](https://github.com/clojure-emacs/cider-nrepl/pull/769)

One of the more annoying things about clojure was getting big java stack traces
that don't actually tell you what exactly happened. I wanted to see what are the
local variables at the moment of an exception, so I added instrumentation to
wrap every form with a try/catch, and enter the debugger in case of an
exception, where you can then see the local variables. It's not really useful
for exceptions you get in production as you have to first instrument the code,
but it works pretty well when you're developing. As clojure is a functional
language, you write and try in the repl one function at a time, and if it get's
an exception you can now press a shortcut to instrument the function and rerun
it and get dropped in the debugger at the point of exception.

#### [thread view](https://github.com/joaotavora/sly/pull/598)

With both the bugs in my common lisp protohackers solutions I mentioned, I ended
up with threads running forever at 100% cpu, so I added a cpu column to sly's
thread view to spot that easier. I also added a column with the amount of time
each thread has been running, thinking that could help spot connections that
don't get closed when they should, if I see a long-lived thread when I expect it
to be short lived. For that it also helps a lot to assign names to your threads.
Here already common lisp's debugging tools on threads was much nicer than
debugging clojure's manifold async streams, which is partly why the CL bugs I had
didn't take very long to find the root cause. I had the thread view where I
could open up the backtrace of any thread, view all the local variables for each
frame and jump to it's source, etc. There's actually a really neat debugger for
clojure called [Flowstorm](https://github.com/jpmonettas/flow-storm-debugger) that
works with core.async, but it's instrumention would hang my program using
manifold's async streams.

### Common Lisp

To sum up my first impressions of common lisp:

#### Advantages
- Speed

In the sense of both development time and compile time. It feels as fast to hack
together a quick prototype as python. Then you can add `(declaim (optimize
speed))` and sbcl will tell you were you need to add type annotations (it can
infer a lot), and your code will perform similar to the fast GC languages like
go, java, and C#.

- Fast feedback

After working some rust projects where I wait 30 seconds to test the effect of
any little change, it is really nice to have instant feedback. With the normal
way of working in CL being modifying the running program without restarting, it
is a faster feedback cycle than anything I've worked with.

- Metaprogramming

In any language programmers will do some sort of metaprogramming to try to
better express what they want without repetitive boilerplate. This can take the
form of a two-step build with code generation, runtime reflection and
metaprogramming, or even compiler extensions. Lisp's macros are I think a
simpler and less problematic approach. Compared to runtime metaprogramming it is
easier to understand as it is happening at compile time, so you can view the
macro definition and expansion, and is safer as there's been many security
vulnerabilities caused by runtime metaprogramming in other languages.

- Concise and Verbose

It is verbose in the sense that it's common to use multiword descriptive names,
but was suprisingly short overall. My protohackers solutions were often half the
lines of code of others' python solutions, which were in turn half or less the
lines of solutions in go or rust. When my CL solutions were nearly the length of
python ones most of the difference was the verbose `defbinary` macro
declaratively stating the binary protocol formats where python just used single
line pack and unpack statements that are imo less readable. This was surprising
to me as I assumed lisp's macros might let it be more concise on a large project
but for short programs it would be more verbose than python. Of course less
lines of code is not a virtue in itself, but I think my CL solutions aren't
cramming lots of expressions into one line or anything, it just seemed to be
able to express more directly the higher-level logic of the problem, with less
boilerplate or repetition or low-level details.

#### Disadvantages

- Unpopular language

It's not a bad thing in itself to be unpopular, but for programming languages it
puts it at a huge disadvantage in terms of ecosystem, documentation, ability to
search online for answers, the libraries you use being actively maintained,
community to answer questions, and people to collaborate on your projects. For
all of these reasons, I will unfortunately probably not use lisp much for
serious projects. The rest of the disadvantages are kind of also a consequence
of unpopularity in that if the language was more popular, someone would put in
the work and improve them, but other niche languages have managed to do them
well.

- Naming (mapcar, mapcan, nconc, caddr ...)

Many CL standard library functions have strange names that I can understand are
offputting to newcomers. Clojure picked better names, but there is a consistency
to the CL naming and it's not much of an obstacle, you get used to it quite fast.

- package manager and build system

The build system asdf is maybe more flexible and simpler for complex projects (I
don't have experience to say), but for simple projects it is more complex than
most languages. The main package manager (quicklisp) doesn't support installing
different versions of a dependency per-project which doesn't seem to be much of
a problem in practice as lisp seems to have a quite good culture of libraries
not breaking backwards compatibility. A bigger problem is it is insecure, not
checking any sort of signature or hash or even downloading over https.

- Distribution

Clojure is pretty good here, you can generate an "uberjar" with all dependencies
that will run anywhere a JVM is installed. CL is not so easy.

- Concurrency

Programming with system threads in CL was actually quite pleasant, it has high
level libraries for it and a nice debugging experience. Nowadays on linux 10
thousand threads is nothing, but if you need a million concurrent connections
then CL is not the best choice as it doesn't have good async libraries now. I
don't know old lisp history but this has maybe even regressed? It seems some old
CL implementations used green threads rather than system threads, and decades
ago there was talk of adding delimited continuations to sbcl which would allow
for ergonomic high-performance concurrency. This is kind of also a consequence
of popularity in that it could be fixed, but also a cause of the lack of
popularity.  Most backend programming is highly concurrent network services and
we can see in rust years ago the huge demand from it's corporate backers to get
it some sort of async support, which was necessary for rust to become mainstream.

### Conclusion

I used to try out shiny new tech in my free time, now I try out old crufty tech.
It seems in many ways ML, lisp, and erlang were more advanced decades ago than
what is "modern" today. And it's not just languages... usenet was in
[many ways superior to modern social
media](https://jfm.carcosa.net/blog/computing/usenet/).

If you think lisp might be fun, give it a try. Don't be like me and discard it
just because the parenthesis look ugly at first sight, you get used to them fast.
Will it make you a more enlightened, more productive programmer? In the words
of na85 on reddit:

> I reached enlightenment when I realized that very rarely does the technology stack actually matter, whether lisp or not.
