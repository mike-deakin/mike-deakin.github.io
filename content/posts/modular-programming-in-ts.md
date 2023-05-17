---
title: "Modular Programming in Typescript"
date: 2023-03-16T14:08:38+01:00
draft: true
---

The javascript ecosystem has changed a lot since it was first created by ---- in ----. Web projects demand more than just the odd wizzy transition and lazy-loading, node and deno are taking a fair share of the server infrastructure now, and there's a new framework, library or language extension every day. However we still don't really know how to organise our code in a clean and maintainable way. This is increasingly problematic as the growth of JS shows no signs of slowing.
Most of our project structures are born of habbit. Java programmers will instinctively cerate `src/` and `test/` directories, despite modern code bundling handling the co-existence of the files just fine. Others with less grey hair may create `components/` and `pages/` or a `specs/`, `stories/`, etc. inherited from whatever framework is closest to their heart. These are all fine (except the Java one) and come from good ideas, but do they match how your team is thinking about the software?

Imagine you're a new member of your team, and working on a newly reported bug, to do with payment processing let's say. Where do you start looking for the problem code? I'm sure you could tell me right now, or know someone who can, but think about someone with no exposure to your codebase. Even asking someone nearby may not help or at least consumes both your time and theirs. Yes this is a contrived disaster scenario, but any number of these factors could happen and will add wasted time to fixing issues or adding featurs. Which of these file structures would you rather deal with in this hellscape?

