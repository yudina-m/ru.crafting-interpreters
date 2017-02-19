^title Карта территории
^part Добро пожаловать

> У вас должна быть карта, неважно, насколько грубая. Иначе вы будете бродить
> повсюду. Во "Властелине Колец" я никогда не заставлял никого пройти дальше,
> чем он мог в определенный день.
>
> <cite>Дж.Р.Р. Толкин</cite>

Мы не хотим бродить повсюду, поэтому, прежде чем мы отправимся в путь, давайте
просканируем территорию, намеченную предыдущими создателями языков. Это поможет
нам понять, куда мы направляемся и какими альтернативными маршрутами пользуются другие.

Для начала позвольте мне обозначить определения. Большая часть этой книги посвящена
*реализации* языка, которая отлична от *самого языка*, представленного в виде какой-то
Платоновской идеальной формы. Такие вещи как "стек", "байткод" и "метод рекурсивного спуска"
являются гайками и болтами, которые может использовать определенная реализация. С точки зрения
пользователя, до тех пор пока результирующая вещь честно следует спецификации языка,
всё это – детали реализации.

Мы собираемся потратить много времени на эти детали, так что если я буду писать
"*реализация* языка" каждый раз, когда их упоминаю, я поизношу свои пальцы.
Вместо этого я буду использовать термин "язык" для обращения либо к языку, либо к его реализации,
либо к ним обоим, если различия не очевидны.

## Части языка

Инженеры создавали языки программирования с Темных Времен
вычислений(?). Как только мы смогли разговаривать с компьютерами и обнаружили, что делать это чрезвычайно
сложно, мы заручились их (прим. компьютеров) помощью. Я нахожу захватывающим, что даже несмотря на то, что сегодняшние
машины буквально в миллион раз быстрее и имеют на порядок больше места
хранения, способ создания компьютерных языков практичесик не изменился.

Хотя территория, охваченная дизайнерами языков огромна, следы, которые они оставили
на ней <span name="dead">невелики</span>. Не каждый язык проходит один
и тот же путь -- некоторые сокращают путь тут или там -- но в остальном они
успокоительно похожи с первого COBOL компилятора контр-адмирала Грейс Хоппер до
самого горячего нового языка транспиллера-в-JavaScript, чья "документация"
состоит полностью из плохо отредактированного README где-то в Git репозитории.

<aside name="dead">

Хотя, конечно же, существуют и тупики, грустные маленькие cul-de-sacs CS работ с
нулевой цитируемостью и ныне забытыми оптимизациями, которые имели смысл только когда
память измерялась в индивидуальных байтах.

</aside>

Я визуализирую сеть путей, которые может выбрать реализация, как восхождение на гору.
Вы начинаете внизу с программой в виде исходного текста,
буквально в виде строки символов. Каждая фаза анализирует программу и
превращает ее в более высокоуровневое представление, где семантика -- то,
что автор хочет, чтобы сделал компьютер -- становится более очевидной.

В конце концов мы достигаем вершины. У нас есть вид с высоты птичьего полета на программу пользователя
и мы может видеть, что *означает* их код. Затем мы спускаемся по другой части горы.
Мы преобразуем самое высокоуровневое представление последовательно вплоть до форм на более низком
уровне, становясь все ближе и ближе к чему-то, что мы знаем как заставить
процессор (CPU) на самом деле выполнить.

<img src="image/a-map-of-the-territory/mountain.png" alt="The branching paths a language may take over the mountain." class="wide" />

Давайте пройдем через каждый из этих маршрутов и достопримечательностей. Наше путешествие
начинается слева с голого текста исходного кода пользователя:

<img src="image/a-map-of-the-territory/string.png" alt="var average = (min + max) / 2;" />

### Сканирование (Scanning)

Первый шаг – это **сканирование (scanning)**, также известное как **лексинг (lexing)**, или
(если вы пытаетесь кого-то впечатлить) **лексический анализ**. Они все означают практически
одно и то же. Я предпочитаю "лексинг (lexing)", потому что это звучит как что-то, что мог бы сделать
суперзлодей, но я буду использовать "сканирование (scanning)", потому что это, кажется,
немного более распространено.

