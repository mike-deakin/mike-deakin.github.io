---
title: "Why You May Want Types"
date: 2023-07-09T13:42:27+01:00
draft: false
---

The debate around strong vs weak and static vs dynamic types is a long and messy one. I don't expect this post to change that, but I want to offer an alternative opinion. I'll also simplify things a little here, because simply defining strong, weak, static and dynamic can be tricky.

For the duration of this post I'll define a language as "having types" as having staticly checked type declarations and being able to trust them absolutely at runtime. So Javascript, Clojure and Python are out, but Haskell, Typescript and Scala are in.

The typical argument either way for this sort of static typing is safety. When you have types the compiler can guarantee certain aspects of your program, eliminating whole categories of runtime errors with zero performance overhead. The counter point is that good practices like TDD also eliminate these issues whilst providing documentation at the same time, and having to write types just slows developers down.

Neither of these arguments satisfy me very much though. It's a common joke in Haskell that if it compiles then you know your program doesn't have bugs, but is that true? I could write any number of implementations for a function of type `a -> a -> a`, how does the compiler know that its correct? (There's a reason QuickCheck exists after all).
Conversely, Haskell is indeed slower to develop in than LISPs, but is that the fault of types or of the alien syntax and cryptic mathematical terminology?

Having said all that, I enjoy writing code in languages with types. Not because they're documentation, or because they guarantee something about the code, but because they guarantee something about the documentation. Comments - like doc files - go out of date and tests are very easy to make opaque and can live a long way away from the implementation - types don't suffer these problems.

Every time you write code in a language with types you have at your disposal information about what is and isn't possible. Every time you compile that code you have more than [40 years of computer science](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system) proving that information is correct and consistent. The stronger the coherence with the domain your working in, the stronger the promise and correctness.

Because types are interpreted by the compiler prior to runtime, all that information, documentation, and correctness can be used by all your developer tools. IDEs are great nowadays and can call on any number of language assurances to assist on the way to a valid program.

Between language with and without types, my exeprience with LSP servers is simply better when they have types at their disposal. For example, when working with javascript I often feel like code completion suggestions are just guesses. Developers are smart, so they're usually pretty good guesses, but I rarely trust what they say. Typescript however, which functionally is the same language but extended to have types, offers fewer and more relevent suggestions.

Ultimately types help (though are not the only factor) to streamline my workflow in many cases. Their self-documenting and code-assisting properties help me to stay in flow of the problem at hand for longer on average. If I know what I'm doing (simple or well-trodden domains) or I don't care for reliability or correctness (quick scripts, toys, etc) then I'll probably stick to a type-less language to stop butting heads with the compiler so much. I encourage you to do the same, instead of looking for some universal truth about whether types are good.
