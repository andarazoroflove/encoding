Subtitler - Sub Station Alpha v2.x/4.x subtitling plugin for VirtualDub
Copyright (C) 2000-2002 Avery Lee, All Rights Reserved.

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program; if not, write to the Free Software
Foundation, Inc., 675 Mass Ave, Cambridge, MA 02139, USA.



What does this filter do?
=========================
The "subtitler" filter is designed to draw text onto video.  As its name
implies, this functionality is usually used to subtitle video that is in
a foreign language.  Subtitler is designed to generate crisp, high-quality
text with a minimum of fuss.

Currently, the script format is compatible with the Sub Station Alpha (SSA)
program by Kotus, which at this time is available for free from:

    http://www.eswat.demon.co.uk/

Since this filter does not provide an interface for you to edit the scripts,
it is highly recommended, although not strictly necessary, to install SSA 4,
and to download a copy of the Sub Station Alpha 4.00 script format document.

This filter is not a part of Sub Station Alpha, nor is it endorsed by Kotus.


What's new in version 2.3
=========================
Version 2.3 fixes a memory corruption issue and incorporates minor speed
optimizations in the rasterizer.


Using the subtitler
===================
Copy subtitler.vdf into the PLUGINS\ directory in the VirtualDub
program directory.  The filter will then automatically load when
the main program starts.

To use it, go to Video > Filters and add the "subtitler" filter.  You'll
then be able to select a script filename and optionally toggle text
antialiasing.  Subtitler accepts Sub Station Alpha V2.x and V4.x
scripts.  Text is then applied to the outgoing AVI according to the script.

Scripts supply a series of _dialogue events_ to display; these are based off
of templates known as _styles_.  Each event has a start time, when the
subtitler finds a place for the text and begins displaying it on screen,
and an end time, when the text expires and disappears.


Viewing a demo of the subtitler's effects
=========================================
A script called DEMO.SSA has been included with this filter, along with an
accompanying video file GRAY.AVI.  Load the AVI into VirtualDub and add the
subtitler filter with default settings and the DEMO.SSA script, then hit
F5 to view.  For fun, you can also load the script into Sub Station Alpha
4.x, but some effects won't display properly.


Timing
======
The post-decimal part of Sub Station Alpha timings are in hundredths of a
second, not pictures.  Times in SMPTE format need to be modified before being
placed in an SSA script.

Although it is not recommended, you can place multiple dialogue events in
a script that have the same start time.  The filter will handle them in the
order that they appear in the script, resolving collisions in favor of events
occurring earlier in the file.

SSA timing correction (ramping) is not supported.  If you find SSA timing
corrections in an existing script, you will need to manually skew the timings
in the actual dialogue lines instead.  (This assumes that the timings in the
script coincide with those in your video source, which is actually pretty
unlikely given the timing accuracy of consumer video and audio capture
devices.)


Positioning
===========
The subtitler filter is designed to properly render scripts that were
originally intended for subbing with a genlock, i.e. titles composed for
full screen rendering.  As a result, it attempts to imitate SSA positioning
as closely as possible and then scales the result to the video window.
Although the SSA script format uses pixel values for positioning, subtitler
interprets these in terms of the original script's screen size.  The screen
size is denoted by a PlayResY: header at the top of SSA version 4 scripts:

  [Script Info]
  ; This is a Sub Station Alpha v4 script.
  ; For Sub Station Alpha info and downloads,
  ; go to http://www.eswat.demon.co.uk/
  ; or email kotus@eswat.demon.co.uk
  Title: <untitled>
  Original Script: <unknown>
  Collisions: Normal
  PlayResY: 480
  PlayDepth: 0

Subtitler assumes a 5:4 aspect ratio for PlayResY:1024 (1280x1024)
and a 4:3 aspect ratio for all other resolutions.  Font sizes are also scaled
down by the same factor as the margins.  If PlayResY: is absent, as it will
be in version 2 scripts, the subtitler will instead interpret margins and
font sizes directly, which is probably not appropriate.