**Сканер (scanner)** (или **"лексер (lexer)"**) принимает на вход линейный поток символов
и объединяет их вместе в серии чего-то более похожего на <span
name="word">"слова"</span>. В языках программирования каждое из этих слов
называется **токен (token)**. Некоторые токены (tokens) - это один символ, например, `(` и `,`. Другие
могут состоять из нескольких символов, таких как числа (`123`), строки (`"hi!"`)
и идентификаторы (`min`).

<aside name="word">

"Лексический (Lexical)" происходит от Греческого "lex", означающего "слово".

</aside>

Некоторые символы в исходном файле не означают ничего. Пробелы обычно
не важны и комментарии, по определению, игнорируются языком.
Сканер (scanner) обычно отбрасывает их, оставляя чистую последовательность
значимых токенов (tokens).

<img src="image/a-map-of-the-territory/tokens.png" alt="[var] [average] [=] [(] [min] [+] [max] [)] [/] [2] [;]" />

### Парсинг (Parsing)

The next step is **parsing**. This is where our syntax gets a **grammar** -- the
ability to compose larger expressions and statements out of smaller parts. Did
you ever diagram sentences in English class? If so, you've done what a parser
does, except that English has thousands and thousands of "keywords" and an
overflowing cornucopia of ambiguity. Programming languages are much simpler.

A **parser** takes the flat sequence of tokens and builds a tree structure that
mirrors the nested nature of the grammar. These trees have a couple of different
names -- **"parse tree"** or **"abstract syntax tree"** -- depending on how
close to the bare syntactic structure of the source language they are. In
practice, language hackers usually call them **"syntax trees"**, **"ASTs"**, or
often just **"trees"**.

<img src="image/a-map-of-the-territory/ast.png" alt="An abstract syntax tree." />

Parsing has a long, rich history in computer science that is closely tied to the
artifical intelligence community. Many of the techniques used today to parse
programming languages were originally conceived to parse *human* languages by AI
researchers who were trying to get computers to talk to us.

It turns out human languages are too messy for the rigid grammars those parsers
could handle, but they were a perfect fit for the simpler artificial grammars of
programming languages. Alas, we flawed humans still manage to fail to use those
simple grammars correctly, so the parser's job also includes telling us by
reporting **syntax errors**.

### Static analysis

The first two stages are pretty similar across all implementations. Now, the
individual characteristics of each language start coming into play. At this
point, we know the syntactic structure of the code -- things like operator
precedence and expression nesting -- but we don't know much more than that.

In an expression like `a + b`, we know we are adding `a` and `b`, but we don't
know what those names refer to. Are they local variables? Global? Where are they
defined?

The first bit of analysis that most languages do is called **binding** or
**resolution**. For each **identifier** we find out where that name is defined
and wire the two together. This is where **scope** comes into play -- the region
of source code where a certain name can be used to refer to a certain
declaration.

If the language is <span name="type">statically typed</span>, this is when we
type check. Once we know where `a` and `b` are declared, we can also figure out
their types. Then if those types don't support being added to each other, we
report a **type error**.

<aside name="type">

The language we'll build in this book is dynamically typed, so it will do its
type checking later, at runtime.

</aside>

Take a deep breath. We have attained the summit of the mountain and a sweeping
view of the user's program. All this semantic insight that is visible to us from
analysis needs to be stored somewhere. There are a few places we can squirrel it
away:

* Often, it gets stored right back as **attributes** on the syntax tree
  itself -- extra fields in the nodes that aren't initialized during parsing
  but get filled in later.

* Other times, we may store data in a look-up table off to the side. Typically,
  the keys to this table are identifiers -- names of variables and declarations.
  In that case, we call it a **symbol table** and the values it associates with
  each key tell us what that identifier refers to.

* The most powerful bookkeeping tool is to transform the tree into an entirely
  new data structure that more directly expresses the semantics of the code.
  That's the next section.

Everything up to this point is considered the **front end** of the
implementation. You might guess everything after this is the **back end**, but
no. Back in the days of yore when "front end" and "back end" were coined,
compilers were much simpler. Later researchers invented new phases to stuff
between the two halves. Rather than discard the old terms, William Wulf and
company lumped them into the charming but spatially paradoxical name **"middle
end"**.

### Intermediate representations

You can think of the compiler as a pipeline where each stage's job is to
organize the code in a way that makes the next stage simpler to implement. The
front end of the pipeline is specific to the source language the user is
programming in. The back end is concerned with the final architecture that the
code will run on.

