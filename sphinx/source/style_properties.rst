================
Style Properties
================

The style properties associated with :doc:`styles <style>` control how
Displayables are shown. Style properties can be either given without a prefix,
in which case they apply to all states of the displayable, or with a prefix
that limits the property to a limited number of states.


Style Property Prefixes
=======================

Applying a prefix to a style property indicates allows a displayable to change
it's look in response to its focus or selection status. For example, a button
can change its color when the mouse hovers above it, or to indicate when the
choice represented by the button is the currently selected one.

There are five states a displayable can be it.

:dfn:`insensitive`
    Used when the user cannot interact with the displayable.

:dfn:`idle`
    Used when the displayable is neither focused nor selected.

:dfn:`hover`
    Used when the displayable is focused, but not selected.

:dfn:`selected_idle`
    Used when the displayable is not focused, and is selected.

:dfn:`selected_hover`
    Used when the displayable is focused and selected.

Button and Bar displayables (and their variants) update their state, and the
state of their children, in response to events. For example, when the user
puts his mouse over an unselected button, it and all its children will be put
into the hover state.

Style property prefixes allow one to set style properties for the different
states. There is a system of implications set up, so that a prefix can imply
setting the property for more than one state.

The implications are:

.. list-table::
 :header-rows: 1

 * - prefix
   - states implied by prefix

 * - (no prefix)
   - insensitive, idle, hover, selected\_idle, selected\_hover

 * - ``idle_``
   - idle, selected\_idle

 * - ``hover_``
   - hover, selected\_hover

 * - ``selected_``
   - selected\_idle, selected\_hover

 * - ``insensitive_``
   - insensitive

 * - ``selected_idle_``
   - selected\_idle

 * - ``selected_hover_``
   - selected\_hover

 * - ``selected_insensitive_``
   - selected\_insensitive

Using a text button, we can show this in action. Text buttons use two styles
by default: ``button`` for the button itself, and ``button_text`` for the
text inside the button. The :propref:`background` style property sets the
background of a button, while the :propref:`color` property sets the color of
text.::

    # The button background is gray when insensitive, light
    # blue when hovered, and dark blue otherwise.
    style button:
        background "#006"
        insensitive_background "#444"
        hover_background "#00a"

    # The button text is yellow when selected, and white
    # otherwise.
    style button_text:
        color "#fff"
        selected_color "#ff0"


Style Property Values
=====================

Each style property expects a specific kind of data. Many of these are
standard Python types, but a few are novel. Here are descriptions of the
novel kinds of value a style property can expect.