Collision detection and resolution
==================================
If the default placement of a subtitle overlaps one that is already being
displayed, a collision is said to take place.  Collision resolution is the
process where the subtitler moves the new subtitle until it no longer overlaps
any existing ones.  Titles that have vertical flush bottom or center placement
move up to resolve; top titles move down.  If the subtitle is moved all the
way to the opposite border without resolving, it simply overlays whatever
exists at the terminal position.

Subtitles never move once they have resolved, so you will need to be careful
whenever you have back-to-back subtitles.  Subtitler removes subtitles before
adding new ones for a given frame, so subtitles that do not overlap in time
will never collide.  (The exact ending time is not part of the subtitle and
does not collide with another subtitle that starts at that same time.)  If
you manage to have a tiny overlap between the titles though, successive titles
will alternate and/or walk up and down the screen as they collide, which is
very disconcerting.

SSA's color collision resolution is not supported and all subtitles will
display in their primary color regardless of collisions.  Also, moving text
(Banner and Scroll Up effects) do not participate in collisions -- they will
never cause collisions or move as a result of them, and thus care should be
taken not to overlap scrollers.

If you need to remain compatible with SSA4 to output to a genlock, you also
need to be aware that SSA's collision detection and resolution support is
considerably more restricted: titles must have the same style, must have
bottom positioning, and must not have alignment or margin overrides in order
to collide and resolve.

Final note: Seeking backwards in the VirtualDub editor with the subtitler
may not produce the expected results when collisions occur, when you have
dialogue lines like this:

	5:27-5:29		Title A
	5:28-5:31		Title B
	5:29-5:33		Title C

When played from the beginning, the screen will look like this at 5:30 due
to a collision between A and B:

    Title B
    Title C

If you seek directly to 5:30, the subtitler may not process title A, and
will instead render this:

    Title C
    Title B

To avoid this problem, keep your collision chains to a manageable length,
and step forward from a couple of seconds back when you need to verify
placement in the presence of collisions.


Text escapes
============
The following character pairs are specially interpreted by the subtitler:

    \n    soft return (manual wordwrap)
    \N    hard return

