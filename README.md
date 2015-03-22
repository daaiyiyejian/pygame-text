# pygame-text

This module simplifies drawing text with the pygame.font module. Specifically, the `ptext` module:

* handles the `pygame.font.Font` objects.
* handles the separate step of generating a `pygame.Surface` and then blitting it.
* caches commonly-used Surfaces.
* handles word wrap.
* provides more fine-grained text positioning options.
* provides a few special effects: outlines, drop shadows, gradient fill, and transparency.

## Quick usage examples

	ptext.draw("Text color", (50, 30), color="orange")
	ptext.draw("Font name and size", (20, 100), fontname="fonts/Boogaloo.ttf", fontsize=60)
	ptext.draw("Positioned text", topright=(840, 20))
	ptext.draw("Allow me to demonstrate wrapped text.", (90, 210), width=180, lineheight=1.5)
	ptext.draw("Outlined text", (400, 70), owidth=1.5, ocolor=(255,255,0), color=(0,0,0))
	ptext.draw("Drop shadow", (640, 110), shadow=(2,2), scolor="#202020")
	ptext.draw("Color gradient", (540, 170), color="red", gcolor="purple")
	ptext.draw("Transparency", (700, 240), alpha=0.1)
	ptext.draw("Vertical text", midleft=(40, 440), angle=90)
	ptext.draw("All together now:\nCombining the above options",
		midbottom=(427,460), width=360, fontname="fonts/Boogaloo.ttf", fontsize=48,
		color="#AAFF00", gcolor="#66AA00", owidth=1.5, ocolor="black", alpha=0.8)

## To install

Download `ptext.py` and put it in your source directory. To install from command line:

    curl https://raw.githubusercontent.com/cosmologicon/pygame-text/master/ptext.py > my-source-directory/ptext.py

## Detailed usage

`ptext.draw` requires the string you want to draw, and the position. You can either do this by
passing coordinates as the second argument (which is the top left of where the text will appear), or
use the positioning keyword arguments (described later).

	ptext.draw("hello world", (20, 100))

`ptext.draw` takes many optional keyword arguments, described below.

The `ptext` module also has many module-level globals that control the default behavior. Please set
these to your desired defaults.

## Font name and size

	ptext.draw("hello world", (100, 100), fontname="fonts/Viga.ttf", fontsize=32)

Keyword arguments:

* `fontname`: filename of the font to draw. Defaults to `ptext.DEFAULT_FONT_NAME`, which is set to
`None` by default.
* `fontsize`: size of the font to use, in pixels. Defaults to `ptext.DEFAULT_FONT_SIZE`, which is
set to `24` by default.
* `antialias`: whether to render with antialiasing. Defaults to `True`.

If you don't want to specify the whole filename for the fonts every time, it can be useful to set
`ptext.FONT_NAME_TEMPLATE`. For instance, if your fonts are in a subdirectory called `fonts` and all
have the extension `.ttf`:

	ptext.FONT_NAME_TEMPLATE = "fonts/%s.ttf"
	ptext.draw("hello world", (100, 100), fontname="Viga")  # Will look for fonts/Viga.ttf

`fontname=None` always refers to the system font.

## Color and background color

	ptext.draw("hello world", (100, 100), color=(200, 200, 200), background="gray")

Keyword arguments:

* `color`: foreground color to use. Defaults to `ptext.DEFAULT_COLOR`, which is set to `"white"` by
default.
* `background`: background color to use. Defaults to `ptext.DEFAULT_BACKGROUND`, which is set to
`None` by default.

`color` (as well as `background`, `ocolor`, `scolor`, and `gcolor`) can be an (r, g, b) sequence
such as `(255,127,0)`, a `pygame.Color` object, a color name such as `"orange"`, an HTML hex color
string such as `"#FF7F00"`, or a string representing a hex color number such as `"0xFF7F00"`.

`background` can also be `None`, in which case the background is transparent. Unlike
`pygame.font.Font.render`, it's generally not more efficient to set a background color when calling
`ptext.draw`. So only specify a background color if you actually want one.

Colors with alpha transparency are not supported. See the `alpha` keyword argument for transparency.

## Positioning

	ptext.draw("hello world", centery=50, right=300)
	ptext.draw("hello world", midtop=(400, 0))

Keyword arguments:

	top left bottom right
	topleft bottomleft topright bottomright
	midtop midleft midbottom midright
	center centerx centery

