# ASCIIToSVG ################################################################

                      .-------------------------.
                      |                         |
                      | .---.-. .-----. .-----. |
                      | | .-. | +-->  | |  <--| |
                      | | '-' | |  <--| +-->  | |
                      | '---'-' '-----' '-----' |
                      |  ascii     2      svg   |
                      |                         |
                      '-------------------------'
					   https://9vx.org/~dho/a2s/

## Introduction #############################################################

### License #################################################################

ASCIIToSVG is copyright © 2012 Devon H. O'Dell and is distributed under an
awesome 2-clause BSD license, which is included in the source file.

### What does it do? ########################################################

ASCIIToSVG is a pretty simple PHP library for parsing ASCII art diagrams and
converting them to a pretty SVG output format.

### But... why? #############################################################

There are a few reasons, mostly to do with things I really like:

 * I like markdown a lot. If I could write everything in markdown, I
   probably would. 
 * I like ASCII art a lot. I've been playing around with drawing ASCII art
   since I was both a wee lad and a BBS user.
 * I like pretty pictures. People say they're worth a thousand words.

So I thought, "What if I could combine all these things and start writing
markdown documents for my technical designs -- complete with ASCII art
diagrams -- that I could then prettify for presentation purposes?"

### Aren't there already things that do this? ###############################

Well, yes. There is a project called [ditaa][1] that has this functionality.
Sort of. But it's written in Java and it generates PNG output. I needed
something that would generate scalable output, and would be a bit easier to
integrate into a wiki (like the one in [mtrack][2], which we use at work).
We got it integrated, but I ran into a few bugs and rendering nits that I
didn't like. Additionally, it takes a long time to shell out the process
and JVM to generate the PNG. If you're doing inline edits on an image, you
can imagine that this gets to be a heavy load on both the client and
server, just from a data transfer perspective alone.

So I reimplemented it in PHP. There are some things it can do better, and
I'm aware of a few rendering bugs already, but so far it works better and
is more extensible than ditaa, and I'm enjoying it more.

## Using ASCIIToSVG #########################################################

### Command Line ############################################################

Currently, there is no CLI consumer for ASCIIToSVG. This is a rather trivial
task, so I imagine I'll write one at some point soon, but currently you'll
have to write a quick script around the API.

### Class API ###############################################################

There are plenty of comments explaining how everything works (and I'm sure
several bugs negating the truth of some of those explanations) in the source
file. Basic flow through the API is:

    :::php
    $asciiDiagram = file_get_contents('some_pretty_art');
    $o = new ASCIIToSVG($asciiDiagram);
    $o->setDimensionScale(9, 16);
    $o->parseGrid();
    file_put_contents('some_pretty_art.svg', $o->render());

I'll put more detailed information in the [wiki][3] and this `README` as I
have time and inclination to do so.

## How Do I Draw? ###########################################################

Enough yammering about the impetus, code, and functionality. I bet you want
to draw something. ASCIIToSVG supports a few different ways to do that.

### Basics: Boxes and Lines #################################################

Boxes can be polygons of almost any shape (don't try anything too wacky
just yet; I haven't). Lines are supported on a horizontal / vertical axis
(diagonals are definitely being considered so that I can fully support
[App::Asciio][4]'s output). Edges of boxes and lines can be drawn by using
the following characters:

 * `-` or `=`: Horizontal lines (dash, equals)
 * `|` or `:`: Vertical line (pipe, colon)
 * `*`: Line of ambiguous direction (asterisk)

Eventually, the `:` and `=` edges will support dashing lines, unless I come
up with a better way to stylize lines.

To draw a box or turn a line, you need to specify corners. The following
characters are valid corner characters:

 * `+`: Angular corner connector (plus)
 * `/` and `\`: Quadratic Bézier corners (forward slash, backslash)
 * `'` and `.`: Also quadratic Bézier corners, but for tighter turns
   (apostrophe, period)

A simple box with a line pointing at it:

    +----------+
    |          | <---------
    +----------+

Oh yes, that brings me to markers. Markers can be attached at the end of a
line to give it a nice arrow by using one of the following characters:

 * `<`: Left marker (less than)
 * `>`: Right marker (greater than)
 * `^`: Up marker (carat)
 * `v`: Down marker (letter v)

### Basics: Text ############################################################

Text can be inserted at almost any point in the image. Text is rendered in
a monospaced font. There aren't many restrictions, but obviously anything
you type that looks like it's actually a line or a box is going to be a good
candidate for turning into some pretty SVG path. Here is a box with some
plain black text in it:

    +-------------------------------------.
    | Hello here and there and everywhere |
    '-------------------------------------+

Oh, also that box has two sweet rounded edges. I bet you can't guess which
ones.

### Basics: Formatting #####################################################

It's possible to change the format of any boxes / polygons you create. This
is done (true to markdown form) by providing a reference on the top left
edge of the object, and defining that reference at the bottom of the input.
Let me reiterate that the references *must* be defined at the *bottom* of
the input. An example:

    /-------------\  .--------------.
    |[1]Red Box   |  |[2]Blue Box   |
    \-------------/  '--------------'

    [1]: {"fill":"#aa4444"}
    [2]: {"fill":"#ccccff"}

(This time we curved all the edges!)

Text appearing within a stylized box automatically tries to fix the color
contrast if the black text would be too dark on the background. The
reference commands can take any valid SVG properties / settings for a
[path element][5]. The commands are specified in JSON form, one per line.
Reference commands do not accept nested JSON objects -- don't try to
place additional curly braces inside!

This method is (in my opinion) much nicer than the one provided by ditaa:

 * It eats up less space.
 * It fits in smaller boxes.
 * It allows more customization.
 * It's easier to see what formatting changes happened in diffs.
 * It's easier to read any text nested in the box itself.

But that's just me. If you have thoughts on how to do this for lines,
please do let me know.

### Basics: Special Objects #################################################

There was some pretty nifty functionality in ditaa for special box types.
The ones I was most interested in were the `storage` and `document` box,
though I recently learned there are more than are advertised on the website.
To do this, one adds an `a2s:type` option to the box and specifies one of
the supported object types:

 * `storage`: The standard "storage" cylinder.
 * `document`: The standard document box with a wavy bottom.

Special objects are implemented as closed SVG paths in a 100x100 box and are
scaled on-demand. Support for external SVG paths for additional object types
would be a nifty feature to add at some point in the future.

## References ###############################################################
[1]: http://ditaa.sourceforge.net/ "ditaa - DIagrams Through ASCII Art"
[2]: http://mtrack.wezfurlong.org/ "mtrack project management software"
[3]: https://bitbucket.org/dhobsd/asciitosvg/wiki/Home "a2s wiki page"
[4]: http://search.cpan.org/dist/App-Asciio/lib/App/Asciio.pm "App::Asciio"
[5]: http://www.w3.org/TR/SVG/paths.html "SVG Paths"
