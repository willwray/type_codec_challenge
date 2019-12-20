# Frequently Asked Questions

## I have no idea what that challenge is asking for.

"Wot?" is the most frequent Q, asked by beginners and by meta-gurus.  
The TL/DR gives a terse statement (expanded in the full statement):

>Find the most effective, modern way to:   
(0) Decompose a compound type then   
(1) Represent its structure and traverse it

The first step is to gain a good understanding of the [compound types](https://stackoverflow.com/a/52251200/7443483).

Challenge 0 is to find ways to remove all layers of compounding from an  
arbitrary type and to evaluate which is the most effective, modern method.

Starter 'shell' code has been posted. You implement a 'decompound' method  
to pass some tests. First solutions have been posted. Regular updates will  
be posted in the GitHub wiki. If you have more Q's then ask in any channel.  
Challenge 0 is good way to learn more about types and meta-programming.  
It is not difficult, in concept. It is hard to take to completion.  
The classic TMP solutions are well known and published since circa 2000.  
It is an open problem to find the most effective, modern method.

Challenge 1 is to seek a way to represent the compound structure of a type  
that is better - more efficient and/or easier to use - than the type itself.

If you're at "Wot?" then skip this; use some simple rep to serve Challenge 0.  
The lack of constraints on what to do here makes Challenge 1 more difficult.  
It is open ended, a chin-strokin' slow-burner. Light a pipe. Retroflect.

## In the context of this challenge, what is a compound type,<br>and what do you mean by "traverse it"?

Here's a snippet from the cppreference on [std::is_compound](https://en.cppreference.com/w/cpp/types/is_compound) type_trait:

> If T is a compound type (that is, array, function, object pointer, function pointer,   
member object pointer, member function pointer, reference, class, union, or enumeration,   
including any cv-qualified variants)

Howard Hinnant's map of the [C++ type system](http://howardhinnant.github.io/TypeHiearchy.pdf), in Resources, is a good overview.

A bit like chemical compounds are built up from atoms,  
compound types are built up from the [fundamental](https://en.cppreference.com/w/cpp/language/types) types.

The problem is to take a type and strip all its layers of compounding,  
while 'memoing' the structure to some intermediate representation "rep".

The rep can then be replayed, or 'traversed', in different ways.  
The structure is a tree; tree traversal is the computer-science term for it.

## why is the "rep" even needed in your challenge?

This is a Jedi level Q; from "Wot?" to "Why?" (or, "wot for?").

1. The idea is to decouple type decomposition from subsequent use.  
For the Challenge, this helps highlight the decomposition method.

2. Single responsibility principle. Mixing the method of decomposition   
with usage constrains both to follow the same traversal order.

3. Decompounding a type is work; a rep that is more efficient, or easier  
to traverse, can save work if used multiple times, e.g. if:
```
DecompoundType2Rep + 2x(TraverseRep & Use) < 2x(DecompoundType & Use)
```

4. This is a 'Retroflective': imagine the possibilities a rep can open.  
Bjarne & Gaby had a vision for IPR; rep is a subset of that.

5. Them's the rules! (As revealed to me in a series of waking dreams.)
