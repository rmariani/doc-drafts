---
navhome: /docs/
sort: 27
next: False
title: Introduction to Sail
---

# Introduction to Sail

Sail is Hoon markup that’s used to render a web page with XML. But what makes
it special is that you can run arbitrary Hoon code within such a web page
without using a separate markup language. In this tutorial, we will
be dealing with Sail as it renders to HTML, a subset of XML.

### Getting Started

Before starting, make sure that your urbit is
[mounted to Unix](https://urbit.org/docs/using/setup/)

Your urbit ship has a built-in mechanism, called the static-site generator, for
taking a Hoon source file and producing an output file as HTML. To this end,
your ship also has a web server that can be found at http://localhost:8080/ if
it’s your first ship that’s running on the a machine, http://localhost:8081/
if it’s the second ship on that same machine, and so on.

That server has access to source files located in `~/<your-urbit>/home/web` and
any sub-folders thereof. For the purposes of this tutorial, we will only use
files that are located immediately in the `~/web/pages` path. It’s possible that
this path does not exist, in which case you must create it.

So, if a Sail source file named sailtest.hoon is saved in your `~/web/pages`
folder on your only running urbit process, you can view a that source rendered
into a web-page at `localhost:8080/pages/sailtest/`.

Any source file that is in `~/web/pages` needs to produce a complete HTML node
-- all Sail must be wrapped in an "HTML" node that must have a "head" and a
"body" as its two children.

### The Basics

It’s easy to see how Sail can directly translate to HTML:

#### Sail code
```
;html
  ;head
    ;title: My first webpage
  ==
  ;body
    ;h1: Welcome!
    ; Hello, world!
    ; We're on the web.
    ;div;
    ;script@"http://unsafely.tracking.you/cookiemonster.js";
  ==
==
```

#### HTML code
```
<html>
  <head>
    <title>My first webpage</title>
  </head>
  <body>
    <h1>Welcome!</h1>
    Hello, world!
    We’re on the web.
    <div></div>
    <script src="http://unsafely.tracking.you/cookiemonster.js"></script>
  </body>
</html>
```

Save the above Sail code in `~/home/web/pages/firstsail.hoon`, and the above
HTML code in `~/home/web/pages/firsth.html`. Access the resulting pages by
navigating to http://localhost:8080/pages/firstsail and
http://localhost:8080/pages/firsth.html, respectively, in your browser. The
resulting pages should be identical!

It shouldn't be hard to see the similarities here. So let's go into more
detail about what the differences are.

### Tags and Closing

In Sail, tag heads are written with the tag name prepended by `;`. Unlike in
HTML, there are different ways on closing tags, depending on the needs of the
tag. One of the nice things about Hoon is that you don’t have to be constantly
closing expressions; Sail inherits this convenience.

#### Empty

Empty tags are closed with a `;` following the tag. Example:

```
;div;
```
Equals:
```
<div></div>
```

#### Filled

Filled tags are closed via line-break. To fill text inside, add `:` after the
tag name, then insert your plain text following a space. Example:

```
;h1: The title
```
Equals:
```
<h1>The title</h1>
```

#### Nested

To nest tags, simply create a new line. Nested tags need to be closed
with `==`, because they expect a list of sub-tags. If we nest a line of plain
text with no tag, a `<p>` is prepended.

Sail, like Hoon, is white-space sensitive. Unlike in Hoon, however, two spaces
(called a "gap") is not necessarily equivalent to a line-break in Sail.

```
;body
  ;h1: Blog title
  This is some good content.
==
```
Equals:
```
<body>
    <h1>Blog title</h1>
    <p>This is some good content.</p>
</body>
```


Conversely, if we want to write a string with no tag at all, then we can prepend
those untagged lines with `;`.

```
;body
  ;h1: Welcome!
  ; Hello, world!
  ; We’re on the web.
==
```
Equals:
```
<body>
    <h1>Welcome~</h1>
    Hello, world.
    We’re on the web.
</body>
```


#### Attributes

Attributes are key-value pairs that go into an HTML node.

Adding attributes is simple: just add the desired attribute between parentheses,
right after the tag name without a space.  We separate different attributes of
the same node by using `,`.

Attributes can be used in two forms: short, which uses one line; and tall, which
uses multiple lines, when a single line would not be practical.


#### Generic

The code below produces a “Submit” button on the page.

Wide-form Sail:

```
;input(type "submit", value "Submit");
```

Tall-form Sail:

```
;input
  =type  "submit"
  =value  "Submit";
==
```
Equals:
```
<input type="submit" value="Submit">`
```

#### IDs
```
;nav#header: Menu
```
Equals:

```
<nav id="header">Menu</nav>
```

Add `#` after tag name to add an ID.

#### Classes

`;h1.text-blue: Title` equals `<h1 class="text-blue">Title</h1>`

Add `.` after tag name to add a class.

#### Image

`;img@"example.png";`
Equals
`<img src="example.png"/>`

Add `@` after tag name to link your source.

#### Linking

`;a/"/urbit.org": A link to Urbit.org`
Equals
`<a href="urbit.org">A link to Urbit.org</a>`

Add `/` after tag name to start an href.

### Hoon in Sail

Our first example is useful to understand how Sail syntax translates to HTML
syntax, but Sail finds its usefulness in the fact that it can be used seamlessly
with Hoon code.

One place where Sail diverges from HTML is that it supports string
interpolation. In Sail, the contents of all tags are a string type called `tape`
unless otherwise designated. Tapes are delineated by the double quotes `""`.
That’s important, because tapes can be interpolated with Hoon expressions,
provided that those expressions produce something a tape as well. Interpolation
is done by wrapping the code to be inserted in `{}`. In your Dojo prompt, try
the command below (without the `>`):

    > "I am a l{(mul 3 11)}t Urbit user."
    ! exit

Oh no! That command produced a nest-fail error, which indicates that there was
a problem related to Hoon type-system. We used the proper `{}` syntax for
interpolation, and we used the proper function-call syntax inside of it. So
what happened?

Trying the `mul` function on its own will help answer this question. Try the
following command in your Dojo:

    > (mul 3 11)
    33

This should work, producing `33`, an atom. That’s our problem. The
interpolation expression expects a tape, but our first command tried to
interpolate with something that results in an atom. Try this now:

    > <(mul 3 11)>
    "33"

We get a similar result, but we know that it’s a tape because it’s wrapped in
`""`. Bingo!  With this knowledge, let’s perform a successful interpolation:

    > "I am a l{<(mul 3 11)>}t Urbit user."
    "I am a l33t Urbit user."

Great. Now let’s apply string interpolation, and other Hoon, in a Sail source
file.

```
/=  gas  /$  fuel:html
=/  show-list  &
=/  ship-type  p.bem.gas
=/  ship
?:  (gth ship-type 255)
  ?:  (gth ship-type 65.535)
    ?:  (gth ship-type 4.294.967.295)
      "comet"
    "planet"
  "star"
"galaxy"
;html
  ;head
    ;meta(charset "utf-8");
    ;title: A more advanced page
  ==
  ;body
  ;h1: This is the Title
    ;a/"/urbit.org": A link to Urbit.org
  ;*  ?:  show-list
      ;ol   :: unordered list; bullet-points
        ;li(style "color: green"): We're doing more interesting stuff, now.
        ;li: We're pretty-printing this sum with Hoon: {<(add 50 50)>}.
        ;li: the code above is shorthand for {(scow %ud (add 50 50))}.
        ;li: I am {(trip '~lodleb-ritrul')}.
        ;li: Actually, my name is {<p.bem.gas>}. I'm a {ship}.
        ==
        ;div; My name is {<p.bem.gas>}. I'm a {ship}.
  ==
==
```

Save the above Sail code in to `home/web/pages/secondsail` and access the
resulting page by navigating to `http://localhost:8080/pages/secondsail` in
your browser.

There are some interesting things here. Let’s go through this code piece by
piece.

```
/=  gas  /$  fuel:html
```

Later in this code we want produce the name of the ship is hosting it, something
that looks like ~lodleb-ritrul. In a generator, we could produce our ship name
by writing `our` -- try it in the Dojo. But `our` is not part of the subject
that a normal Sail page is rendered against. So the above expression uses two
Ford runes, `/=` and `/$`, to add the relevant subject to the current scope. We
can pull the ship’s name out of that scope later.

`/$` is used to get data from the environment, and `/=` puts that in the the
face `gas`. The
[Ford user manual](https://urbit.org/docs/arvo/internals/ford/runes/) has
details on these and other Ford runes, but understanding these runes isn’t
necessary for the purposes of this tutorial.

`fuel:html` gets extra data from your ship’s
[Eyre](https://urbit.org/docs/using/web/) module, which handles all things HTTP,
and sticks it in the subject when Eyre renders a page.

```
=/  show-list  &
=/  ship-type  p.bem.gas
```

Here we are storing the variables used later for conditionals. `&` means `true`,
meaning that any evaluation of `show-list` will be true until changed in the
source code.

`p.bem.gas` is an expression that looks for `p` within `bem` within `gas`. That
wing happens to resolve to the name of the ship that is hosting the webserver.

```
?:  (gth ship-type 255)
  ?:  (gth ship-type 65.535)
    ?:  (gth ship-type 4.294.967.295)
      "Comet"
    "planet"
  "star"
"galaxy"
```

This code chunk is a series of conditionals that checks the host ship’s name to
see what its value is. Ship names are simply another representation of atoms,
and ship categories are different ranges of possible atomic values. Because of
this, we perform greater-than tests to see what kind of ship is running the
webserver.

```
;html
  ;head
    ;meta(charset "utf-8");
    ;title: A more advanced page
  ==
```

Sail proper begins here. These four lines are four nodes. The `;html` node
indicates that the contained code should be rendered as HTML; this tag is closed
on the final line. The `;head` node contains metadata, and is closed three lines
later.  The `;meta` node sets the character set to UTF-8, and the `;title` node
sets the title of the page that shows up in the browser tab.

```
  ;body
  ;h1: This is the Title
    ;a/"/urbit.org": A link to Urbit.org
```

The first line here opens, of course, the body node. The second and third lines
of the above chunk create header text and a hyperlink, respectively.

```
  ;+  ?:  show-list
          ;ol
            ;li(style "color: green"): We're doing more interesting stuff now.
            ;li: We're pretty-printing this sum with Hoon: {<(add 50 50)>}.
            ;li: The code above is shorthand for {(scow %ud (add 50 50))}.
            ;li: I am {(trip '~lodleb-ritrul')}.
            ;li: Actually, my name is {<p.bem.gas>}. I'm a {what-kind}.
          ==
```
Here, we start to use some Hoon expressions again.

On **line 20** we use the `;+` Sail rune, which resolves an expression to a single
node. That expression, in this case, is `?:`, which tests a the truth value of
`show-list`, which we assigned the value `&` (meaning “true”) at the beginning
of the program. It’s closed by the `==` on line 30.

Thus, this program resolves to the first child of `show-list`, the node `;ol`
on **line 21**. That node creates an ordered list, and the items
contained within that node. Items that are worth discussing are the ones that
use `{}`. Recall that these braces, pronounced “lob” and “rob” in Hoon-speak,
allow for the interpolation of code within a string.

**line 22** is the only element of this list that does not use Hoon proper. The
line renders as green, as interpreted by the browser, because of the Sail
attribute.

On **line 23**, interpolated codes produces the sum of 50 plus 50, which is
wrapped in `<>` to put the product in double quotes to make it a tape, like
the rest of the line.

On **line 24**, `{(scow %ud (add 50 50)}` is just a different way writing the
`{<(add 50 50)>}` expression. `scow` is a Hoon function that turns a noun
into a `tape`, Hoon’s string type. `%ud` tells `scow` that its argument should
be displayed in the form of an unsigned decimal.

The code in **line 25** turns a `cord` into a `tape`. Cords are one datatype for
text in Hoon. A cord is just a big atom formed from adjacent unicode bytes, and
are delineated by `''`. `trip` is a function that takes a cord and produces a
tape. All text is treated aso, by converting `~lodleb-ritrul` to a tape, it can
be included in the tape that it is interpolated in.

**Line 26** has interpolated code that uses a wing expression to access the
name of the host ship, and then accesses the face `what-kind` contains a string
describing what type of ship it is. We declared `gas` on the very first line so
that we could add that sort of information to the subject that our Sail is
rendered against. The wing expression `p.bem.gas` looks for `p` within `bem`
within `gas`, which happens to be where the name of the ship is located.

Note how `{what-kind}` does not have a `<>` wrapper. Why do you think this is?

```
        ;div; My name is {<p.bem.gas>}. I'm a {ship}.
  ==
==
```

The node on **line 28** is only rendered, as an alternative to the above list,
if `show-list` is evaluated as true. This means it won’t be shown with your
default code. To show this line, change the `&` on line 2 to `|` (“false”).


### Sail Runes

In the previous example, we used a few special runes that aren’t used in typical
Hoon code. But there are others that are worth learning about at this point.

`;+`     is an expression that resolves to a single node. We saw this on line 20
 in our Sail example above: `;+  ?:  show-list` resolves to the node `;ol` and
 all of that node’s children.

`;* `    is an expression that resolves to a list of nodes. If we want to render
multiple nodes side-by-side, and not just the children of a single node, this
is the expression that we use.

`;=`     turns a tuple of nodes into a list of nodes.


To see how these runes work, let’s take a chunk of our previous code starting at
 line 20 and modify it a bit:

```
  ;*  ?:  show-list
        ;=
        ;li: I am {(trip '~lodleb-ritrul')}.
        ;li: Actually, my name is {<p.bem.gas>}.
        ==
      ;=
    ;div: My name is {<p.bem.gas>}. I'm a {what-kind}.
    ;div: No list here!
      ==
  ==
```

We changed the conditional branches so that each contains two Sail nodes,
side-by-side -- notice that the `;ol` is absent from the first branch.

We begin with the `;*` rune, which resolves to a list of nodes, instead of the
`;+` rune, which only resolves to a single node.

But that’s not enough on its own. `;*` needs to resolve to a list, and a few
Sail nodes aren’t lists until they are explicitly made into lists. Until we wrap
those nodes in `;=`, the nodes treated as mere tuples. All branches must be
valid lists for the `;*  ?:  show-list` expression to resolve.


### Conclusion

You should now have foundational knowledge for making web pages with Sail. There
is, of course, more to learn. If you’re curious, there’s more advanced
resources out there.
The [Ford manual](https://urbit.org/docs/arvo/internals/ford/runes/)
can show you how to access various ship resources for user in your pages. To
learn more about how the renderer works, take a look at the
`~/home/ren/urb.hoon` file inside your urbit.
