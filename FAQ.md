# Frequently Asked Questions

## I have no idea what that challenge is asking for.

"Wot?" is by far the most frequent Q (all 3 reddit posts, more on slack).  
It is asked by beginners and by guru meta-programmer 'language lawyers'.  
It's not a riddle; it's intended to be intriguing, not incomprehensible.  
In concept, it’s not difficult; lack of constraints on what to do makes it so.  
As a 40-day Challenge it's a slow-burner. Week 1 saw 1st code and solution.  
I'll post weekly updates with code and links to help give this Q some As.

The TL/DR at the top is a terse statement:

>Find the most effective, modern way to:   
(0) Decompose a compound type then   
(1) Represent its structure and traverse it

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