In the middle, the code may be stored in some <span name="ir">**intermediate
representation**</span> (or **"IR"**) that isn't tightly tied to either the
source or destination forms (hence "intermediate"). Instead, the IR acts as an
interface between these two languages.

<aside name="ir">

There are a few well-established styles of IRs out there. Hit your search engine
of choice and look for "control flow graph", "static single-assignment",
"continuation-passing style", and "three-address code".

</aside>

This lets you support multiple source languages and target platforms with less
effort. Say you want to implement Pascal, C and Fortran compilers and you want
to target x86, ARM, and, I dunno, SPARC. Normally, that means you're signing up
to write *nine* full compilers: Pascal&rarr;x86, C&rarr;ARM, and every other
combination.

A <span name="gcc">shared</span> intermediate representation reduces that
dramatically. You write *one* front end for each source language that produces
the IR. Then *one* back end for each target architecture. Now you can mix and
match those to get every combination.

<aside name="gcc">

If you've ever wondered how [GCC][] supports so many crazy languages and
architectures, like Modula-3 on Motorola 68k, now you know. Language front ends
target one of a handful of IRs, mainly [GIMPLE][]. Then target backends like the
one for 68k take the GIMPLE and produce native code.

[gcc]: https://en.wikipedia.org/wiki/GNU_Compiler_Collection
[gimple]: https://gcc.gnu.org/onlinedocs/gccint/GIMPLE.html

</aside>

There's another big reason we might want to transform the code into a form that
makes the semantics more apparent...

### Optimization

Once we understand what the user's program means, we are free to swap it out
with a different program that has the *same semantics* but implements them more
efficiently -- we can **optimize** it.

A simple example is **constant folding**: if some expression always evaluates to
the exact same value, we can do the evaluation at compile time and replace the
code for the expression with its result. If the user typed in:

```java
pennyArea = 3.14159 * (0.75 / 2) * (0.75 / 2);
```

We can do all of that arithmetic in the compiler and change the code to:

```java
pennyArea = 0.4417860938;
```

Optimization is a huge part of the programming language business. Many language
hackers spend their entire careers here, squeezing every drop of performance
they can out of their compilers to get their benchmarks a fraction of a percent
faster. It can become a sort of obsession.

We're going to <span name="rathole">hop over that rathole</span> in this book.
Many successful languages have surprisingly few compile-time optimizations. For
example, Lua and CPython generate relatively unoptimized code, and focus most of
their performance effort on the runtime.

<aside name="rathole">

If you can't resist poking your foot into that hole, some keywords to get you
started are "constant propagation", "common subexpression elimination", "loop
invariant code motion", "global value numbering", "strength reduction", "scalar
replacement of aggregates", "dead code elimination", and "loop unrolling".

</aside>

### Code generation

We have applied all of the optimizations we can think of to the user's program.
The last step is converting it to a form the machine can actually run. In other
words **generating code**, where "code" refers to the kind of primitive
assembly-like instructions a CPU runs and not the kind of "source code" a human
might want to read.

Finally, we are in the **back end**, descending the other side of the mountain.
From here on out, our representation of the code becomes more and more
primitive, like evolution run in reverse, as we get closer to something our
simple-minded machine can understand.

We have a decision to make. Do we generate instructions for a real CPU or a
virtual one? If we generate real machine code, we get an executable that the OS
can load directly onto the chip. Native code is lightning fast, but generating
it is a lot of work. Today's architectures have piles of instructions, complex
pipelines, and enough <span name="aad">historical baggage</span> to fill a 747.

Speaking the chip's language also means your compiler is tied to a specific
architecture. If your compiler targets [x86][] machine code, it's not going to
run on an [ARM][] device. All the way back in the 60s, during the Cambrian
explosion of computer architectures, that lack of portability was a real
obstacle.

<aside name="aad">

For example, the [AAD][] ("ASCII Adjust AX Before Division") instruction lets
you perform division, which sounds useful. Except that instruction takes as
operands two binary-coded decimal digits packed into a single 16-bit register.
When was the last time *you* needed BCD on a 16-bit machine?

[aad]: http://www.felixcloutier.com/x86/AAD.html

</aside>

[x86]: https://en.wikipedia.org/wiki/X86
[arm]: https://en.wikipedia.org/wiki/ARM_architecture