.. glossary::

    position
        Positions are used to specify locations relative to the upper-left
        corner of the containing area. (For placement and size, the containing
        area is given by the layout the displayable is in, if one is given, or
        the screen otherwise. For anchors, the containing area is the size of
        the displayable itself.)

        The way a position value is interpreted depends on the type of the
        value:

        int (like 0, 1, 37, or 42)
            An integer is interpreted as the number of pixels from the left
            or top side of the containing area.
        float (like 0.0, 0.5, or 1.0)
            A floating-point number is interpreted as a fraction of the
            containing area. For example, 0.5 is a point halfway between the
            sides of the containing area, while 1.0 is on the right or bottom
            side.

        .. function:: absolute(value, /)

            For example, ``absolute(100.25)``, or ``absolute(a+b)`` when both
            ``a`` and ``b`` may be floats or ints.

            An ``absolute`` number is interpreted as the number of pixels from
            the left or top side of the screen, when using subpixel-precise
            rendering.

        .. function:: position(absolute, relative, /)

            For example, ``position(-10, .5)``.

            A combination of `absolute`, which will be treated as an absolute
            position, and `relative`, which will be treated as a relative
            position, both being as described above. The two components are
            added together to form the final position.

            Both parameters should always be passed, otherwise unspecified
            results may occur.

    displayable
        Any displayable. If a displayable contains a "[prefix\_]" substitution,
        a prefix search is performed as described below.

    color
        Colors in Ren'Py can be expressed as strings beginning with the hash
        mark (#), followed by a hex triple or hex quadruple, with each of the
        three or four elements consisting of a one or two hexadecimal character
        color code.

        In a triple, the components represent red, green, and blue. In a
        quadruple, the components represent red, green, blue, and alpha. For
        example:

        * ``"#f00"`` and ``"#ff0000"`` represent an opaque red color.
        * ``"#0f08"`` and ``"#00ff0080"`` represent a semi-transparent green
          color.

        The color triples are the same as used in HTML.

        Colors can also be represented as a 4-component tuple, with the 4
        components being integers between 0 and 255. The components correspond to
        red, green, blue, and alpha, in that order.

        * ``(0, 0, 255, 255)`` represents an opaque blue color.

        Finally, colors can be an instance of :class:`Color`.


.. _style-prefix-search:

Style Prefix Search
-------------------

When a style property contains the "[prefix\_]" substitution, a prefix
search is performed. The prefix search is performed separately for
each state, including states that are implied by the original property
assigned.

For example, if we have::

    style button:
        hover_background "[prefix_]background.png"

separate searches are performed for the hover and selected\_hover states. The
prefixes searched vary based on the state.

.. list-table::
 :header-rows: 1

 * - state
   - search order

 * - idle
   - "idle\_", ""

 * - hover
   - "hover\_", "",

 * - insensitive
   - "insensitive\_", "", "idle\_"

 * - selected\_idle
   - "selected\_idle\_", "idle\_", "selected\_", ""

 * - selected_hover
   - "selected\_hover\_", "hover\_", "selected\_", ""

 * - selected\_insensitive
   - "selected\_insensitive\_", "hover\_", "selected\_", "", "selected\_idle\_", "idle\_"

When a search is performed, each prefix is tried in the order given. The string
has "[prefix\_]" replaced with the prefix, and then Ren'Py checks to see if
a loadable file or image with that name exists. If the file or image exists,
the search stops and the displayable found is used. Otherwise, it proceeds to
the next prefix.

The style prefix is passed through displayables that do not take user input,
including containers, transforms, and frames.

As an example of how this can be used, if the files :file:`idle_button.png` and
:file:`hover_button.png` exist (and no other files ending in :file:`button.png` do)::

    style button:
        background "[prefix_]button.png"

is equivalent to::

    style button:
        idle_background "idle_button.png"
        hover_background "hover_button.png"
        insensitive_background "idle_button.png"

        selected_idle_background "idle_button.png"
        selected_hover_background "hover_button.png"
        selected_insensitive_background "idle_button.png"


List of All Style Properties
============================

The style properties control the look of the various displayables. Not all
style properties apply to all displayables, so we've divided them up into
groups.


.. _position-style-properties:

Position Style Properties
-------------------------

These properties apply to all displayables, and mostly control the
position of a displayable inside the area allocated to it by a layout,
or on the screen when not inside a layout.

.. style-property:: alt string or None

    Alternative text used for the displayable when self-voicing is
    enabled. See the :doc:`self voicing <self_voicing>` section for
    more information.

.. style-property:: xpos position

    The position of the displayable relative to the left side of the
    containing area.

.. style-property:: ypos position

    The position of the displayable relative to the top side of the
    containing area.

.. style-property:: pos tuple of (position, position)

    Equivalent to setting xpos to the first component of the tuple,
    and ypos to the second component of the tuple.

.. style-property:: xanchor position

    The position of the anchor relative to the left side of the
    displayable.

.. style-property:: yanchor position

    The position of the anchor relative to the top side of the
    displayable.

    If the displayable is a :func:`Text`, then there is one special value
    of this property. If the property is set to renpy.BASELINE, the anchor
    is set to the baseline of the first line of text.

.. style-property:: anchor tuple of (position, position)

    Equivalent to setting xanchor to the first component of the tuple,
    and yanchor to the second component of the tuple.

.. style-property:: xalign float

    Equivalent to setting xpos and xanchor to the same value. This has
    the effect of placing the displayable at a relative location on
    the screen, with 0.0 being the left side, 0.5 the center, and 1.0
    being the right side.

.. style-property:: yalign float

    Equivalent to setting ypos and yanchor to the same value. This has
    the effect of placing the displayable at a relative location on
    the screen, with 0.0 being the top, 0.5 the center, and 1.0
    the bottom.

.. style-property:: align tuple of (float, float)

    Equivalent to setting xalign to the first component of the tuple,
    and yalign to the second.

.. style-property:: xcenter position

    Equivalent to setting xpos to the value of this property, and
    xanchor to 0.5.

.. style-property:: ycenter position

    Equivalent to setting ypos to the value of this property, and
    yanchor to 0.5.

.. style-property:: xycenter tuple of (position, position)

    Equivalent to setting xcenter to the first component of the tuple,
    and ycenter to the second.

.. style-property:: xoffset int

    Gives a number of pixels that are added to the horizontal position
    computed using xpos and xalign.

.. style-property:: yoffset int

    Gives a number of pixels that are added to the vertical position
    computed using ypos and yalign.

.. style-property:: offset tuple of (int, int)

    Equivalent to setting xoffset to the first component of the tuple,
    and yoffset to the second component of the tuple.

.. style-property:: xmaximum int

    Specifies the maximum horizontal size of the displayable, in pixels.

.. style-property:: ymaximum int

    Specifies the maximum vertical size of the displayable in pixels.

.. style-property:: maximum tuple of (int, int)

    Equivalent to setting xmaximum to the first component of the
    tuple, and ymaximum to the second.

.. style-property:: xminimum int

    Sets the minimum width of the displayable, in pixels. Only works
    with displayables that can vary their size.

.. style-property:: yminimum int

    Sets the minimum height of the displayables, in pixels. Only works
    with displayables that can vary their size.

.. style-property:: minimum tuple of (int, int)

    Equivalent to setting xminimum to the first component of the
    tuple, and yminimum to the second.

.. style-property:: xsize int or float

    Equivalent to setting xminimum and xmaximum to the same value. This
    has the effect of setting the width of the displayable.  f the size
    is a float, it's relative to the width of the containing area.

.. style-property:: ysize int or float

    Equivalent to setting yminimum and ymaximum to the same value. This
    has the effect of setting the height of the displayable. If the size
    is a float, it's relative to the height of the containing area.

.. style-property:: xysize tuple of (int or float, int or float)

    Equivalent to setting xsize to the first component of
    the tuple, and ysize to the second component.

.. style-property:: xfill boolean

    If true, the displayable will expand to fill all available
    horizontal space. If not true, it will only be large enough to
    contain its children.

    This only works for displayables that can change size.

.. style-property:: yfill boolean

    If true, the displayable will expand to fill all available
    vertical space. If not true, it will only be large enough to
    contain its children.

    This only works for displayables that can change size.

.. style-property:: area tuple of (int, int, int, int)

    The tuple is interpreted as (`xpos`, `ypos`, `width`,
    `height`). Attempts to position the displayable such that it's
    upper-left corner is at `xpos` and `ypos`, and its size is `width`
    and `height`.

    It does this by setting the xpos, ypos, xanchor, yanchor,
    xmaximum, ymaximum, xminimum, yminimum, xfill, and yfill
    properties to appropriate values.

    This will not work with all displayables and all layouts.

.. style-property:: mipmap boolean or None

    This controls if textures created by this displayable are mipmapped.
    This applies only to certain displayables, including :func:`Text`,
    :func:`Movie`, and dissolves.

    If None, the default for this is taken from config variables such
    as :var:`config.mipmap_text`, :var:`config.mipmap_movies`, and
    :var:`config.mipmap_dissolves`.

.. _text-style-properties:

Text Style Properties
---------------------

.. style-property:: antialias boolean

    If True, the default, TrueType font text will be rendered
    anti-aliased.

.. style-property:: adjust_spacing boolean or str

    If True, Ren'Py will adjust the spacing of drawable-resolution text
    to match the spacing of the text rendered at virtual resolution, to
    ensure the size of frames and other containers holding text does not
    change.

    When set to False, text will not change in size, but it is the creator's
    responsibility to ensure there is enough space to layout text at any
    window size.

    When set to the string "horizontal", text will adjust in the horizontal
    direction only. When set to the string "vertical", text will adjust in
    the vertical direction only.

    Defaults to True for most text, but False for text in an ``input``.

.. style-property:: altruby_style style or None

    If not None, this should be a style object. The style that's used for
    alternate ruby text.

.. style-property:: axis dict or None

    This allows the axes of a :ref:`variable font <variable-fonts>` to be
    set. If not None, this should be a dictionary mapping axis names to
    values. For example::

        style default:
            font "VariableFont.ttf"
            axis { "weight" : 500, "width" : 95 }

.. style-property:: black_color color

    When rendering an image-based font, black will be mapped to this
    color. This has no effect for TrueType fonts.

    This may be None in the case of ruby/furigana text, to use the same
    color as the parent text.

.. style-property:: bold boolean

    If True, render the font in a bold style. For a TrueType font,
    this usually involves synthetically increasing the font weight. It
    can also cause the font to be remapped, using
    :var:`config.font_replacement_map`.

.. style-property:: caret displayable or None

    If not None, this should be a displayable. The input widget will
    use this as the caret at the end of the text. If None, a 1 pixel
    wide blinking line is used as the caret.

.. style-property:: color color

    The color the text is rendered in. When using a TrueType font,
    the font is rendered in this color. When using an image-based
    font, white is mapped to this color.

    This may be None in the case of ruby/furigana text, to use the same
    color as the parent text.

.. style-property:: emoji_font string

    The font that's used to render Emoji characters. This is automatically
    used when a sequence of one or more Emoji are encountered, and no
    font text tag is in use.

.. style-property:: first_indent int

    The amount that the first line of text in a paragraph is indented
    by, in pixels.

.. style-property:: font string

    A string giving the name of the font used to render text.

    For a TrueType font file, this is usually the name of the file
    containing the font (like ``"DejaVuSans.ttf"``). To select a second
    font in a collection, this can be prefixed with a number and
    at sign (like ``"0@font.ttc"`` or ``"1@font.ttc"``). For an
    image-based font, this should be the name used to register the
    font.

.. style-property:: font_features dict or None

    If not None, this should be a dictionary mapping OpenType layout features to values.
    The keys of the dictionary should be the four-character OpenType feature tags, and
    the values should be booleans.

    A list of layout features can be found `here <https://learn.microsoft.com/en-us/typography/opentype/spec/featuretags>`_.

.. style-property:: hinting str

    Controls how the font will be hinted. This should be one of the following
    strings:

    "auto"
        The default, forces use of the Freetype auto hinter.
    "auto-light"
        Forces the use of the freetype auto hinter in light mode, which only
        hints vertically.
    "bytecode"
        Uses bytecode hinting information found in the font.
    "none"
        Does not hint the font.

    This may also be True, in which case the value is looked up in
    :var:`config.font_hinting`.

.. style-property:: hyperlink_functions tuple of (function, function, function)

    This is a tuple of three functions relating to hyperlinks in text.

    The first item is the hyperlink style function. When called with a single
    argument, the argument of the hyperlink, it must return a style object to
    use for the hyperlink, such as ``style.hyperlink_text``. Note that a
    style object is not a string.

    The second item is the hyperlink clicked function. This function is called
    when a hyperlink is chosen by the user. If it returns a value other than
    None, the interaction returns that value.

    The third item is the hyperlink focus function. This function is called
    with the argument of the hyperlink when the hyperlink gains focus, and
    with None when it loses focus. If it returns a value other than None,
    the interaction returns that value.

.. style-property:: instance string or None

    When using a :ref:`variable font <variable-fonts>`, this can be
    a string giving a named instance of the font to use. For example,
    if the font has a "Bold" instance, this can be set to "Bold" to
    use that instance.

.. style-property:: italic boolean

    If true, the text will be rendered in italics. For a TrueType font,
    this usually involves synthetically increasing the font slant. It
    can also cause the font to be remapped, using
    :var:`config.font_replacement_map`.

.. style-property:: justify boolean

    If True, additional whitespace is inserted between words so that
    the left and right margins of each line are even. This is not
    performed on the last line of a paragraph.

.. style-property:: kerning float

    A kerning adjustment, the number of pixels of space that's added
    between each pair of characters. (This can be negative to remove
    space between characters.)

.. style-property:: language string

    Controls the language family used to break text into lines, and
    for certain other text transformations. Legal values are:

    ``"unicode"`` (default)
        Uses the Unicode linebreaking algorithm, which is suitable for
        most languages.

    ``"japanese-strict"``
        Formats Japanese text in a "strict" manner. It
        forbids breaks before small kana and prolonged sound marks.

    ``"japanese-normal"``
        Formats Japanese text in a "normal" manner. It
        allows breaks before small kana, prolonged sound marks, and
        certain hyphens.

    ``"japanese-loose"``
        Formats Japanese text in a "loose" manner. It allows breaks
        before small kana , prolonged sound marks, iteration marks,
        inseparable characters, centered punctuation marks, and postfixes;
        and allows breaks before prefixes.

    ``"korean-with-spaces"``
        Used for Korean text delimited by whitespace. This prevents linebreaking
        between adjacent Korean characters.

    ``"thaic90"``
        Used for Thai text displayed in fonts that support the
        `C90 encoding for Thai <http://www.bakoma-tex.com/doc/fonts/enc/c90/c90.pdf>`_.
        This combines groups of characters into single glyphs, allowing for better
        display of vowel and tone marks. Line breaking uses the unicode algorithm.

    ``"western"``
        Allows breaking only at whitespace. Suitable for most
        languages.

    ``"anywhere"``
        Allows breaking at anywhere without ruby.

    The three Japanese breaking modes are taken from the `CSS3 text module <http://www.w3.org/TR/css3-text/#line-break>`_.

.. style-property:: layout string

    Controls how words are allocated to lines. Legal values are:

    ``"tex"`` (default)
        Uses the Knuth-Plass linebreaking algorithm, which attempts to minimize
        the difference in line lengths of all but the last line.

    ``"subtitle"``
        Uses the Knuth-Plass linebreaking algorithm, but attempts to even out
        the lengths of all lines.

    ``"greedy"``
        A word is placed on the first line that has room for it.

    ``"nobreak"``
        Do not line-break.

.. style-property:: line_leading int

    The number of pixels of spacing to include above each line.

.. style-property:: line_overlap_split int

    When in slow text mode, and two lines overlap, this many pixels of
    the overlap are allocated to the top line. Increase this if the
    bottoms of characters on the top line are clipped.

.. style-property:: line_spacing int

    The number of pixels of spacing to include below each line.

.. style-property:: min_width int

    Sets the minimum width of each line of that. If a line is shorter
    than this, it is padded to this length, with ``textalign`` used to
    specify where such padding is placed.

.. style-property:: newline_indent boolean

    If true, the :propref:`first_indent` indentation is used after
    each newline in a string. Otherwise, the :propref:`rest_indent`
    indentation is used.

.. style-property:: outlines list of tuple of (int, color, int, int)

    This is a list of outlines that are drawn behind the text. Each
    tuple specifies an outline, and outlines are drawn from back to
    front.

    The list contains (`size`, `color`, `xoffset`, `yoffset`)
    tuples. `size` is the amount the font is expanded by, in
    pixels. `color` is the color of the outline. `xoffset` and
    `yoffset` are the amount the outline is shifted by, in pixels.

    The outline functionality can also be used to give drop-shadows to
    fonts, by specifying a size of 0 and non-zero offsets.

    By default, `size`, `xoffset` and `yoffset` are scaled with the text.
    When given as the absolute type, they are not scaled. For example::

        style default:
            outlines [ (absolute(1), "#000", absolute(0), absolute(0)) ]

    will always produce a 1 pixel-wide border.

    Outlines only work with TrueType fonts. Outlines only work when applied
    to an entire Text displayable. They do not work when applied to a hyperlink,
    text tag, or other method that applies to only a portion of the text.

.. style-property:: outline_scaling string

    This determines how outline sizes and offsets are scaled when the
    window is scaled.

    ``"linear"``
        The default, best for text with thick outlines. The window scaling factor
        is applied to the outline size, and then rounded to an integer.
        This looks better for thick outlines, but concentric outlines of similar
        thickness may become indistinguishable.

        The resulting width is always at least 1 pixel.

    ``"step"``
        Best for text that uses thin outlines and text
        that uses multiple outlines. The window scaling factor is
        rounded down to an integer and applied to the outline size
        and offsets. This ensures that multiple outlines all scale together,
        without risk of eclipsing one another, but this yields different visual
        results depending on the size of the game window.

        The window scaling factor is always above 1.

.. style-property:: prefer_emoji boolean

    Some unicode characters have both Emoji and non-Emoji presentations. This
    style property chooses if such characters are given the Emoji presentation
    or not.

.. style-property:: reading_order string

    This controls the how text is presented for reading when RTL support has
    been enabled. The main use of this is to influence directionally neutral
    lines, such as those made up purely of punctuation.

    ``None``
        The default, the reading order will be determined based on the text.
        Most neutral lines will be treated as left-to-right.

    ``"ltr"``
        Forces left-to-right presentation, even when a line is predominantly
        made up of runs of RTL characters.

    ``"rtl"``
        Forces right-to-left presentation, even when a line is predominantly
        made up of runs of LTR characters.

    ``"wltr"``
        Prefers left-to-right presentation, but will switch to right-to-left
        presentation if it makes more sense for the current text. Useful for
        impacting punctuation-only lines such as ``"..!"``.

    ``"wrtl"``
        Prefers right-to-left presentation, but will switch to left-to-right
        presentation if it makes more sense for the current text. Useful for
        impacting punctuation-only lines such as ``"..!"``.

.. style-property:: rest_indent int

    Specifies the number of pixels the second and later lines in a
    paragraph are indented by.

.. style-property:: ruby_line_leading int

    The number of pixels of spacing to include above each line that
    contains :ref:`ruby text <ruby-text>`. This is in addition to
    :propref:`line_leading`.

.. style-property:: ruby_style style or None

    If not None, this should be a style object. The style that's used for
    ruby text.

.. style-property:: shaper "harfbuzz" or "freetype".

    The shaper used on text. This should be one of "harfbuzz" or "freetype".
    The harfbuzz shaper is more capable but only works on Ren'Py 8, while
    "freetype" works in Ren'Py 7 as well.

    The shaper takes a series of characters and turns it into a series of
    positioned glyphs. This is used for things like ligatures, Indic/Bhramic
    languages, and combining emjoi.

.. style-property:: size int

    The size of the font on the screen. While this is nominally in
    pixels, font files may have creative interpretations of this
    value.

.. style-property:: slow_abortable boolean

    If True, a click that reaches the Text object will cause the slow
    text to abort, which means that the rest of the slow text will be
    displayed instantly.

.. style-property:: slow_cps int or True

    If a number, shows text at the rate of that many characters per
    second. If True, shows text at the speed taken from the "Text
    Speed" preference.

.. style-property:: slow_cps_multiplier float

    The speed of the text is multiplied by this number. This can be
    used to have a character that speaks at a faster-than-normal rate
    of speed.

.. style-property:: strikethrough boolean

    If True, a line is drawn through the text.

.. style-property:: textalign float

    This is used when a line is shorter than the width of the text
    displayable. It determines how much of the extra space is placed
    on the left side of the text. (And hence, the text alignment.)

    0.0 will yield left-aligned text, 0.5 centered text, and 1.0
    right-aligned text.

.. style-property:: underline boolean

    If True, an underline will be added to the text.

.. style-property:: vertical boolean

    If True, the text will be rendered vertically.

    There are multiple caveats:

    * If :propref:`shaper` is "freetype", vertical layout will likely be incorrect.

    * Harfbuzz will convert horizontal forms to vertical forms if those
      forms are present in the font.

    * Characters will be laid out top to bottom, right to left, but will
      not be rotated. This means that horizontal characters will be laid
      out one on top of another.

    * If vertical text information is not present in the font, it will be
      synthesized, but may be incorrect. Generally, ideographic text works
      better than non-ideographic text.

.. _window-style-properties:

Window Style Properties
-----------------------

Window properties are used to specify the look of windows, frames, and
buttons. :ref:`margin-style-properties` also form part of this group.

.. style-property:: background displayable or None

    A displayable that is used as the background of the window. This
    is often a :func:`Frame`, allowing the size of the background to
    scale with the size of the window.

    If None, no background is drawn, but other properties function as
    if the background was present.

.. style-property:: foreground displayable or None

    If not None, this displayable is drawn above the contents of the
    window.

.. style-property:: left_padding int

    The amount of space between the background and the left side of
    the window content, in pixels.

.. style-property:: right_padding int

    The amount of space between the background and the right side of
    the window content, in pixels.

.. style-property:: xpadding int

    Equivalent to setting left_padding and right_padding to the same
    value.

.. style-property:: top_padding int

    The amount of space between the background and the top side of
    the window content, in pixels.

.. style-property:: bottom_padding int

    The amount of space between the background and the bottom side of
    the window content, in pixels.

.. style-property:: ypadding int

    Equivalent to setting top_padding and bottom_padding to the same
    value.

.. style-property:: padding tuple

    When given a two-item tuple, equivalent to setting xpadding and
    ypadding to the two items. When given a four-item tuple, equivalent
    to setting left_padding, top_padding, right_padding, and bottom_padding
    to the four items.

.. style-property:: size_group string or None

    If not None, this should be a string. Ren'Py will render all
    windows with the same size_group value at the same size.

.. style-property:: modal boolean or callable

    If True, the window is treated as modal. Events will stop propagating
    through layers while the mouse is within the window or button. If False,
    the window is not modal.

    This may also be a callable function. If it is, the function is called
    with (ev, x, y, w, h), where `ev` is either a pygame event, or None to
    represent a generic mouse event; `x` and `y` are coordinates relative
    to the window, and `w` and `h` are the height and width of the window.
    If the callable returns true, the windows is treated as modal. If it
    returns false, it is not.

.. _button-style-properties:

Button Style Properties
-----------------------

.. style-property:: child displayable or None

    If not None, this gives a displayable that replaces the child of the
    button. For example, this (as insensitive_child) can be used to replace the
    contents of an insensitive button with an image that indicates the button
    is locked.

.. style-property:: hover_sound string

    A sound that is played when the button gains focus.

.. style-property:: activate_sound string

    A sound that is played when the button is activated (clicked). This is also
    played when a bar or drag is activated.

.. style-property:: mouse string

    The mouse style that is used when the button is focused. This
    should be one of the styles in :var:`config.mouse`.

.. style-property:: focus_mask multiple

    A mask that's used to control what portions of the button can be
    focused, and hence clicked on. The type of this property determines
    how it is interpreted.

    Displayable
        The areas of the displayable that are not transparent cause the button
        to be focused.
    True
        The button itself is used as the displayable (so
        non-transparent areas of the button cause the button to be
        focused).
    callable
        If a non-displayable callable (like a function, method, or object
        with a ``__call__`` method) is given, the function is called with two
        arguments, the x and y offset from the top-left corner of the
        displayable.

        If the function returns a callable when called with two arguments,
        that callable is called with four arguments - the x and y offsets, and the width
        and height of the displayable.

        If the function returns true, the displayable is focused.
    None
        If none is given, the entire button can be focused.

.. style-property:: keyboard_focus boolean

   If True, the default, this button can be focused using the keyboard focus
   mechanism, if it can be focused at all. If False, the keyboard focus
   mechanism will skip this button. (The keyboard focus mechanism is used
   by keyboards and keyboard-like devices, such as gamepads.)

.. style-property:: keyboard_focus_insets (int, int, int, int) or None

    If not None, this should be a tuple of four integers, giving the
    number of pixels that are used to shrink the left, top, right, and
    bottom sides of the focus rectangle, when it's used for keyboard
    focus.

    This can be useful when buttons overlap. The keyboard focus algorithm
    doesn't work with overlapping buttons, and so this can be used to to
    shrink the size of the buttons internally, allowing focus to work.

.. style-property:: key_events boolean

    If True, keyboard-generated events are passed to the children of this
    button. If False, those events are not propagated. In this default style,
    this is set to True while the button is hovered, and False otherwise.

    Setting this to True can be used to propagate keyboard events to an input
    inside a button, even when the button isn't focused.


.. _bar-style-properties:

Bar Style Properties
--------------------

Bars are drawn with gutters on the left and right, that when clicked can cause
the bar to move by a small amount. The remaining space is the portion of the
bar that can change, with the amount on each side proportional to the bar's
value as a fraction of the range.

The thumb is an area in the center of the bar that can be dragged by the user.

When a bar is drawn, the thumb's shadow is drawn first. Then the left/bottom
and right/top sides of the bar, followed by the thumb itself.

Note that the valid sides of a bar depend on the value of the bar_vertical
property. If it's True, the top and bottom sides are relevant. Otherwise, the
left and right sides are used.

.. style-property:: bar_vertical boolean

    If True, the bar has a vertical orientation. If False, it has a
    horizontal orientation.

.. style-property:: bar_invert boolean

    If True, the value of the bar is represented on the right/top
    side of the bar, rather than the left/bottom side.

.. style-property:: bar_resizing boolean

    If True, we resize the sides of the bar. If False, we render the
    sides of the bar at full size, and then crop them.

.. style-property:: left_gutter int

    The size of the gutter on the left side of the bar, in pixels.

.. style-property:: right_gutter int

    The size of the gutter on the right side of the bar, in pixels.

.. style-property:: top_gutter int

    The size of the gutter on the top side of the bar, in pixels.

.. style-property:: bottom_gutter int

    The size of the gutter on the bottom side of the bar, in pixels.

.. style-property:: left_bar displayable

    The displayable used for the left side of the bar.

.. style-property:: right_bar displayable

    The displayable used for the right side of the bar.

.. style-property:: top_bar displayable

    The displayable used for the top side of the bar.

.. style-property:: bottom_bar displayable

    The displayable uses for the bottom side of the bar.

.. style-property:: base_bar displayable

    A single displayable that is used for left_bar/right_bar or
    top_bar/bottom_bar, as appropriate. (This can be used
    with thumb to make a slider or scrollbar.)

.. style-property:: thumb displayable or None

    If not None, this is a displayable that is drawn over the break
    between the sides of the bar.

.. style-property:: thumb_shadow displayable or None

    If not None, this is a displayable that is drawn over the break
    between the sides of the bar.

.. style-property:: thumb_offset int or tuple of (int, int)

    The amount that by which the thumb overlaps the bars, in
    pixels. To have the left and right bars continue unbroken, set
    this to half the width of the thumb in pixels. This may also be
    a tuple, in which case the first number is used for the left/top
    thumb offset, and the second number is used for the right/bottom
    thumb offset.

.. style-property:: mouse string

    The mouse style that is used when the button is focused. This
    should be one of the styles in :var:`config.mouse`.

.. style-property:: unscrollable string or None

    Controls what happens if the bar is unscrollable (if the range is
    set to 0, as is the case with a viewport containing a displayable
    smaller than itself). There are three possible values:

    ``None``
        Renders the bar normally.

    ``"insensitive"``
        Renders the bar in the insensitive state. This allows the
        bar to change its style to reflect its lack of usefulness.

    ``"hide"``
       Prevents the bar from rendering at all. Space will be allocated
       for the bar, but nothing will be drawn in that space.

.. style-property:: keyboard_focus boolean

   If True, the default, this button can be focused using the keyboard focus
   mechanism, if it can be focused at all. If False, the keyboard focus
   mechanism will skip this button. (The keyboard focus mechanism is used
   by keyboards and keyboard-like devices, such as joypads.)


.. _box-style-properties:

Box Style Properties
--------------------

These are used for the horizontal and vertical box layouts.

.. style-property:: spacing int

    The spacing between members of this box, in pixels.

.. style-property:: first_spacing int

    If not None, the spacing between the first and second members of
    this box, in pixels. This overrides the spacing property.

.. style-property:: box_reverse boolean

    If True, the placement of the items in the box will be reversed. When
    this is True, a hbox will be filled right-to-left, and a vbox will
    be filled bottom-to-top. This defaults to False.

.. style-property:: box_wrap boolean

    If True, then boxes will wrap when they reach the end of a line or column.
    If False (the default), they will extend past the end of the line.

.. style-property:: box_wrap_spacing int

    When box_wrap is True, this is the spacing between wrapped lines
    or columns. (So it is the vertical spacing between lines in a wrapped
    hbox, and the horizontal spacing between columns in a wrapped vbox.)

.. style-property:: box_align float or None

    This determines the alignment of rows or columns in a box. If 0.0,
    the default, rows are left-aligned and columns are top-aligned. If 0.5,
    it will center wrapped rows.

    When this is not None, addional space will not be offered to the child when
    :propref:`xfill` or :propref:`yfill` is set, ignoring the child's position
    in the box's primary direction.

.. style-property:: order_reverse boolean

    If False, the default, the items in the box will be drawn first-to-last,
    with the first item in the box being below the second, and so on. If True,
    this order will be reversed, and the first item in the box will be above
    all other items in the box.

.. style-property:: box_justify boolean or "first" or "all"

    If not False, the contents of the box will be justified - that is, the items
    inside the box will have the spacing between them adjusted so they evenly
    span the size of the box. True will cause all lines but the final line to be
    justified. "first" will cause only the first line to be justified. "all"
    will cause all lines to be justified, including the final line. Justified
    lines do not obey box_align, except when there is only one item in a line.

    When this is not false, addional space will not be offered to the child when
    :propref:`xfill` or :propref:`yfill` is set, ignoring the child's position
    in the box's primary direction.


.. _grid-style-properties:

Grid Style Properties
---------------------

These are the properties used by the grid and vpgrid displayables.
:ref:`margin-style-properties` also form part of this group.

.. style-property:: spacing int

    The spacing between cells of the grid, in pixels.

.. style-property:: xspacing int or None

    The spacing between cells of the grid in the horizontal direction, in pixels.
    This takes precedence over the spacing property when it is not None.

.. style-property:: yspacing int or None

    The spacing between cells of the grid in the vertical direction, in pixels.
    This takes precedence over the spacing property when it is not None.


.. _fixed-style-properties:

Fixed Style Properties
----------------------

These are used with the fixed layout.

.. style-property:: fit_first boolean or "width" or "height"

    If True, then the size of the fixed layout is shrunk to be equal with
    the size of the first item in the layout. If "width", only the width is changed
    (the fixed will fill the screen vertically). Similarly, "height" only changes
    the height.

    The position of the child is ignored for the purpose of fit\_first.

.. style-property:: xfit boolean

    If True, the size of the fixed layout is shrunk horizontally to match the
    right side of the rightmost child of the fixed.

.. style-property:: yfit boolean

    If True, the size of the fixed layout is shrunk vertically to match the
    bottom side of the bottommost child of the fixed.

When these properties are supplied, the children of the fixed are first
offered the full area given to the fixed by its parent. Once the size
and placement of the children are known, these properties are then used
to select the size of the fixed.


.. _margin-style-properties:

Margin Style Properties
-----------------------

Margin properties are used to add transparent space around some
displayables. Most notably: windows, frames, buttons and outside
a grid or vpgrid.

.. style-property:: left_margin int

    The amount of transparent space to the left of the displayable, in
    pixels.

.. style-property:: right_margin int

    The amount of transparent space to the right of the displayable, in
    pixels.

.. style-property:: xmargin int

    Equivalent to setting left_margin and right_margin to the same
    value.

.. style-property:: top_margin int

    The amount of transparent space above the displayable, in pixels.

.. style-property:: bottom_margin int

    The amount of transparent space below the displayable, in pixels.

.. style-property:: ymargin int

    Equivalent to setting top_margin and bottom_margin to the same
    value.

.. style-property:: margin tuple

    When given a two-item tuple, equivalent to setting xmargin and
    ymargin to the two items. When given a four-item tuple, equivalent
    to setting left_margin, top_margin, right_margin, and bottom_margin
    to the four items.