These are nonstandard and supported only by the subtitler:

    \h    nonbreakable space (160)
    \!    VirtualDub version string
    \\    backslash
	\{    open brace

Note that the filter will discard single backslashes that would begin
illegal escapes, whereas SSA displays them.  This incompatibility is
intentional.  SSA's handling of backslashes outside of style overrides makes
some text combinations impossible, such as actually displaying "\n."  You
can do this with the filter with \\n.


Style overrides
===============
Text within braces {} is considered to be part of a style override; these
are used to modify the display of text by modifying parameters beyond what
the style would normally prescribe.  Subtitler supports the following style
overrides:

    \a      Alignment                   {\a4}top
    \b      Bold                        {\b1}bold{\b0}
    \c      Color                       {\c&Hffff00;}cyan{\r}
    \fs     Font size                   {\fs100}big text
    \fn     Font name                   {\fnCourier}typewriter
    \fe     Font encoding               {\fe2}l{\fe0} bulletted item
    \i      Italics                     {\i1}sit!!!!{\i0}
    \k      Karaoke                     {\k20}I {\k50}can't {\k50}sing
    \K      Smooth karaoke (extension)  {\K100}AAAAAAAAAAaaaaaaaaa!!!!!
    \q      Wrap control (extension)    {\q1}I like EigoAnim style.
    \r      Reset style overrides       {\fs20}oops {\r}...aahhh...

See the SSA4 script format documentation for better descriptions of these
overrides.  Text escape characters, such as \n, are not processed inside
of style override sets.


DBCS support and brace escapes
==============================
Subtitler ignores open braces and backslashes in the second character
of text encoded using a double-byte character set (DBCS).  This skip
is sensative to the font encoding currently in use, so in the dialogue
text

	{\fe128}ソNOボマ{\fe0}ソNOボマ

the section using Shift-JIS (\fe128) decodes to three katakana glyphs
and two Latin letters, while the second gives two italic f's, an O,
a hard line break, and an illegal style override.  (No, it doesn't
actually mean anything in Japanese.  At least, I hope.)  DBCS processing,
like escapes, are suspended inside style overrides, but you shouldn't
need to use any double-byte characters in them.

Windows 95/98 do not normally have the necessary fonts or routines to
support rendering DBCS text, but installing the appropriate language
pack from Internet Explorer will suffice.  Japanese Language Support from
IE 5.01 has been tested to work under Windows 98, and Chinese and Korean
should work as well; you do not need to install the Input Method Editor
(IME) support.  Under Windows 2000, you can install the appropriate
language support from the Regional Control Panel.

If you are using third-party DBCS support that hooks into the font
routines of a non-DBCS (SBCS) version of Windows, such as NJWIN, you will
probably need to manually escape trailing bytes that are open braces or
backslashes, because Windows won't know how to skip over the trail bytes
and the subtitler will process them.  The above example would thus become:

	{\fe128}ソ\NOソ{ソ}{\fe0}ソNOボマ

This escaping is incompatible with both SSA and regular DBCS
processing, however.


Font encodings
==============
By default, ANSI_CHARSET encoding is used unless overridden with the encoding
setting in the style or with the {\fe} override.  You will need to specify
a different code page if the character you want is not in the standard
printable 8-bit set.  Some useful encodings are:

    ANSI_CHARSET            0
    DEFAULT_CHARSET         1
    SYMBOL_CHARSET          2
    SHIFTJIS_CHARSET        128
    HANGEUL_CHARSET         129
    HANGUL_CHARSET          129
    GB2312_CHARSET          134
    CHINESEBIG5_CHARSET     136
    OEM_CHARSET             255

Whether or not a given character set is available depends mainly on the font;
if you are running Windows NT/2000, Character Map can be used to check since
it supports Unicode on that platform.  Arial in particular, supports a large
number of encodings.  Note that a change in character set can result in a
change of font, if the current font doesn't support the encoding but another
font in the system does.

Double-check the subtitler's output if you are using a double-byte character
set (DBCS), such as Shift-JIS, since the word-wrapping hasn't been
extensively checked for DBCS compatibility although it should work fine.


Controlling word wrapping
=========================
Subtitler supports three modes for word wrapping:

* Manual mode (\q2):

  No lines are broken unless either a soft break (\n) or hard break (\N) is
  used.

* Automatic mode (\q1):

  Lines are broken automatically at margins.  Soft breaks are ignored but
  hard breaks work.  Automatic mode is recommended for text that is left-
  justified.

* Smart mode (\q0):

  Same as automatic mode except that the subtitler squeezes the margins as
  far as possible without adding an extra line, so that the lines come out
  more even.  This mode is best used for centered text.

The default mode may be selected in the filter options, and can also be
controlled by the {\q} style override.  However, only one wrapping mode can
be used per dialogue line -- it is not possible to use manual breaking on
one part and have the rest be automatic.


Space alignment
===============
Adding spaces at the beginning or end of dialogue lines or around \N's to
push text on-screen won't work with the filter, since it trims whitespace off
the ends of each processed line during word wrapping.  If you must force
space at the ends, use nonbreakable spaces (\h or Alt-0160).  Spaces are not
collapsed in the middle of a line.


Alignment overrides
===================
Alignment overrides {\a} are supported, but multiple \a's in the same dialogue
line should be avoided.  The reason is that SSA4 interprets multiple alignment
overrides differently in its full-screen renderer than in its preview window.
As a result,

    abc{\a3}def{\a1}

renders on the left in the script editor and on the right onto tape.  Subtitler
imitates the full-screen renderer and always uses the first alignment override
it finds.

It is not possible to use two alignments in the same subtitle -- the entire
subtitle always uses one particular alignment.


Antialiasing
============
Subtitler antialiases all fonts by default at an 8x8 setting, i.e. it renders
*all* glyphs at eight times normal size and reduces it down.  The result is
crisp, high-quality text.  You can turn off font antialiasing by disabling the
advanced rasterizer, but beware that a number of features will stop working if
you disable it: borders will be forced to one pixel, scrolls and banners will
not work, and no shadows will be drawn.


Shadows and borders
===================
Shadows always render as dropshadows of the outline with 50% translucency;
borders render at a given circular radius from the text.  Shadows and borders
antialias along with the text.  Note that large borders should be avoided since
it is possible for borders to overlap text.

Although not recommended, the filter permits a border of size zero (0), and
will turn off the border in response.

Shadows and borders are always specified in absolute pixel values, regardless
of the PlayResY: tag.  Otherwise, most shadows and borders from existing
scripts would be too small to be seen.


The *Default style problem
==========================
Asterisks are ignored at the beginning of style names, so the *Default style
is really the Default style.

Sub Station Alpha saves your default style in the script under the *Default
tag.  However, its playback engine always interprets the Default style as
whatever style is default on your computer, not the Default style in the
script.  Subtitler does not have access to the SSA settings and will use the
Default style actually in the script.  Thus, if you attempt to use the script
on another computer it may be rendered differently by SSA and subtitler.  To
fix this problem, rename the Default style in the .ssa file to force SSA to
use the style defined in the script or modify the script to match your SSA
settings.

In general, you should avoid using *Default, and define a custom style instead.


Karaoke
=======
Karaoke is the highlighting of text in response to a song, so that others can
sing along to it.  Apply this feature with caution, since many people shouldn't
be allowed to sing in the first place.

Activate karaoke by specifying {\k} or {\K} style overrides in the text:

    {\k50}This takes half a second {\K100}This takes a second

The number indicates the hundredths of seconds it should take to highlight the
text; {\k} snaps the text on like SSA, while {\K} gradually highlights from
the left.  In both cases, text is changed from the secondary style color to
the active color, which is the primary style color unless overridden.

You do not need to ensure that the sum of the \k tags adds up to the event
length.  If the total karaoke time is too short, the event will simply be
displayed fully highlighted for the remainder, and if it is too long, it is
cut off.  The "Karaoke" event type in the dialogue line is also unnecessary,
so you may apply other events to the karaoke line if you wish and both effects
will take place.

A {\k0} or {\K0} tag acts as a brickwall to karaoke since it caps the end of
the last karaoke string and activates immediately to the primary color once
the time has passed.  Subtitler tracks karaoke even in the same override, so
{\K50\K50} pauses for half a second and then runs the next string of text for
another half second.


Banner and scroll
=================
Text can be scrolled horizontally or upwards by making use of the Banner and
Scroll Up effects.  These effects are specified in the effects portion of the
dialogue control line:

Dialogue: Marked=0,0:02:02.00,0:02:17.00,MainT,,0000,0000,0000,Banner;30,Text is fun.  We like scrolling text.
Dialogue: Marked=0,0:02:02.00,0:02:17.00,MainT,,0000,0000,0000,Scroll Up;40;50;120,{\q1}You can scroll lots of text up this way. Personally, I find it a little annoying to have a lot of text coming up very slowly on screen, but it's a useful technique to have for credits and other long lists in tabular form.

The Banner effect scrolls text left from the right border and takes a single
argument, the speed of the scroll.  Scroll Up takes three arguments, two
being top and bottom Y positions to define a window, and the third being
the speed.  If the Y window is (0,0), the entire screen is used.  In both
cases, the speed is five times the number of seconds the text should scroll
before just hitting the opposite border, so a speed of 40 means an 8-second
window.  This is considerably different from SSA's interpretation of the
speed as a delay parameter, but it's also easier to use.

Animated effects are automatically excluded from collisions.  They will never
move away from their designated path, nor will they cause other text to move.
This means you will need to take care not to overlay animations on top of
regular dialogue unless you specifically intend to do so.

NOTE: The SSA4 script format document says that the order of arguments for
      Scroll Up is y1;y2;delay.  This is incorrect.  The proper order of
	  arguments is delay;y1;y2.


Fast reload
===========
The subtitler always reloads the script whenever it is reinitialized by
VirtualDub, which is at the beginning of any preview or save, and after some
menu commands (mostly those that change the filter chain or input video).
This means that you can edit the script in SSA or Notepad, and see the changes
in the next preview without having to modify any settings.



Using Sub Station Alpha in wave timing mode to time against AVI
===============================================================
The most efficient way to create scripts for subtitler is to use SSA's wave
timing mode, which allows you to quickly generate script timings from a sound
file.  You can create the wave file necessary to do this by copying it out
of the source video file in VirtualDub.  However, the current version of
SSA at this time (4.08) does not accept stereo or 16-bit WAV files.  To
generate a WAV file of the correct format:

* Change the audio mode to Full Processing Mode.
* Make sure the audio compression setting is set to PCM (uncompressed).
* In Audio > Conversion, select 8-bit and mono.
* Use File > Save WAV to create the WAV file.

You need to do this with the final video source and settings that you are
going to use to subtitle -- your range settings and cuts must be the same,
or the WAV file will not match the video fed into subtitler.  VirtualDub
1.4.6 and earlier save both range settings and cuts into .vcf files, so
the Load/Save Processing Settings commands will allow you to preserve these
should you need to close VirtualDub between the Save WAV and final Save AVI
operations.  Another way to handle this problem is to create a batch job
for the final process when you do the Save WAV -- subtitler will reload
the script from the specified filename when the job actually executes.

Incidentally, you can cut/copy/paste from dialogue text in SSA by right-
clicking on the text pane.


Saturated colors
================
Highly saturated colors such as deep red should be avoided, particularly if
you are going to output the result to composite video or compress the result
with a low-bandwidth codec.  The reason is that sharp color edges are poorly
handled by the luminance/chrominance encoding in both cases.  You will get
better results if you use less saturated colors and ensure that your border and
text colors are far apart in brightness; yellow on black is a popular choice,
and mild green is also used.


Sub Station Alpha unsupported features
======================================
The following SSA features are not supported:

*  Audio, video, and program exec events.
*  Color collision resolution (primary is always used).
*  Embedded fonts and images.


[V4 Styles]
===========
Subtitler determines whether a script is in SSA v2 or v4 format by the
presence of the [V4 Styles] group marker.  The only difference in the
filter's behavior between V2 and V4 mode is that the BorderStyle entry in the
style is assumed to be absent in V2 mode, whereas it is skipped in V4 mode.
If you find that your outline, shadow, alignment, margins, and encoding
values in your style entries are being interpreted incorrectly, make sure
the [V4 Styles] marker is present above them.


Creating SSA scripts from scratch
=================================
Scripts can also be created manually with a text editor.  The best way to do
this is by picking up a copy of the SSA script format from the SSA web site,
but you can start with this:

    [Script Info]
    PlayResY: 480

    [V4 Styles]
    Style: *Default,Arial,22,8454143,8454143,8454143,2039583,0,0,2,4,2,30,30,20

    [Events]
    Dialogue: Marked=0,0:00:00.00,0:00:01.00,*Default,Name,0000,0000,0000,!Effect,Your text goes here

The style fields are (from Kotus' reference):

    Name, Fontname, Fontsize, PrimaryColor, SecondaryColor, TertiaryColor,
       BackColor, Bold, Italic, BorderStyle, Outline, Shadow, Alignment,
       MarginL, MarginR, MarginV, AlphaLevel, Encoding

AlphaLevel is not used; BorderStyle should be left 1, and bold/italic should
be 0 for off and -1 for on.  Alignment should be 1 for left, 2 for center,
or 3 for right, with an additional 4 added on for align up and 8 for vertical
align center.

All four colors are specified as 24-bit BGR, which is backwards from HTML's
#rrggbb.  SSA unfortunately writes out its colors in a single unreadable
decimal number, but both SSA and the subtitler will also accept color values
specified using Microsoft Basic hex notation (&Hxxxxxx).  Thus, for a yellow,
specify &H00ffff, and for cyan, use &Hffff00.  You can use Windows Calculator
in Scientific mode to convert between decimal and hex color values.

Dialogue events have this form (again, from Kotus):

    Marked, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text

Marked and Name are ignored by the subtitler; MarginL/R/V should be 0 to use
the default style values.  Start and End must be specified as h:mm:ss.tt, with
't' being hundredths of a second.

Be careful to not wordwrap or otherwise break any script lines, since this is
not permitted by the format.  Also, under Windows NT/2000, don't save out
Unicode from Notepad, or programs will be quite unhappy.



Known issues
============
Due to a bug in VirtualDub 1.3d, this filter cannot be used in frameserver
mode with that version.  The bug was first fixed in VirtualDub 1.4.

Special thanks to paQ, who found a number of bugs in earlier releases
and helped immensely in the development of this filter.


Contact
=======
Please email comments, bug reports, and suggestions to
<phaeron@virtualdub.org>.  I can't guarantee that I'll be able to respond,
but I'll do what I can.  The website for this filter is the same as the
VirtualDub website -- http://www.virtualdub.org/.

Have fun!

-- Avery Lee <phaeron@virtualdub.org>
   March 3, 2002


Changelog
=========
Version 2.3:

* Fix for addressing error in rasterizer that caused all subtitles to be
  off by one pixel, and the filter to occasionally crash.  Thanks to gabest
  for finding this one.

* Added some quick optimizations to the rasterizer to boost curve
  rasterization speed -- basically, the old fist/fistp trick.  Too lazy
  to code a proper Bezier subdivision rasterizer.

----------------------------------
Version 2.2:

* Fix for crash on {...}\n construct.

* Fixed rasterizer to handle the case where a font outline has a linear
  Bezier curve, i.e. the curve is actually a line, and avoid dividing by
  zero.

----------------------------------
Version 2.1:

* Added workarounds for the limitations of 16-bit GDI, which caused severe
  wordwrap and rasterization failures in some cases.  Windows 95/98 sucks.

* Did some work to try to make the filter DBCS clean.  Backslashes and
  open braces are no longer interpreted in the trailing bytes of text when
  DBCS encodings are active.

* You can now escape open braces (\{) to display them.

* All text is now converted to Unicode before display.  This allows CJK text
  to display on SBCS platforms with the appropriate IE support installed.

* Word wrap behavior is now adjustable on a per-item basis, and \n works
  as a line break if word wrapping is disabled.

* Relaxed the syntax restrictions for time fields; in particular, the
  0:00:00:00 syntax in the SSA script document is now allowed, even though
  it's different from the 0:00:00.00 syntax used by SSA4.  SSA itself doesn't
  care what separators you use.

* Changed the ordering of fields in the Scroll Up effect to Scroll Up;d;y;y
  to match SSA.

----------------------------------
Version 2.0:

* New antialiased font rasterizer working directly off of Bezier curve
  outlines.  Much, much faster and can support arbitrary rotation, scaling,
  and shearing transforms, although these capabilities are not yet used.

* Added support for the Banner and Scroll up effects.

* Added karaoke.  The hacks that people were forced to do for karaoke in
  V1.3b were too embarrassingly bad for me to let this go.

* I accidentally disabled font caching in the last release.  Whoops.
  This only really speeds up the non-antialiased (i.e. sucky) mode,
  so no big deal.

* Dialogue line sorting now checks the order lines are submitted to
  break ties.  Before, the sort was not stable, which meant that
  two lines that had the same start time essentially appeared in
  random order, and you couldn't tell which would end up on top
  if the lines collided.  Now, two events with the same start time
  always issue to the collision handler in-order, so the second
  event always collision resolves against the first.

* Font encodings other than ANSI_CHARSET are now supported thanks
  to Karel Suhajda, both in styles and in the {\fe} style override.
  The wordwrapping code is likely not DBCS clean, however, so this
  may not work with some encodings like Shift-JIS. (I can't test
  this, since both my computer and my brain are configured for US
  English.)

----------------------------------
Fixed in version 1.3b:

* Fixed a critical smart wordwrapping problem.  The smart wrapping works
  by crunching the margins until an extra line drops out.  If the
  subtitler encountered a dialogue line like:

    w\Nw\Nw

  it hung in a loop because none of the lines would ever wrap.  The
  filter now stops smart wrapping when this occurs.

----------------------------------
Fixed in version 1.3a:

* Fixed some memory leaks and stability issues.

----------------------------------
Fixed in version 1.3:

* Style overrides did not always work properly, because subtitler skipped
  too many characters after the first override.

* After word-wrapping, all lines from a dialogue line were left-justified
  relative to each other, even though the whole was justified correctly.

* \n (soft break) was accepted as a line break, but \N (hard break) was
  not.

* Outlines could overlap text if style overrides were used in between
  non-whitespace characters (i.e. err{\i1}or).

----------------------------------
Fixed in version 1.2:

* Style name checking ignores leading *'s.

----------------------------------
Fixed in version 1.1:

* Subtitles work past one hour.