To get around that, hackers like Martin Richards and Niklaus Wirth, of BCPL and
Pascal fame, respectively, made their compilers produce *virtual* machine code.
Instead of instructions for some real chip, they produced code for a
hypothetical, idealized machine. Wirth called this **"p-code"** for "portable",
but today, we generally call it **bytecode** because each instruction is often a
single byte long.

These synthetic instructions are designed to map a little more closely to the
language's semantics, and not be so tied to the peculiarities of any one
computer architecture and its accumulated historical cruft. You can think of it
like a dense, binary encoding of the language's low-level operations.

### Virtual machine

If your compiler produces bytecode, your work isn't over once that's done. Since
there is no chip that speaks that bytecode, it's your job to translate. Again,
you have two options. You can write a little mini-compiler for each target
architecture that converts the bytecode to native code for that machine. You
still have to do work for <span name="shared">each</span> chip you support, but
this last stage is pretty simple and you get to reuse the rest of the compiler
pipeline across all of the machines you support. You're basically using your
bytecode as an intermediate representation.

<aside name="shared">

The basic principle here is that the farther down the pipeline you can push the
architecture-specific work, the more of the earlier phases you can share across
architectures.

There is a tension, though. Many optimizations, like register allocation and
instruction selection, work best when they know the strengths and capabilities
of a specific chip. Figuring out which parts of your compiler can be shared and
which should be target-specific is an art.

</aside>

Or you can write a **virtual machine** (**"VM"**), a program that emulates a
hypothetical chip supporting your virtual architecture at runtime. Running
bytecode in a VM is slower than translating it to native code ahead of time
because every instruction must be simulated at runtime each time it executes. In
return, you get simplicity and portability. Implement your VM in, say, C, and
you can run your language on any platform that has a C compiler. This is what
our second interpreter does.

### Runtime

We have finally hammered the user's program into a form that we can execute. The
last step is running it. If we compiled it to machine code, we simply tell the
operating system to load the executable and off it goes. If we compiled it to
bytecode, we need to start up the VM and load the program into that.

In both cases, for all but the basest of low-level of languages, we usually need
some services that our language provides while the program is running. For
example, if the language automatically manages memory, we need a garbage
collector going in order to reclaim unused bits. If our language supports
"instance of" tests so you can see what kind of object you have, then we need
some representation to keep track of the type of each object during execution.

All of this stuff is going at runtime, so it's called, well, the **"runtime"**.
In a fully compiled language, the code implementing the runtime gets inserted
directly into the resulting executable. In, say, [Go][], each compiled
application has its own copy of Go's runtime directly embedded in it. If the
language is run inside an interpreter or VM, then the runtime lives there. This
is how most implementations of languages like Java, Python, and JavaScript work.

[go]: https://golang.org/

## Shortcuts and Alternate Routes

That's the long path covering every possible phase you might implement. Many
languages do walk the entire route, but there are a few shortcuts and alternate
paths.

### Single-pass compilers

Some simple compilers interleave parsing, analysis, and code generation so that
they produce output code directly in the parser, without ever allocating any
syntax trees or other IRs. These <span name="sdt">**single-pass
compilers**</span> restrict the design of the language. You have no intermediate
data structures to store global information about the program, and you don't
revisit any previously parsed part of the code. That means as soon as you see
some expression, you need to know enough to correctly compile it.

<aside name="sdt">

[**Syntax-directed translation**][pass] is a structured technique for building
these all-at-once compilers. You associate an *action* with each piece of the
grammar, usually one that generates output code. Then, whenever the parser
matches that chunk of syntax, it executes the action, building up the target
code one rule at a time.

[pass]: https://en.wikipedia.org/wiki/Syntax-directed_translation

</aside>

Pascal and C were designed around this limitation. At the time, memory was so
precious that a compiler might not even be able to hold an entire *source file*
in memory, much less the whole program. This is why Pascal's grammar requires
type declarations to appear first in a block. It's why in C you can't call a
function above the code that defines it unless you have an explicit *forward
declaration* that tells the compiler what it needs to know to generate code for
a call to the later function.

### Tree-walk interpreters

Some programming languages begin executing code right after parsing it to an AST
(with maybe a bit of static analysis applied). To run the program, it traverses
the syntax tree one branch and leaf at a time, evaluating each node as it goes.