Positioning keyword arguments behave like the corresponding properties of `pygame.Rect`. Either
specify two arguments, corresponding to the horizontal and vertical positions of the box, or a
single argument that specifies both.

If the position is overspecified (e.g. both `left` and `right` are given), then extra specifications
will be (arbitrarily but deterministically) discarded. For constrained text, see the section on
`ptext.drawbox` below.

## Word wrap

    ptext.draw("splitting\nlines", (100, 100))
    ptext.draw("splitting lines", (100, 100), width=60)

Keyword arguments:

* `width`: maximum width of the text to draw, in pixels. Defaults to `None`.
* `widthem`: maximum width of the text to draw, in font-based em units. Defaults to `None`.
* `lineheight`: vertical spacing between lines, in units of the font's default line height. Defaults
to `1.0`.

`ptext.draw` will always wrap lines at newline (`\n`) characters. If `width` or `widthem` is
set, it will also try to wrap lines in order to keep each line shorter than the given width. The
text is not guaranteed to be within the given width, because wrapping only occurs at space
characters, so if a single word is too long to fit on a line, it will not be broken up. Outline and
drop shadow are also not accounted for, so they may extend beyond the given width.

You can prevent wrapping on a particular space with non-breaking space characters (`\u00A0`).

## Text alignment

    ptext.draw("hello\nworld", bottomright=(500, 400), align="left")

Keyword argument:

* `align`: horizontal positioning of lines with respect to each other. Defaults to `None`.

`align` determines how lines are positioned horizontally with respect to each other, when more than
one line is drawn. Valid values for `align` are the strings `"left"`, `"center"`, or `"right"`, a
numerical value between `0.0` (for left alignment) and `1.0` (for right alignment), or `None`.

If `align` is `None`, the alignment is determined based on other arguments, in a way that should be
what you want most of the time. It depends on any positioning arguments (`topleft`, `centerx`,
etc.), `anchor`, and `ptext.DEFAULT_TEXT_ALIGN`, which is set to `"left"` by default. I suggest you
generally trust the default alignment, and only specify `align` if something doesn't look right.

## Outline

	ptext.draw("hello world", (100, 100), owidth=1, ocolor="blue")

Keyword arguments:

* `owidth`: outline thickness, in outline units. Defaults to `None`.
* `ocolor`: outline color. Defaults to `ptext.DEFAULT_OUTLINE_COLOR`, which is set to `"black"` by
default.

The text will be outlined if `owidth` is specified. The outlining is a crude manual method, and will
probably look bad at large sizes. The units of `owidth` are chosen so that `1.0` is a good typical
value for outlines. Specifically, they're the font size times `ptext.OUTLINE_UNIT`, which is set to
`1/24` by default.

Valid values for `ocolor` are the same as for `color`.

## Drop shadow

	ptext.draw("hello world", (100, 100), shadow=(1.0,1.0), scolor="blue")

Keyword arguments:

* `shadow`: (x,y) values representing the drop shadow offset, in shadow units. Defaults to `None`.
* `scolor`: drop shadow color. Defaults to `ptext.DEFAULT_SHADOW_COLOR`, which is `"black"` by
default.

The text will have a drop shadow if `shadow` is specified. It must be set to a 2-element sequence
representing the x and y offsets of the drop shadow, which can be positive, negative, or 0. For
example, `shadow=(1.0,1.0)` corresponds to a shadow down and to the right of the text.
`shadow=(0,-1.2)` corresponds to a shadow higher than the text.

The units of `shadow` are chosen so that `1.0` is a good typical value for the offset. Specifically,
they're the font size times `ptext.SHADOW_UNIT`, which is set to `1/18` by default.

Valid values for `scolor` are the same as for `color`.

## Gradient color

	ptext.draw("hello world", (100, 100), color="black", gcolor="green")

Keyword argument:

* `gcolor`: Lower gradient stop color. Defaults to `None`.

Specify `gcolor` to color the text with a vertical color gradient. The text's color will be `color`
at the top and `gcolor` at the bottom. Positioning of the gradient stops and orientation of the
gradient are hard coded and cannot be specified.

Requries `pygame.surfarray` module, which uses numpy or Numeric library.

## Alpha transparency

	ptext.draw("hello world", (100, 100), alpha=0.5)

Keyword argument:

* `alpha`: alpha transparency value, between 0 and 1. Defaults to `1.0`.

In order to maximize reuse of cached transparent surfaces, the value of `alpha` is rounded.
`ptext.ALPHA_RESOLUTION`, which is set to `16` by default, specifies the number of different values
`alpha` may take internally. Set it higher (up to `255`) for more fine-grained control over
transparency values.

