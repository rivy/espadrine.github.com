# Aulx: The Tricks

Aulx is an autocompletion engine for the Web. It is ongoing work, but you can look at it [here](https://github.com/espadrine/aulx) and try it [there](http://espadrine.github.io/aulx/). Here, I am going to focus on JS autocompletion, and talk about two specific tricks I implemented.

# The Contextualizer

One of the challenges of writing an autocompletion system is that we need to learn from partially written code, since it is still being edited. A common approach is to write a separate parser that is able to recover from missing parentheses, unmatched braces, any incorrect syntax. For example, Tern's Acorn parser relies on indentation to derive the correct braces. This costs some performance in the parser, and there is no "silver bullet" implementation, no royal algorithm that works intuitively in all cases.

Falling that, I decided to do the obvious heuristic. It turns out that, in most of the code you write, newlines are significant. They mean that you are going from one procedural statement to the next. Even when they are mid-statement, your code probably is readable by a simple parser when you write the code. If your code really isn't parseable, we simply keep the latest information. Then, we keep it up-to-date when we can.

However, given that, how would you know where the cursor is? How would you determine when to suggest what, without an abstract syntax tree? For that purpose, I create a contextualizer which only relies on a lexer. Based on the list of tokens produced, we can find the position of the cursor. Then, depending on which token precedes it, we obtain all the information we would need. We know when we have dot completion, we know all the identifiers defining the successive properties to suggest from, and so on. A remarkable advantage of this technique is that you can figure out where the cursor is ("contextualize") even with very improper code, and give some good suggestions nonetheless. Indeed, the lexer will succeed with any code which the error-tolerant JS parser can eat, and then some.

So, I wrote Esprima's lexer. The one, famous pain point there is that in JS, it is notably hard to discriminate the use of `/` (slash) as a division symbol and as a regular expression marker. The usual method is to rely on the parser, running on the side, for hints. That, however, means that we need to run the parser when running the lexer, which both has a performance cost, and may throw if the code isn't syntactically correct. Fortunately, Tim Disney from Mozilla [discovered](http://disnetdev.com/blog/2012/12/20/how-to-read-macros/) a way to properly lex code without the use of a parser. I implemented his algorithm in Esprima, and it is now merged upstream.

That said, the lexer has a performance cost of itself. I wanted to minimize that as much as possible. It turns out that, again, the idea that JS code has implicitly significant newlines turns out to be great. Indeed, that has an impact on the tokenizing rules. Newlines are not allowed in single-line comments, nor in regular expressions, which means that I could cut corners quite a bit. In the end, I proved that we only really need to lex the line the cursor is on, and go through a "shortcut" algorithm for all the code leading to that line. That shortcut only needs to keep track of string literals and multiline comments. Indeed, the productions that can include a line terminator are MultilineNotAsteriskChar, MultilineNotForwardSlashOrAsteriskChar, LineContinuation (for strings), and InputElementDiv and InputElementRegExp, those last two being only used to distinguish between the `/` for division and for regular expressions, which we are not concerned with. That makes the contextualizer remarkably fast in common situations.

# Type as in Interface

Another thing I wanted to try was a design that absorbed information wherever it could. Useful data, I reasoned, is twofold. One may either wish to search for a specific property / variable name / symbol, to type it faster, or may wish to look up all properties available in an object, for exploratory purposes. Never underestimate the power of exploratory programming. There's always something you stumble upon and that will make you go "heh, that looks sweet!" The other day, I noticed that Firefox had `navigator.mozSms`, on the desktop version. I looked it up, it turns out that it has been deprecated in favor of `navigator.mozMobileMessage`. And boom, you find [this spec draft](http://www.w3.org/2012/sysapps/messaging/) dealing with mobile messaging, with editors from Telefonica and Intel. Fumbling through APIs with your eyes wide open can give you accidental ideas, and refresh your memory. Hey, I had forgotten about `navigator.onLine`!

As a language tool, you can either be very strict about types, and require a lot of written type information from the developer with the implicit promise that this strictness will bring her more tooling information, or you can eat every piece of information you find lying on the floor, and use that. Aulx' approach is the hoover one.

Of course, no matter what approach you take, you need to organize the information you aggregate, and for that purpose, in JS, you benefit from having some concept of types. In my case, I gather data from every function call, from every assignment, from every property access. Therefore, I have defined my types as being the *union of all information related to functions*. If a variable was used as a second parameter to a function, that is stored in that variable's type. If we learned that it is an instance of some constructor, that too is stored. Then, on the other side of the fence, each function accumulates information about properties hold by each of their parameter, by their return value, and by their instances (if it is a constructor). That way, when completing properties, all this information is combined to draw the largest and most accurate picture we can.

# Not just JS

Aulx is meant for the Web, not just JS. That was the goal from the start. It now has a world-class CSS autocompletion system, which Girish Sharma of [Firefox fame](https://hacks.mozilla.org/2013/08/new-features-of-firefox-developer-tools-episode-25/) contributed. Expect work on HTML autocompletion too. Imagine receiving autocompletion even for inlined JS and CSS! Imagine combining that information so as to provide some DOM / CSS information in your JS!

<script type="application/ld+json">
{ "@context": "http://schema.org",
  "@type": "BlogPosting",
  "datePublished": "2013-08-20T19:41:00Z",
  "keywords": "js" }
</script>