This style of interpretation is common for student projects and little
languages, but is not widely used for general-purpose languages since it tends
to be slow. A notable exception is the original implementation of <span
name="ruby">Ruby</span> which was a tree walker prior to version 1.9.

<aside name="ruby">

At 1.9, the canonical implementation of Ruby switched from the original MRI
("Matz' Ruby Interpreter") to Koichi Sasada's YARV ("Yet Another Ruby VM"). YARV
is A Bytecode Virtual Machine.

</aside>

Some people use "interpreter" to mean only these kinds of implementations, but
others define that word more generally, so I'll use the inarguably explicit
**"tree-walk interpreter"** to refer to these. Our first interpreter rolls this
way.

### Transpilers

<span name="gary">Writing</span> a complete back end for a language can be a lot
of work. If you have some existing generic IR to target, you could bolt your
front end onto that. Otherwise, it seems like you're stuck. But what if you
treated some other *source language* as if it were an intermediate
representation?

We write a front end for our language. Then, in the back end, instead of doing
all the work to *lower* the semantics to some primitive target language, we
produce a string of valid source code for some other language that's about as
high level as ours. Then, we use the existing compilation tools for *that*
language as our escape route off the mountain and down to something we can
execute.

They used to call this a **"source-to-source compiler"** or a
**"transcompiler"**. After the rise of languages that compile to JavaScript in
order to run in the browser, they've affected the hipster sobriquet
**"transpiler"**.

<aside name="gary">

The first transcompiler, XLT86, translated 8080 assembly into 8086 assembly.
That might seem straightforward, but keep in mind the 8080 was an 8-bit chip and
the 8086 a 16-bit chip that could use each register as a pair of 8-bit ones.
XLT86 did data flow analysis to track register usage in the source program and
then efficiently map it to the register set of the 8086.

It was written by Gary Kildall, a tragic hero of computer science if there
ever was one. One of the first people to recognize the promise of
microcomputers, he created PL/M and CP/M, the first high level language and OS
for them.

He was a sea captain, business owner, licensed pilot, and motocyclist. A TV host
with the Kris Kristofferson-esque look sported by dashing bearded dudes in the
80s. He took on Bill Gates and, like many, lost, before meeting his end
in a biker bar under mysterious circumstances. He died too young, but sure as
hell lived before he did.

</aside>

While the first transcompiler translated one assembly language to another,
today, almost all transpilers work on higher-level languages. After the viral
spread of UNIX to machines various and sundry, there began a long tradition of
compilers that produced C as their output language. C compilers were available
everywhere UNIX was and produced efficient code, so targetting C was a good way
to get your language running on a lot of architectures.

Web browsers are the "machines" of today, and their "machine code" is
JavaScript, so these days it seems [almost every language out there][js] has a
compiler that targets JS since that's the <span name="js">only</span> way to get
your code running in a browser.

[js]: https://github.com/jashkenas/coffeescript/wiki/list-of-languages-that-compile-to-js

<aside name="js">

JS may not be the only language browsers natively support for much longer. If
[Web Assembly][] takes off, browsers will support another lower-level language
specifically designed to be targeted by compilers.

[web assembly]: https://github.com/webassembly/

</aside>

The front end -- scanner and parser -- of a transpiler looks like other
compilers. Then, if the source language is only a simple syntactic skin over the
target language, it may skip analysis entirely and go straight to outputting the
analogous syntax in the destination language.

If the two languages are more semantically different, then you'll see more of
the typical phases of a full compiler including analysis and possibly even
optimization. Then, when it comes to code generation, instead of outputting some
binary language like machine code, you produce a string of grammatically correct
source (well, destination) code in the target language.

Either way, you then run that resulting code through the output language's
existing compilation pipeline and you're good to go.

### Just-in-time compilation

This last one is less of a shortcut and more a challenging scramble best
reserved for experts. The fastest way to execute code is by compiling it to
machine code, but you might not know what architecture your end user's machine
supports. What to do?

You can do the same thing the HotSpot JVM, Microsoft's CLR and most JavaScript
interpreters do. On the end user's machine, when the program is loaded -- either
from source in the case of JS, or platform-independent bytecode for the JVM and
CLR -- you compile it to native for the architecture their computer supports.
Naturally enough, this is called **just-in-time compilation.** Most hackers just
say "JIT", pronounced like it rhymes with "fit".

