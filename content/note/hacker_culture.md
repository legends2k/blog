+++
title = "Hacker Culture"
description = "Articles resonating with me"
date = "2021-02-18T18:40:53+05:30"
tags = ["tech", "bookmark"]
+++

You can’t learn everything from scratch practically; you need to stand on the shoulder of giants.  These articles were very insightful; either it’s a clear writing that shares my already-held belief or something that reshaped my thinking.  I’ve collected them over the years.  I’ve given short descriptions to most links.

It’s a bookmark page updated once in a while.

# By Authors

## Eric S Raymond

Renowned hacker[^1].  He’s written a lot about Unix philosophy including [_The Art of Unix Programming_][taoup] and [_The Unix Koans of Master Foo_][unix-koans].

* [How to Become a Hacker](http://www.catb.org/esr/faqs/hacker-howto.html): detailed treatise on traits of great hackers
    - [FAQs on Hackers](http://www.catb.org/esr/faqs/): follow-ups on doubts you might have on above
* [How to Ask Questions The Smart Way](http://catb.org/~esr/faqs/smart-questions.html): please read this if you haven’t the experience of asking (good) questions; not just via digital mediums
    - [Short, Self Contained, Correct (Compilable), Example](http://www.sscce.org/): come up with this before asking programming questions
* [Compactness and Orthogonality](http://www.catb.org/esr/writings/taoup/html/ch04s02.html): two very important concepts for programmers and architects

[taoup]: http://www.catb.org/esr/writings/taoup/html/index.html
[unix-koans]: http://www.catb.org/esr/writings/unix-koans/

## Joel Spolsky

Co-founder of StackOverflow.com, Product Manager of _Trello_, Program Manager of _Microsoft Excel_, founder of _Fog Creek_.

* [The Perils of JavaSchools](https://www.joelonsoftware.com/2005/12/29/the-perils-of-javaschools-2/): diluting programming curriculum with _easier_ languages replacing _powerful_ languages, abstracting away the underlying machine, is a bane
* [Things You Should Never Do, Part I](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/): explains why against scrapping a working system and rewriting it from scratch (instead of incremental fixes) is almost always a blunder
* [The Law of Leaky Abstractions](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/): all designs draw the line somewhere and are bound to leak, letting the abstracted show through the abstraction
* [Making Wrong Code Look Wrong](https://www.joelonsoftware.com/2005/05/11/making-wrong-code-look-wrong/): bring all relevant information about a line of code physically as close as possible to improve readability, avoid bugs and increase productivity; talks about Hungarian notation, macros and exceptions too
* [Can your programming language do this?](https://www.joelonsoftware.com/2006/08/01/can-your-programming-language-do-this/): in praise of functional languages and the meta-programming features they sport
* [The Gorilla Guide to Interviewing](https://www.joelonsoftware.com/2006/10/25/the-guerrilla-guide-to-interviewing-version-30/): explains good interviewing process with clear examples and checkpoints
* [Don’t Let Architecture Astronauts Scare You](https://www.joelonsoftware.com/2001/04/21/dont-let-architecture-astronauts-scare-you/): avoid analysis paralysis and get down to real work as the latter (giving tangible output/product) almost always trumps former (giving confusions, ideas but just vapourware)
    - [The Duct Tape Programmer](https://www.joelonsoftware.com/2009/09/23/the-duct-tape-programmer/): similar to above, advocates on getting job done with boring tech that’s easy to maintain instead of overthinking and using niche stuff most wouldn’t want to maintain
* [The Absolute Minimum Every Software Developer Absolutely, Positively Must Know About Unicode and Character Sets (No Excuses!)](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/): minimum concepts you’ve to know about Unicode when working with text

## Jeff Atwood

Co-founder of StackOverflow.com.

* [Inherits Nothing](https://blog.codinghorror.com/inherits-nothing/): advocates composition over inheritance
* [Shall I Call it... SomethingManager](https://blog.codinghorror.com/i-shall-call-it-somethingmanager/): avoid vague names; employing proper conventions help clarifying your code and improve its health
* [The Broken Window Theory](https://blog.codinghorror.com/the-broken-window-theory/): ignoring one issue, and letting it slip by leads to poor software health as it becomes a habit and in no time we’ve lots of issues unattended
* [The Best Code is No Code At All](https://blog.codinghorror.com/the-best-code-is-no-code-at-all/): more code = more bugs; minimalism for programmers
* [The Delusion of Reuse](https://blog.codinghorror.com/the-delusion-of-reuse/): avoid over-engineering in the name of code reuse as it’s seldom done; this is similar to [Write Games, Not Engines](https://geometrian.com/programming/tutorials/write-games-not-engines/index.php)
    - [Stop Future-Proofing Software](https://medium.com/@george3d6/stop-future-proofing-software-c984cbd65e78)
* [Code Smells](https://blog.codinghorror.com/code-smells/): list of code smells -- questionable idioms or patterns -- for easy reference
* [Don't be Afraid to Break Stuff](https://blog.codinghorror.com/dont-be-afraid-to-break-stuff/): encourages programmers to poke around, make mistakes and learn instead of treading softly around code worrying about existing design/architecture breakage; problems of magical thinking
* [Are You a Doer or a Talker?](https://blog.codinghorror.com/are-you-a-doer-or-a-talker/): against analysis paralysis and getting the job done, even if it’s not great
    - [Good Programmers Get Off Their Butts](https://blog.codinghorror.com/good-programmers-get-off-their-butts/): against over planning and how software development is an iterative, not theoretical, process
* [How to Hire a Programmer](https://blog.codinghorror.com/how-to-hire-a-programmer/): programmer interviews should be about real programming; give the design and implementation of a small piece in your current project, discuss result and hire
* [Nobody’s Going to Help You, and That’s Awesome](https://blog.codinghorror.com/nobodys-going-to-help-you-and-thats-awesome/): why self-help books can only give tips but not drastically change anything; self-motivation and elbow grease does; real change comes from within
* [Regex: Now You’ve Two Problems](https://blog.codinghorror.com/regular-expressions-now-you-have-two-problems/): why regular expressions are a good tool but apply it at the wrong place leads to more problems than solutions
* [What’s Your Backup Strategy](https://blog.codinghorror.com/whats-your-backup-strategy/): talks about backup stratgies for a home

## Paul Graham

Co-founder of Y Combinator (the startup seeder) and [_Hacker News_][hn].  His work/writings on [Lisp][] are popular too.

* [The Bus Ticket Theory of Genius](http://paulgraham.com/genius.html): having obsessive interest in something just for its sake, without any gains, distinguishes geniuses from normals
    - [The Value of Learning “Useless” Things](https://www.scotthyoung.com/blog/2020/12/21/knowledge-foundation/) by _Scott H Young_: explains how having interest in random things actually come together when you don’t expect
* [Beating the Averages](http://paulgraham.com/avg.html): how using Lisp when most weren’t gave Viaweb an edge over competitors -- this is a departure from using boring tech which I believe is better, but it’s a nice read

[hn]: https://news.ycombinator.com
[Lisp]: https://en.wikipedia.org/wiki/Lisp_(programming_language)

## Randall Munroe (XKCD)

Super popular web comics on programming.  [explainXKCD.com][] details each comic.

* [Standards](https://xkcd.com/927/): how standards proliferate and doesn’t fully solve the compatibility problem
* [Duty Calls](https://xkcd.com/386/): arguing on the internet (or elsewhere) is pointless and never-ending; we don’t learn from a disagreeing perspective but impose our ideas without listening; programmers strongly believe in what they believe; alternatively, being a keyboard warrior ruins one’s happiness in the real world
* [Real Programmers](https://xkcd.com/378/): parody on how programmers are weighed by their editors than their code and the versatility of Emacs :)
* [Wisdom of the Ancients](https://xkcd.com/979/): parody about landing on a page with the exact question you’re seeking an answer for, with no answers

[explainXKCD.com]: https://www.explainxkcd.com/wiki/index.php

## James Hague

* [Organizational Skills Beat Algorithmic Wizardry](https://prog21.dadgum.com/177.html): how asking mind-bending, hardly used in day-to-day clever algorithm/data-structure/puzzle question is pointless and how organizational skills are more important for programmers; very relatable
* [Do You Really Want to be Doing This When You’re 50?](https://prog21.dadgum.com/154.html): how programming for a profession is neither desirable nor rosy
* [Constantly Create](https://prog21.dadgum.com/99.html): do what’s natural to you and keep creating; don’t try to a perfectionist, volume not perfection; don’t cave in to audience/review pressure, do it for yourself
    - [Why Trying to Be Perfect Won’t Help You Achieve Your Goals (And What Will)](https://jamesclear.com/repetitions) by _James Clear_
    - [The Perils of Future-Coding](https://www.sebastiansylvan.com/post/the-perils-of-future-coding/): most insidious form of tech debt; over-engineering, second-system syndrome, over-generalized code
* [How Did Things Ever Get This Good?](https://prog21.dadgum.com/51.html): just pick the tool of least resistance and get to work instead of wasting time

## Noel Llopis

* [Start Pre-allocating and Stop Worrying](https://gamesfromwithin.com/start-pre-allocating-and-stop-worrying): when game making, stop worrying about memory allocation strategies and start pre-allocating
* [What’s Your Pain Threshold](https://gamesfromwithin.com/whats-your-pain-threshold): build/test times breaking programmer flow
* [Data-Oriented Design (Or Why You Might Be Shooting Yourself in The Foot With OOP)](https://gamesfromwithin.com/data-oriented-design): benefits of DOD
    - [Explaining Data-Oriented Design](http://www.codersnotes.com/notes/explaining-data-oriented-design/): further explanations on DOD

# Misc

* [OMG Ponies!!! (a.k.a Humanity: Epic Fail)](https://codeblog.jonskeet.uk/2009/11/02/omg-ponies-aka-humanity-epic-fail/): funny but meaningful exposition on bad decisions that make life of programmers hard
* [Choose Boring Technology](https://mcfunley.com/choose-boring-technology): choose simple, boring and time-tested tools that’re easy to reason about and maintain than some niche tech only elites know
  - [Orthodox C++](https://gist.github.com/bkaradzic/2e39896bc7d8c34e042b): the subset of C++ that’s better understood and helps with maintainability
  - [Why I Write Games in C (yes, C)](https://jonathanwhiting.com/writing/blog/games_in_c/) gives a clear exposition on why _C_ still has many advantages over other contending languages for a game programmer
  - [One Year of C](https://floooh.github.io/2018/06/02/one-year-of-c.html) talks about a key advantages of C over C++: less anxiety
  - [Follow Boring Advice](http://nywkap.com/other/follow-boring-advice.html) is the meta-concept
* [The Rise of “Worse is Better”](https://www.dreamsongs.com/RiseOfWorseIsBetter.html) argues how “worse-is-better” philosophy (followed by C and Unix) is better than “the-right-thing” (followed by Common Lisp); how former has better survival characteristics, etc.
* [What Color is Your Function](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/): explains how async programming has introduced two classes of functions which can’t be mixed
* [Write Games, Not Engines](https://geometrian.com/programming/tutorials/write-games-not-engines/index.php): writing games, and many of them, gives you enough insights eventually it distil reusable parts into an engine; don’t hurt yourself doing it the other way around
  - [Make Games Not Engines... to learn Engines](https://seanmiddleditch.com/makes-games-not-engines-to-learn-engines/)
* [Teach Yourself Programming in Ten Years](http://norvig.com/21-days.html) - Peter Norvig’s excellent piece on practise
  - [The Forty-Year Programmer](https://codefol.io/posts/the-forty-year-programmer/) - Great insights (and useful tips) from an experienced developer
* [Your Brain on the Internet — Multi-Tasking Research](http://teaching.idallen.com/cst8207/15w/notes/005_this_is_your_brain.html): on the horrors of _continuous partial attention_ due to multi-tasking, smartphones and internet.
* [Essays on programming I think about a lot](https://www.benkuhn.net/progessays/): a page similar to this one
* [The Rise of User-Hostile Software](https://den.dev/blog/user-hostile-software/): explains the rampant problem with proprietary, tracking software crippling the industry


[^1]: in the [right sense][hacker] of the word

[hacker]: https://en.wikipedia.org/wiki/Hacker
