# <div align="center">The Great<br><code>2020</code><br>Scott Meyers Outtake<sup>*</sup><br>[Modern C++ type CoDec Challenge](#the-modern-c-type-codec-challenge)</div>
<p align="center">(click title ^^^ to go direct)<br><b>A meta-programming challenge in compound type destructuring</b><br>(assumes intermediate to advanced level C++ with some meta skills)<br><br>Find the most effective, modern way to:&nbsp;&nbsp;&nbsp;<br>(0) <b>Decompose a compound type</b> then&nbsp;&nbsp;&nbsp;<br>(1) <b> Represent its structure and traverse it</b><br><br><b><code>now()<2020</code></b>: Using Modern C++ up to c++20 with compiler extensions included<br><b><code>now()>2020</code></b>: Using Post Modern C++ with Scalable Reflection or other reflection</p>

#### (The slow-boiled-egg or shaggy-chicken version)...

Scott is MIA from the C++ melee, and missed, following publication of his parting gift, **EMC++**  
(an early Easter egg inside hid; once missing, now hatching to missing-link - `tid`):  

### [Effective Modern C++](http://www.jdoqocy.com/click-7708709-11290546?url=http%3A%2F%2Fshop.oreilly.com%2Fproduct%2F0636920033707.do%3Fcmp%3Daf-code-books-video-product_cj_0636920033707_%25zp&cjsku=0636920033707), Scott Meyers, 2014