The most sophisticated JITs insert profiling hooks into the generated code to
see which regions are most performance critical and what kind of data is flowing
through them. Then, over time, they will automatically recompile those <span
name="hot">hot spots</span> with more advanced optimizations.

<aside name="hot">

This is, of course, exactly where the HotSpot JVM gets its name.

</aside>

## Compilers and Interpreters

Now that I've stuffed your head with a dictionary's worth of programming
language jargon, we can finally address a question that's plagued coders since
time immemorial: "What's the difference between a compiler and an interpreter?"

It turns out this is like asking the difference between a fruit and a vegetable.
That seems like a binary either-or choice, but actually "fruit" is a *botanical*
term and "vegetable" is *culinary*. One does not imply the negation of the
other. There are fruits that aren't vegetables (apples) and vegetables that are
not fruits (carrots), but also edible plants that are both fruits *and*
vegetables, like tomatoes.

<span name="veg"></span></span>

<img src="image/a-map-of-the-territory/plants.png" alt="A Venn diagram of edible plants" />

<aside name="veg">

There are even plant-based foods that are *neither*, like nuts and cereals. (And
peanuts aren't even nuts!)

</aside>

So, back to languages:

* **Compilation** is an *implementation technique* that involves translating a
  source language to some other -- usually lower-level -- form. When you
  generate bytecode or machine code, you are compiling. When you transpile to
  another high-level language you are compiling too. If users run a tool that
  takes a source language and outputs some target language and then stops, we
  call that tool a **compiler**.

* **Interpretation** describes the *user experience of executing a language*. If
  the end user has a single tool that takes in source code and is able to then
  execute it immediately, that tool is an **interpreter**.

Like apples and oranges, some implementations are clearly compilers and *not*
interpreters. GCC and Clang take your C code and compile it to machine code. An
end user runs that executable directly and may never even know which tool was
used to compile it. So those are *compilers* for C.

In older versions of Matz' canonical implementation of Ruby, the user ran Ruby
from source. The implementation parsed it and ran it directly by traversing the
syntax tree. No other translation occurred, either internally or in any
user-visible form. So this was definitely an *interpreter* for Ruby.

But what of CPython? When you run your Python program using it, the code is
parsed and converted to an internal bytecode format, which is then executed
inside the VM. From the user's perspective, this is clearly an interpreter --
they run their program from source. But if you look under CPython's scaly skin,
you'll see that there is definitely some compiling going on.

The answer is that it is <span name="go">both</span>. CPython *is* an
interpreter, and it *has* a compiler. In practice, most scripting languages work
this way, as you can see:

<aside name="go">

The [Go tool][go] is even more of a horticultural curiosity. If you run `go
build`, it compiles your Go source code to machine code and stops. If you type
`go run`, it does that then immediately executes the generated executable.

So `go` *has* a compiler, *is* an interpreter, and *is* also a compiler.

[go tool]: https://golang.org/cmd/go/

</aside>

<img src="image/a-map-of-the-territory/venn.png" alt="A Venn diagram of compilers and interpreters" />

That overlapping region in the center is where our second interpreter lives too,
since it internally compiles to bytecode. So while this book is nominally about
interpreters, we'll cover some compilation too.

## Our Journey

That's a lot to take in all at once. Don't worry. This isn't the chapter where
you're expected to *understand* all of these pieces and parts. I just want you
to know that they are out there and roughly how they fit together.

This map should serve you well as you explore the territory beyond the guided
path we take in this book. I want to leave you yearning to strike out on your
own and wander all over that mountain.

But, for now, it's time for our own journey to begin. Tighten your bootlaces,
cinch up your pack, and come along. From <span name="here">here</span> on out,
all you need to focus on is the path in front of you.

<aside name="here">

Henceforth, I promise to tone down the whole mountain metaphor thing.

</aside>

<div class="challenges">

## Challenges

1. Pick an open source implementation of a language you like. Download the
   source code and poke around in it. Try to find the code that implements the
   scanner and parser. Are they hand-written, or generated using tools like
   Lex and Yacc? (`.l` or `.y` files usually imply the latter.)

1. Just-in-time compilation tends to be the fastest way to implement a
   dynamically-typed language, but not all of them use it. What reasons are
   there to *not* JIT?

1. Most Lisp implementations that compile to C also contain an interpreter that
   lets them execute Lisp code on the fly as well. Why?

</div>
