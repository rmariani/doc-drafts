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
taking a Hoon (or other) source file and producing an output file as HTML.

To host that output, your ship also has a web-server, called
[Eyre](https://urbit.org/docs/using/web/), that can be found at
http://localhost:8080/ if it’s your first ship that’s running on the a
machine, http://localhost:8081/ if it’s the second ship on that same machine,
and so on.

The `/web` directory of the `%home` desk is served by default. You can change
which desk you're serving by using the command `|serve` in the Dojo. If you
have already created and mounted a desk named `%sandbox`, for example, you can
switch to serving it with this command:

```
|serve %sandbox
```

The first time you serve a desk, `/web` will be included as default. After that,
you will need to explicitly append `/web` to the desk you're switching to --
otherwise, you'll need to write http://localhost:8080/web every time you
navigate to your web-server. Newly served desks not automatically having `/web`
as their root directory is a bug.

To switch back to your `%home` desk and set `/web` as your root directory, run:

```
|serve %/web
```

We'll be serving the `%home` desk for the examples in this tutorial.

#### Rendering

That server has access to source files located in `/<your-urbit>/home/web` and
any sub-folders thereof. For the purposes of this tutorial, we will only be
using files that are located in the `/<your-urbit>/home/web/pages` path.

There is an important distinction between these two paths. According to
`/ren/urb.hoon`, any file inside of `/web` or any of its sub-directories
is put through a tree of renderers specified at `/ren/urb/tree.hoon`
-- *except* for files inside of `/web/pages`. All of the files in `/web` and
its non-`/pages` children directories have various operations on files, such as
wrapping your Sail the `html`, `head`, and `body` nodes. This rendering tool,
called simply **Tree**, also adds certain formatting polish, such as a sidebar
that lists sibling files in the same directory.

The `/web/pages` sub-directory is a unique case with its own rendering rules.
The Tree is not used when rendering files in this path. The existence of no tag
is inferred, and no special formatting is applied. Such simplicity is ideal for
introducing the basics of Sail. Any source file located in `/web/pages` must
produce a complete HTML node with "head" and "body" nodes as children.

Understanding the details of the rendering pipeline is not necessary to complete
this tutorial. But, if you are curious, you might find it illuminating to
explore the contents of the `/ren` directory.

So, if a Sail source file named `sailtest.hoon` is saved in your `/web/pages`  
folder on your only running urbit process, you can view that source rendered
into a web-page at `localhost:8080/pages/sailtest/`.

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

Save the above Sail code in `/home/web/pages/firstsail.hoon`, and the above
HTML code in `/home/web/pages/firsth.html`. Access the resulting pages by
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

Attributes can be used in two forms: flat, which uses one line; and tall, which
uses multiple lines, when a single line would not be practical. Flat forms and
flat forms are two syntaxes of semantically equivalent expressions.


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

`;a/"urbit.org": A link to Urbit.org`
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

Great!

### The Subject

The default subject of a Sail file is different from the subject that
expressions in the Dojo or a Hoon generator are evaluated against.

To see this difference, first save the code below to a file at
`home/web/pages/sub.hoon` and view the rendering at
http://localhost:8080/pages/sub.

```
;html
  ;head;
  ;body
    {<.>}
  ==
==
```

Then enter the following command in the Dojo:

    > .

You'll notice that the Sail subject is like a stripped-down version of the Dojo
subject. Its final line should match the Dojo subject.

Now swap the code below into your Sail file.

```
/=  gas  /$  fuel:html
;html
  ;head;
  ;body
    {<.>}
  ==
==
```

You'll find that the resulting Sail subject is much larger. If you look closely,
you'll see that there's some new similarities with the Dojo subject, such as
containing the name of your ship. There's also some new differences, since
`fuel:html` contains things that are specifically useful for performing
web-related operations.

### A More Interesting Example

Now let’s apply knowledge of string interpolation and the Sail subject in a Sail
source file. The following code produces more interesting things than the
example we saw before.

```
/=  gas  /$  fuel:html
=/  show-list  &
=+  wid=(met 3 p.bem.gas)
=/  what-kind
?:  (lte wid 1)   "galaxy"
?:  =(2 wid)      "star"
?:  (lte wid 4)   "planet"
?:  (lte wid 8)   "moon"
  "comet"
;html
  ;head
    ;meta(charset "utf-8");
    ;title: A more advanced page
  ==
  ;body
  ;h1: This is the Title
    ;a/"http://urbit.org": A link to Urbit.org
    ;+  ?:  show-list
        ;ol
            ;li(style "color: green"): We're doing interesting stuff now.
            ;li: We're pretty-printing this sum with Hoon: {<(add 50 50)>}.
            ;li: The code above is shorthand for {(scow %ud (add 50 50))}.
            ;li: I am {(trip '~lodleb-ritrul')}.
            ;li: Actually, my name is <p.bem.gas>. I'm a {what-kind}.
          ==
        ;div: My name is {<p.bem.gas>}. I'm a {what-kind}.
  ==
==                                                                   
```

Save the above Sail code in to `home/web/pages/secondsail.hoon` and access the
resulting page by navigating to `http://localhost:8080/pages/secondsail` in
your browser.

There are some interesting things here. Let’s go through this code piece by
piece.

```
/=  gas  /$  fuel:html   
```

Later in this code we want produce the name of the ship is hosting it, something
that looks like ~lodleb-ritrul. In a generator, we could produce our ship name
by writing `our` -- try it in the Dojo. But remember that Sail is rendered
against its own subject, `our` is not part of the default subject. To augment
our subject with the information that we need, we use  `/=  gas  /$  fuel:html`.

The above expression uses two Ford runes, `/=` and `/$`, to add the relevant
information to the current subject. We can pull the ship’s name out of this
augmented subject later. It's important to note that these runes are not part of
Hoon. They are part of Ford, our build system.

`/$` is used to get data from the environment, and `/=` adds that data to
subject in the face `gas`. The
[Ford user manual](https://urbit.org/docs/arvo/internals/ford/runes/) has
details on these and other Ford runes, but understanding these runes isn’t
necessary for the purposes of this tutorial.

`fuel:html` gets extra data from your ship’s Eyre module, which handles all
things HTTP, and sticks it in the subject when Eyre renders a page. This
includes the name of the ship, which is what we look for in the next line, but
it includes much more.

`p.bem.gas` is an expression that looks for `p` within `bem` within `gas`, the
face that we stored the above `fuel:html` expression in. That wing happens to
resolve to the name of the ship that is hosting the web-server. If we wanted to
access the current desk, we would use the wing expression `q.bem.gas`.

You can take a look at what you can access by typing `fuel:html` in the Dojo,
and then explore it by modifying your wing expression in your Sail file
accordingly.

```
=/  show-list  &                                                    
```

The above line of code stores a variable used later for a conditional. `&` means
`true`, meaning that any evaluation of `show-list` will be true until changed in
the source code.

```
=+  wid=(met 3 p.bem.gas)
=/  what-kind                                                             
  ?:  (lte wid 1)   "galaxy"                                               
  ?:  =(2 wid)      "star"                                                 
  ?:  (lte wid 4)   "planet"                                               
  ?:  (lte wid 8)   "moon"                                                 
  "comet"    
```

This code chunk is a series of conditionals that checks the host ship’s name to
see what its value is.

The first line, `=+  wid=(met 3 p.bem.gas)`, combines a new noun with the
subject and gives it the face `wid`. This noun is produced by the the
standard-library function [`met`](../library/2c), which measures the number of
[blocks](../library/1c/) of size `a` within `b`. Blocks are units that have a
bitwidth of `2^a`. So, in this case, it measures how many bytes (blocks of size
3 are bitwidths of 8; `2^3 = 8`) are fully contained in your ship's address.

Galaxies contain zero to one 3-blocks; stars contain two 3-blocks; planets
contain three to four 3-blocks; moons contain five to eight 3-blocks; comets
contain nine to sixteen 3-blocks. Each test uses `lte`, the
less-than-or-equal-to function, to determine if `wid` is of a given size. The
result of this series of tests is added to the subject and given the face
`what-kind`.

Ship names are simply another representation of atoms, and ship
categories are different ranges of possible atomic values. Because of this, we
perform less-than tests to see what kind of ship is running the web-server.

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
    ;a/"http://urbit.org": A link to Urbit.org
```

The first line here opens, of course, the body node. The second and third lines
of the above chunk create header text and a hyperlink, respectively.

In the following lines, we beginning to use some Hoon expressions again.

```
  ;+  ?:  show-list
          ;ol
```

In the top line in the chunk above, we use the `;+` Sail rune, which
resolves an expression to a single node. That expression, in this case, is
`?:`, which tests the truth value of `show-list`, which we assigned the value
`&` (meaning “true”) at the beginning of the program. It’s closed by the `==` on
the last line.

Thus, this program resolves to the first child of `show-list`, the node `;ol`.
That node creates an ordered list, and the items contained within that node.

```
            ;li(style "color: green"): We're doing more interesting stuff now.
```
The line above  is the only element of this list that does not use
Hoon proper. The line renders as green, as interpreted by the browser,
because of the Sail attribute.

The next items in the list use `{}`. Recall that these braces, pronounced “lob”
and “rob” in Hoon-speak, allow for the interpolation of code within a string.
```
            ;li: We're pretty-printing this sum with Hoon: {<(add 50 50)>}.
```
The line above is interpolated with Hoon code that produces the sum of 50 plus
50, which is wrapped in `<>` to put the product in double quotes to make it a
tape, like the rest of the line.


```
            ;li: The code above is shorthand for {(scow %ud (add 50 50))}.
```
The `{(scow %ud (add 50 50)}` on this line is just a different way
writing the `{<(add 50 50)>}` expression. `scow` is a Hoon function that turns a
noun into a `tape`, Hoon’s string type. `%ud` tells `scow` that its argument
should be displayed in the form of an unsigned decimal.

```
            ;li: I am {(trip '~lodleb-ritrul')}.
```

`trip` is a function that takes a `cord` and produces a `tape`. A piece of
data of the cord type is one big atom formed from adjacent unicode bytes and
delineated by `''`. Remember that all text rendered in Sail  is treated as a
tape. So, by converting `~lodleb-ritrul` to a tape, it can be included in the
tape that we are trying to insert it into without getting a type error.

```
            ;li: Actually, my name is {<p.bem.gas>}. I'm a {what-kind}.
          ==
```

The line above has interpolated code that uses a wing expression to access the
name of the host ship, and then accesses the face `what-kind` contains a string
describing what type of ship it is. We declared `gas` on the very first line so
that we could add that sort of information to the subject that our Sail is
rendered against. The wing expression `p.bem.gas` looks for `p` within `bem`
within `gas`, which happens to be where the name of the ship is located.

Note that `{what-kind}` does not have a `<>` wrapper. Why do you think this is?

```
        ;div; My name is {<p.bem.gas>}. I'm a {ship}.
  ==
==
```

The `;div;` node above is only rendered, as an alternative to the above list,
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
`/home/ren/urb.hoon` file inside your urbit.