**EMC++ __Item 4__**: __Know How to View Deduced Types__ discusses how to map from a type to its  
displayable representation. This snippet is from the final section of a [pre-publication draft](https://aristeia.com/Papers/EMC++%20Early%20Release%202014-07-01%20Item%204.pdf):

<details><summary><b><i>Beyond typeid</i></b>:<blockquote> ... <br> ... implement your own mechanism for mapping from a type to its displayable representation.
<br><i>In concept, it’s not difficult</i>: <b>you just use type traits and template metaprogramming</b> ...</blockquote>
</summary>

>  
> #### Beyond typeid
> If you want accurate runtime information about deduced types, we’ve seen that  typeid  
is not a reliable route to getting it. One way to work around that is to   
<b>implement your own mechanism for mapping from a type to its displayable representation.  
<i>In concept,  it’s not difficult</i>: you just use type traits and template metaprogramming</b>  
(see Item 9) to break a type into its various components (using type traits such as std::is_const,  
std::is_pointer, std::is_lvalue_reference, etc.), and you create a string representation of the type  
from textual representations of each of its parts. (You’d still be dependent on typeid and std::  
type_info::name to generate string representations of the names of user-defined classes, though.)

</details>

(My emphasis. Click to unfold the full paragraph for context.)  
<sup>*</sup> The section was removed from the published version in favor of discussion of [Boost.TypeIndex](https://www.boost.org/doc/libs/1_71_0/doc/html/boost_typeindex.html).  
&nbsp;&nbsp;A reader noticed the removal, asked why it was cut, and Scott wrote a [blog](https://scottmeyers.blogspot.com/2015/07/emc-outtake-prettyfunction-and-funcsig.html) post giving reasons.

Let's call this the ***TMP typename problem***, or just ***typename!*** (with eggsclamation).

### Down with typename!

***typename!*** is triply troubled (a bad egg, self destructor concealed in an empty shell).  

Firstly, the bad; the quirky declaration syntax of C/C++ only adds accidental complexity  
(and confusion and comical canonical controversy as to whether East or West is best).  
Try examples from the cdecl [C gibberish ↔ English](https://cdecl.org/) translator, e.g. `char (*(*())[5])()`

Secondly, why bother to decompose a type at all if, in extracting a type's name, you have to  
use some means to get all its 'leaf' names; you'd just use that means to get the 'root' name.  
Reductio ad Absurdum! Self destruct!

(Scalable **Reflection** proposes a first class means to get a type's name; `meta::name_of`.  
&nbsp;Plus ça change; typename is still a quirky serialization, only fit for human confusion,   
&nbsp;absurd as a format for machine consumption, unless for actual C++ source codegen,  
&nbsp;and the typename format will never be canonical without standardization effort.)

Thirdly, *typename!* is shallow. What if we want to recreate the full declaration of a type?  
Then we need to dig deeper into user-defined types - reflection *is* needed for this  
(to blow the hinges off the box-without-key-or-lid and get to the golden treasure - `tid`).

So, reflection is required in order to achieve *typename!* completeness; `await(reflection)`  
(we're "waiting for godbolt" to add a Scalable Reflection implementation; I'll egg it on).

Part of the *typename!* problem lies in its statement; converting a type directly to a name,  
with no explicit intermediate step, conflates *type decomposition* with all the issues above  
(perhaps this was the real, hidden cause of its great fall).

### TMP CoDec

Abrahams & Gurtovoy set a similar exercise a decade earlier in their classic TMP text  
**[C++ Template Metaprogramming](https://www.oreilly.com/library/view/c-template-metaprogramming/0321227255/)**

**TMP Chapter 2**, *Traits and Type Manipulation*, Exercise 2-3 and follow up Exercise 2-4:
> **2-3**. Use the type traits facilities to implement a type_descriptor class template,  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; whose instances, when streamed, print the type of their template parameters.  
**2-4**. Write an alternative solution for exercise 2-3 that does not use the Type Traits library.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Compare the solutions.

Dave Abrahams and Aleksey Gurtovoy, 2004 (prior simplifying context cut, for eggsposition)

Crucially, the exercise is explicit in requiring an intermediate `type_descriptor` representation;  
*Dec*ompound a type to a descriptor then, from the descriptor, *Co*mpound to serialized form.

Let's call this a *type CoDec*. The *typename!* problem is void. Long live the **_CoDec_** challenge!

### The structure and interpretation of types

Need a structural decomposition of a compound type? You need a tree rep: 

* **rep**: a representation of the type, easier to decode to and from than the type itself

For example, take the type `int const*` and deconstruct it via mutating type_traits:
```c++
  rep_t<int const*> -> add_pointer_t<add_const_t<int>>
```
Here, type traits provide a 'decode-to' target for mocking up type-decoder techniques  
(`rep_t`, a type decompounder algo, should compute a type identity transformation).  
As such, this use of traits is *half* a rep - the *decode-to* half.

**Padawan challenge**: find the most effective, modern method to decompound types.

The trivial half-a-rep above is sufficient for trying out different decomposition methods.  
It isn't hard from there to design a trait-like nested template rep that memos each level:
```c++
  rep<int const*> -> Pointer<Const<Funda<int>>>
```
Then, add a uniform API to iterate, traverse or visit its decompounded levels.  
This is now a full rep; a decode-to target also designed to be easy to decode-from.

**Jedi challenge**: design a rep for ease of traversal and demonstrate its use.

The nested template rep sketched above is *not* type-erased as it represents a type  
in terms of another type, a wrapper type to which a traversal API can be added.  
Non type-erased reps should suffice for all the challenges enumerated below  
(except for *parsing*, an optional final challenge, if reading from runtime data).  

Here are some branching types with ascii sketches of sample type-tree structures:

```c++
enum E {eh=1,be};
struct S {E a; int b;};

     E              S            int[4]       void(int)        E S::*

    Enum - "E"    Struct - "S"   Array        Function      MemberPointer
    /  \           /  \          /   \         /    \         /     \
   1    2         E   int      int    4      void   int      S       E
 "eh"  "be"      "a"  "b"
```

#### UDTs User-defined types (advanced, optional)

<details><summary>Without reflection, it is very challenging to destructure user-defined types</summary>
  
completely, to include any base classes, member identifiers and initializers.  
They may be class template specializations. They may be anonymous.

For the purposes of this challenge, at least the pre-reflection part, we will  
ignore UDT member identifiers; imagine they're accessed by index rather than id:

* Enum models a set of allowable values
* Struct models a language tuple
* Union models a language variant

Erase all names from the user-defined types above:

```c++
    Enum          Struct        unify         Enum <--      Struct 
    /  \           /  \        ======>        /  \     \     /  \ 
   0    1       Enum  int                    0    1     \___/   int  
                /  \     
               0    1    
```

('unify' comes for free with template reps; the compiler unifies `Enum<0,1>`)  
We've gone too far and erased UDT id. Add it back with strong Typedef nodes:  

```c++
char* sym[]{"E","S"};
Typedef* id[]{&,&}
             /   \
      Typedef     Typedef
         |          |   
       Enum       Struct
       /  \        /  \
      0    1    id[0] int 
```
This is just a (soft-boiled) sketch.  

</details>
  
#### Compounds and their introspection (possible spoiler alert)

<details><summary>Here is the list of compound types and methods of introspection (click to unfold)<br>&nbsp;&nbsp;&nbsp;&nbsp;(could be a spoiler if you want to work out or discover these things for yourself)</summary>

<br>**Compound types** in rough order of difficulty.  

* **cv** qualifiers `const` and/or `volatile`<br>(technically, cv is not a compound but a *modifier*)
* **pointer** `*`
* **reference**
  * lvalue `&`
  * rvalue `&&`
* **array**
  * unbounded `[]`
  * bounded `[N]`
* **function**
  * free (no cvref) function `R(P...,[...]) noexcept(NX)`
  * cvref qualified function `R(P...,[...]) CVREF noexcept(NX)`
* **member-pointer**
  * member object pointer  `T C::*`
  * member function pointer `F C::*`
* **enum**  `enum [class] E : I { id0[=e0], id1[=e1], ... }`
* **class** `struct S : B... { T0 id0[=v0]; T1 id1[=v1]; ... }`
* **union** `union  U : B... { T0 id0[=v0]; T1 id1[=v1]; ... }`
* **bit-field**<br>(whether bit-field is a type or not, we need to deal with it)
* **class template specialization** `struct S<?...>`<br>(not a compound, need to deal with the template Args) 

#### Introspection

**Summary**  
The std type_traits provide for basic introspection of the compound types.  
For built-in types, any missing traits are straightforward to implement.  
For user-defined types, going beyond the basic traits calls for 'magic';  
exploiting compiler extensions as a way to implement basic reflection.

(Scalable reflection aims at a complete list of reflection-expression operators  
&nbsp;mirroring and extending the capabilities of type_traits.)

**Traits**
* **`cv`**, **`pointer`** and **`reference`** have full std type_trait support.
* **`array`** lacks mutating `add` trait(s), which would take array extent(s).  
* **`function`** type only has `std::is_function` trait.
* **`member-pointer`** types only have `std::is_member_*pointer` trait.

For these built-in types all traits, including missing traits, are implementable  
via template argument pattern matching.  

For array, the missing traits are straightforward to implement.  
For function, the traits are straightforward but tediously repetitive  
(there are 24 or 48 possible specializations of cvref-qualified signatures)  
(ignoring cvref-quals reduces this to 2 or 4 'free function' signatures).  
For member-pointers, decomposition is easy enough though the syntax is tricky.

The user-defined types **`enum`**, **`class`** and **`union`** have no trait support  
to introspect enumeration values or non-static data members.

There is a history of heroic meta-programming to achieve UDT introspection.  
Check the [Resources](#resources.md) for links and descriptions.

Reflection should eventually enable full decomposition of compound types.
</details>

### What's the rep?

How low do you go?

Between 2004 and 2005 Gabriel Dos Reis and Bjarne Stroustrup designed a C++ rep  
[IPR](https://github.com/GabrielDosReis/ipr) Internal Program Representation, a type-erased AST for all C++(03) syntax  
(AST: Abstract Syntax Tree).

Fork IPR, take its type representation, make that constexpr, and it's a rep!  
(ITR?) That'd be a whole other challenge!  
Let's start at ⊥ and work up to T.

First, find ways to represent fundamental types, or some subset of them;  
`void`, `int`, `double`, `std::nullptr_t`...

Then, attack the built-in type compounds in increasing level of complexity:

1. `cv`, `pointer`, `reference`
2. `cv`, `pointer`, `reference`, `array`
3. `cv`, `pointer`, `reference`, `array`, `function` (no cvref)
4. `cv`, `pointer`, `reference`, `array`, `function`

Level 1 is a good GotW goal for the meta learner. A flat value-based rep may work.  
Level 2 calls for intermediate TMP to add array extents.  
Levels 2, 3 and 4 bring combinatorial explosion. Tree rep becomes more necessary.

The UDTs, user-defined types, `enum`, `class` and `union`, bring new levels of pain.  
Going into 2020 the challenge looks to reflection for complete UDT decomposition.  
Advanced meta programmers are invited to use whatever methods in the meantime.

Dealing with UDTs means dealing with type names, or some alternative form of type id.  
In designing a rep, aim to make it extensible to work with UDTs.

A common rep would help in sharing test cases and in comparing solutions like-for-like.  
The choice of rep is left open, vague even, and no reference rep is made available yet.  
The rep is part of the challenge; incidental to decomposition, central to traversal.

#### Serialization

Which came first; the type or its serialization?

From the programmer's perspective, the type is typed first in serialized form.  
Then again, the type was already there in the mind of the programmer.  
From the compiler's perspective, deserialization comes first to some AST form.  
Then again, the type vanishes in the compiled output. (Did it ever eggsist?)

For the purposes of this challenge, serialization is just a tree traversal,  
most likely a familiar pre-order traversal, to demonstrate ease of use.

Prefer a simple serialization over typename (see [Down with typename!](#down-with-typename)).  
For instance, `int const* -> Pointer<Const<Funda<int>>> -> "*c_i"`  
(could serialize directly as `"Pointer<Const<Funda<int>>>"`; more verbose, less cryptic)  
(Itanium ABI mangled name is an interesting alternative; compile-time mangling?).  

Then again, in order to test solutions it's useful to have something to test against.  
A typename can be compared against any of the sources listed in Scott's Item 4.  
Also, as noted above, for UDT identification it's hard to avoid the name of the type.

In practice, typename is ingrained, mangled-in wherever UDTs are involved.

The *deserialization* or *parsing* problem is a very optional final challenge `0b11`.  
Usually runtime in practice, it's a fun constexpr conundrum (don't get egg-bound).

### What's the object?

C's type system endures because it maps well to memory and is rooted in registers.  
For cross-language data communication, `extern "C"` linkage is lingua franca.  
The object of the challenge is to reflect on the underlying structure of data.

To make this challenge tractable, the target is to represent [standard layout](https://en.cppreference.com/w/cpp/named_req/StandardLayoutType) types.  
Ignore alignment, etc. - a simple object model with portable representation.

### What is it?

A challenge in the spirit of Herb's [GotW](https://herbsutter.com/gotw/) running ∓20 days over NY 2020:

**20 days** of 'modern' C++ ranging from 'retro' TMP to latest C++2a features,  
**20 days** of 'post-modern' C++ looking ahead to an era of static reflection.

* A personal challenge to learn about types, introspection and meta coding  
* A community challenge towards effective methods of type decomposition

It is open to all and open ended. There's no prepared answer.

A competition? No, no prize. Certainly not a competitive speed-coding exercise.  
Think contemplative meta coding to `begin()`; reflective programming to `end()`.  
For 40 days and 40 nights go contemplate the template & meditate on the meta.

Let's call it a **Meta Retroflective**.

**Type decomposition** is surely *the* archetypal type-based TMP problem.  
The problem is out there, obvious, simply stated and not difficult (in concept)  
(though impossible to complete, down to the devilled details, sans reflection).  
Despite this it appears to be an open problem...

Where are the solutions? (Let the egg hunt begin.)
  
## The Modern C++ type CoDec Challenge

* **Modern**: Use Modern or Post Modern (proposed, preferably implemented) features
* **C++ type**: Any C++ type, at least up to some compound complexity limit
* **CoDec**: Compound-Decompound; a Coder-Decoder for types and their compounds
* **Challenge**: 'Best' - fastest to compile, most complete, shortest, clearest code...

### Challenge`0b0`

Write or describe a **type decoder**:

* **`0b00`** Convert a type to a rep
* **`0b10`** Convert a rep to a type

The rep is *not* the focus here; use the simplest rep that works to show the method  
(the rep should highlight the compound structure of the type see [What's the rep?](#whats-the-rep))  
(`0b10` constrains `0b00` to be compile-time. It is trivial for non type-erased reps).  

### Challenge`0b1`

Design and demonstrate ease of use of a rep in a serialization traversal:

* **`0b01`** Convert a rep to serialized form
* **`0b11`** Convert serialized form to a rep (optional)

Now the rep is chosen or designed for ease of use in compound-type traversal.  
Runtime solutions are acceptable; the aim is to establish ease of use.

### Example valid input-output

<details><summary>Some example sketches with pseudo code (possible spoiler)</summary>

<br>Let rep be a set (possibly hierarchy) of trait-like class templates  
`rep == decompound` that nest to represent the tree of compound types  
(as sketched above):  
```c++
template <typename T> using rep = decompound<T>;
template <typename T> using rep_t = typename rep<T>::type;
```
Let `x = serialize<rep<T>>` and `deserialize(x)` be Challenge`0b1` parts.  
Then, for the second part of both challenges, we should always have:
```c++
2 rep_t<T> -> T
3 deserialize(serialize<rep<T>>) -> rep<T>
```

This leaves the first half of both challenges, parts 0 and 1.  
We will make up an  ascii serialization on the fly; let's call it 'manglish'.

#### Simple built-in type

Pointer to const int;
```c++
using T = int const*;

0 rep<T> -> Pointer<Const<Funda<int>>>
1 serialize<rep<T>> -> "*c_i"
```

#### Tricky built-in type

Array (incomplete) of array of 4 pointers to function (noexcept) returning bool

```c++
using Q = bool()noexcept;
using T = Q*[][4]; // T = bool(*[][4])()noexcept

0 rep<T> -> Array<Pointer<Function<NoExcept,Funda<bool>>,0,4>
1 serialize<rep<T>> -> "[][4]*()F_b"
```

#### User-defined types

```c++
enum E {eh=1,be};
struct S {E a; int b;};
```
For the 20 days before NY 2020, 'shallow' output is valid:
```c++
0 rep<E> -> Enum<E>
0 rep<S> -> Struct<S>
1 serialize<rep<E>> -> "||E"
1 serialize<rep<S>> -> "{}S"
```
For the 20 days after NY 2020, 'deep' output is the target:
```c++
0 rep<E> -> Enum<Id<E>,1,2>
0 rep<S> -> Struct<Id<S>,Enum<Id<E>,1,2>,Funda<int>>
1 serialize<rep<E>> -> "|1,2|E"
1 serialize<rep<S>> -> "{|1,2|E,_i}S"
```
This raises questions of UDT id and how to link back to earlier inline defs.

Note that enum enumerator ids and struct member ids are omitted; this is valid  
(see earlier discussion of [UDTs](#udts-user-defined-types-advanced-optional)).

</details>

### `2020`: A Meta Retroflective

The Meta Challenge runs:

&nbsp;&nbsp;&nbsp;&nbsp;For twenty days before  
&nbsp;&nbsp;&nbsp;&nbsp;and twenty days into  
&nbsp;&nbsp;&nbsp;&nbsp;the twentieth year<sup>0</sup> of  
&nbsp;&nbsp;&nbsp;&nbsp;the twentieth century<sup>0</sup>

(<sup>0</sup> zero-based indexing).

If you participate in private then please star the GitHub repo to indicate numbers.  
Contribute in any way you want or can:

* Discuss or critique the problem itself - be respectful
* Discuss approaches and techniques for solving the challenges
* Add links to resources, libraries, related reading, prior art
* Submit code - to [GitHub repo](https://github.com/willwray/type_codec_challenge) or as online-compiler links
* Provide test cases - lists of types - try to break posted codes
* Benchmark compile-times of posted codes

Channels:

 * slack cpplang: [#type_codec_challenge](https://cpplang.slack.com/messages/type_codec_challenge/) channel
 * reddit /r/cpp [The Modern C++ type CoDec Challenge
](https://www.reddit.com/r/cpp/comments/e9t0y0/the_modern_c_type_codec_challenge/)
 * other channels will be listed here

I will write up the challenge experiences and credit any novel ideas, with permission.

### Use of language features

Solutions can use any features that provide a benefit, including latest C++2a features.  
As 2020 approaches, C++20 features are coming in GCC, Clang and MSVC compilers.  
Use `-std=c++2a` or `/std:c++latest` language support flag for the challenge.  
Use the latest compiler versions. Compiler extensions are fair game.

Classic to Modern to Post Modern progression:

* TMP no traits
* TMP with traits
* constexpr with traits
* constexpr with reflection operators

There's no handicap, advantage or disadvantage, to Classic TMP.  
The aim is to find the objective best Modern methods.

#### Reflection

Language-based reflection is in play going in to 2020.  
Anyone interested to pursue reflection is encouraged read up in the meantime.  
Links to implementations will be posted in Resources around New Year 2020.

### Goals

1. Have fun and learn via personal challenge, community challenge and discourse
2. Find effective solutions for type codec, a useful yet neglected meta problem
3. Gain experience with reflection and its interaction with other language features
4. Feed experience to improve current proposals and help steer language evolution

(The Golden Egg: a true, portable, compile-time type id; `tid`.)

#### Fin

Check [Resources](resources.md) for more.

##### Ack: Thanks to Scott and anonymous reviewers