Requries `pygame.surfarray` module, which uses numpy or Numeric library.

## Anchored positioning

	ptext.draw("hello world", (100, 100), anchor=(0.3,0.7))

Keyword argument:

* `anchor`: a length-2 sequence of horizontal and vertical anchor fractions. Defaults to
`ptext.DEFAULT_ANCHOR`, which is set to `(0.0, 0.0)` by default.

`anchor` specifies how the text is anchored to the given position, when no positioning keyword
arguments are passed. The two values in `anchor` can take arbitrary values between `0.0` and `1.0`.
An `anchor` value of `(0,0)`, the default, means that the given position is the top left of the
text. A value of `(1,1)` means the given position is the bottom right of the text.

## Rotation

	ptext.draw("hello world", (100, 100), angle=10)

Keyword argument:

* `angle`: counterclockwise rotation angle in degrees. Defaults to `0`.

Positioning of rotated surfaces is tricky. When drawing rotated text with `ptext`, the anchor point,
the position you actually specify, remains fixed, and the text rotates around it. For instance, if
you specify the top left of the text to be at `(100, 100)` with an angle of `90`, then the Surface
will actually be drawn so that its bottom left is at `(100, 100)`.

If you find that confusing, try specifying the center. If you anchor the text at the center, then
the center will remain fixed, no matter how you rotate it.

In order to maximize reuse of cached rotated surfaces, the value of `angle` is rounded to the
nearest multiple of `ptext.ANGLE_RESOLUTION_DEGREES`, which is set to `3` by default. Set it lower
for more fine-grained control over rotation. It's recommended you set it only to values that divide
evenly into 90 in floating-point representation. Such values include:

	0.25 0.5 0.75 1 1.25 1.5 2 2.25 2.5 3 3.75 4.5 5 6 7.5 9 10 15 18 30

## Destination surface

	mysurface = pygame.Surface((400, 400)).convert_alpha()
	ptext.draw("hello world", (100, 100), surf=mysurface)

Keyword arugment:

* `surf`: destination `pygame.Surface` object. Defaults to the display Surface.

Specify `surf` if you don't want to draw directly to the display Surface
(`pygame.display.get_surface()`).

## Text Surface caching

	ptext.draw("hello world", (100, 100), cache=False)
	ptext.AUTO_CLEAN = False
	ptext.clean()

Keyword argument:

* `cache`: whether to cache Surfaces generated while rendering text during this call. Defaults to
`True`.

`ptext` caches `pygame.Surface` objects, so they don't have to rendered with subsequent calls. You
should be able to not worry about this part.

In order to keep memory from getting arbitrarily large, `ptext` will free previously cached
`Surface` objects, starting with the least recently used objects. In theory, this could cause
noticeable skips in gameplay. I haven't noticed it, but if you want to control this behavior, set
`ptext.AUTO_CLEAN` to `False`, and call `ptext.clean` yourself at times when framerate is not
cruical (e.g. menu screens).

`ptext.MEMORY_LIMIT_MB` is the approximate size of the cache in megabytes before a cleanup occurs.
It's set to `64` by default. As long as the cache stays below this size, `ptext.clean` is a no-op.
`ptext.MEMORY_REDUCTION_FACTOR` controls how much is deleted in this process. Valid values range
from `0.0` (everything is deleted) to `1.0` (just enough is deleted to drop below the limit). It's
set to `0.5` by default.

## `ptext.drawbox`: Constrained text

	ptext.drawbox("hello world", (100, 100, 200, 50))

`ptext.drawbox` requires two arguments: the text to be drawn, and a `pygame.Rect` or a `Rect`-like
object to stay within. The font size will be chosen to be as large as possible while staying within
the box. Other than `fontsize` and positional arguments, you can pass all the same keyword arguments
to `ptext.drawbox` as to `ptext.draw`.

## Other public methods

These methods are used internally, but you can use them if you want. They should work fine.

	ptext.getfont(fontname, fontsize)

`ptext.getfont` returns the corresponding `pygame.font.Font` object.

	ptext.wrap(text, fontname, fontsize, width=None, widthem=None)

`ptext.wrap` returns a list of substrings of `text`, one for each line in the word wrapped text.

	ptext.getsurf(text, **kwargs)

`ptext.getsurf` takes the same keyword arguments that `ptext.draw` takes (except for arguments
related to positioning), and returns the `pygame.Surface` containing the text to be drawn.
