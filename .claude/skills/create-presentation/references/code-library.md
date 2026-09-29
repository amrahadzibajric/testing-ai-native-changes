# Code Library

Referenced from `../SKILL.md`. All the copy-paste code for a generated script, in the order you'd assemble it: brand system first (always needed), then the slide patterns you pick for the outline, then the automatic appendix (unless the user opts out). See `references/examples.md` for a complete script that assembles these pieces end-to-end.

## Part 1: Brand System — Dimensions, Constants, Logo, Helpers

### Slide dimensions

All slides are widescreen 13.333" × 7.5" (which matches 1280×720 pixels at 96 DPI, identical to the HTML skill canvas). Set this at the top of the script:

```python
from pptx import Presentation
from pptx.util import Inches, Pt, Emu

prs = Presentation()
prs.slide_width = Inches(13.333)
prs.slide_height = Inches(7.5)
```

### Brand constants

Copy this block into every script. These are the design tokens, equivalent to the CSS custom properties in the HTML skill.

```python
from pptx.dml.color import RGBColor
from pptx.util import Inches, Pt

# Brand colors
AP_LIME          = RGBColor(0xc1, 0xd8, 0x2f)
AP_DARK_LIME     = RGBColor(0x9f, 0xa6, 0x17)
AP_LIGHT_LIME    = RGBColor(0xe4, 0xe7, 0x6d)
AP_LIGHT_GRAY    = RGBColor(0xaf, 0xb8, 0xbf)
AP_COOL_GRAY     = RGBColor(0x83, 0x8d, 0x8f)
AP_DARK_COOL_GRAY= RGBColor(0x69, 0x6a, 0x6d)
AP_DARK_GRAY     = RGBColor(0x5c, 0x6f, 0x7c)
AP_BLACK         = RGBColor(0x1f, 0x1f, 0x1f)
AP_WHITE         = RGBColor(0xff, 0xff, 0xff)
AP_BG_GRAY       = RGBColor(0xea, 0xef, 0xf2)
AP_CREAM         = RGBColor(0xff, 0xff, 0xf0)
AP_SKY_BLUE      = RGBColor(0x33, 0x9c, 0xcc)

# Typography (Segoe UI primary, Open Sans fallback)
# python-pptx only sets one font name per run, so use Segoe UI and rely on
# PowerPoint's font substitution to fall back to Open Sans if Segoe UI is missing.
FONT = 'Segoe UI'

# Type scale (in points)
PT_COVER_TITLE   = 44
PT_TITLE         = 28
PT_SECTION_DIV   = 40   # for section divider slides
PT_EYEBROW       = 11
PT_LEDE          = 16
PT_H3            = 16
PT_BODY          = 13
PT_KPI_VALUE     = 40
PT_KPI_LABEL     = 11
PT_QUOTE         = 26
PT_META_LABEL    = 9
PT_META_VALUE    = 12
PT_CONFIDENTIAL  = 8

# Layout spacing (in inches)
SLIDE_W          = 13.333
SLIDE_H          = 7.5
PAD_X            = 0.75    # left/right padding for content
PAD_Y            = 0.67    # top/bottom padding for content
COVER_PAD_X      = 1.0
COVER_PAD_Y      = 1.0
LOGO_W_CONTENT   = 1.46    # 140 px equivalent
LOGO_H_CONTENT   = 0.42
LOGO_W_COVER     = 2.08    # 200 px equivalent
LOGO_H_COVER     = 0.59
LIME_STRIPE_W    = 0.08    # lime accent stripe on cover left edge
```

### Logo handling

Three logos ship with the skill, all transparent PNGs:

- `assets/logo-color.png`: gray and lime swoosh with dark gray "AUTHORITY PARTNERS" wordmark. The default, used on light backgrounds.
- `assets/logo-white.png`: white swoosh and white wordmark. Used on dark backgrounds (cover, closing, any dark-themed slide).
- `assets/logo-black.png`: the same mark flattened to solid black (swoosh and wordmark). Used where the logo sits over a busy or photographic light area and the color mark reads poorly, currently the appendix "Why Clients Choose Us" slide (library slide 6, the NPS slide). Selected via `add_logo(slide, black=True)`.

The skill directory path is available at runtime. Resolve it like this at the top of the generated script:

```python
import os
SKILL_DIR = os.path.dirname(os.path.abspath(__file__)) if '__file__' in dir() else os.path.expanduser('~/authority-partners-slides-pptx')
LOGO_COLOR = os.path.join(SKILL_DIR, 'assets', 'logo-color.png')
LOGO_WHITE = os.path.join(SKILL_DIR, 'assets', 'logo-white.png')
LOGO_BLACK = os.path.join(SKILL_DIR, 'assets', 'logo-black.png')
```

If the skill is being executed from a different working directory, the generated script should accept the skill directory as an argument or have it hard-coded. For Claude executing the skill, the path is wherever the skill was installed (typically a path Claude has visibility into via the skill system).

If a logo file cannot be read for any reason, fall back to a text mark: a small text box that reads "AUTHORITY PARTNERS" in 11pt bold, letter-spaced uppercase, in `AP_DARK_COOL_GRAY` for light slides or `AP_WHITE` for dark slides, at the same top-right position.

### Helper functions

Copy these once into the script. They handle chrome (logo, confidential marker) and common text styling so each slide pattern stays short.

```python
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR

def _set_run(run, *, text=None, size=None, bold=False, color=None, font=FONT, italic=False):
    if text is not None:
        run.text = text
    run.font.name = font
    if size is not None:
        run.font.size = Pt(size)
    run.font.bold = bold
    run.font.italic = italic
    if color is not None:
        run.font.color.rgb = color

def add_text(slide, *, text, left, top, width, height, size, bold=False, italic=False,
             color=AP_BLACK, align=PP_ALIGN.LEFT, anchor=MSO_ANCHOR.TOP,
             char_spacing=None, all_caps=False, shrink=False):
    box = slide.shapes.add_textbox(Inches(left), Inches(top), Inches(width), Inches(height))
    tf = box.text_frame
    tf.word_wrap = True
    tf.margin_left = tf.margin_right = tf.margin_top = tf.margin_bottom = 0
    tf.vertical_anchor = anchor
    p = tf.paragraphs[0]
    p.alignment = align
    run = p.add_run()
    display = text.upper() if all_caps else text
    _set_run(run, text=display, size=size, bold=bold, color=color, italic=italic)
    # Character spacing (letter-spacing equivalent). python-pptx exposes this via XML.
    if char_spacing is not None:
        from pptx.oxml.ns import qn
        rPr = run._r.get_or_add_rPr()
        rPr.set('spc', str(char_spacing))  # in 1/100 pt units
    # Shrink text to fit the shape (normAutofit — equivalent to PowerPoint's
    # "Shrink text on overflow"). Use on any fixed-height box where text length varies.
    if shrink:
        from pptx.oxml.ns import qn
        from lxml import etree
        bodyPr = tf._txBody.find(qn('a:bodyPr'))
        if bodyPr is not None:
            for tag in [qn('a:noAutofit'), qn('a:spAutoFit'), qn('a:normAutofit')]:
                el = bodyPr.find(tag)
                if el is not None:
                    bodyPr.remove(el)
            etree.SubElement(bodyPr, qn('a:normAutofit'))
    return box

def fit_text_to_box(text, width_in, height_in, *, size, bold=False, line_spacing=1.25, min_size=7):
    """
    Return the largest font size <= size at which text fits in the box.
    Uses a character-width approximation: accurate enough for slide copy.
    Use this instead of shrink=True for any fixed-height card body or heading text.
    """
    char_w = 0.56 if bold else 0.50   # fraction of em
    while size >= min_size:
        chars_per_ln = (width_in * 72) / (size * char_w)
        lines, col = 1, 0
        for word in text.split():
            wlen = len(word) + 1
            if col + wlen > chars_per_ln and col > 0:
                lines += 1
                col = wlen
            else:
                col += wlen
        if (lines * size * line_spacing / 72) <= height_in:
            return size
        size -= 1
    return min_size


def add_eyebrow(slide, *, text, left, top, color=AP_DARK_LIME, width=8.0):
    return add_text(slide, text=text, left=left, top=top, width=width, height=0.3,
                    size=PT_EYEBROW, bold=True, color=color, all_caps=True, char_spacing=180)

def add_title(slide, *, text, left, top, width, height, color=AP_BLACK, size=PT_TITLE):
    box = add_text(slide, text=text, left=left, top=top, width=width, height=height,
                   size=size, bold=True, color=color)
    box.text_frame.paragraphs[0].line_spacing = 1.15
    return box

def add_lede(slide, *, text, left, top, width, height=1.2, color=AP_DARK_COOL_GRAY):
    box = add_text(slide, text=text, left=left, top=top, width=width, height=height,
                   size=PT_LEDE, color=color)
    box.text_frame.paragraphs[0].line_spacing = 1.5
    return box

def add_rect(slide, *, left, top, width, height, fill=None, line=None, line_width=None):
    shape = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE,
        Inches(left), Inches(top), Inches(width), Inches(height))
    if fill is None:
        shape.fill.background()
    else:
        shape.fill.solid()
        shape.fill.fore_color.rgb = fill
    if line is None:
        shape.line.fill.background()
    else:
        shape.line.color.rgb = line
        if line_width is not None:
            shape.line.width = Pt(line_width)
    shape.shadow.inherit = False
    return shape

def add_rounded_rect(slide, *, left, top, width, height, fill, radius=0.06):
    shape = slide.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE,
        Inches(left), Inches(top), Inches(width), Inches(height))
    shape.adjustments[0] = radius  # 0..1 where ~0.05 is a soft corner
    shape.fill.solid()
    shape.fill.fore_color.rgb = fill
    shape.line.fill.background()
    shape.shadow.inherit = False
    return shape

def set_slide_background(slide, color):
    fill = slide.background.fill
    fill.solid()
    fill.fore_color.rgb = color

def add_logo(slide, *, dark=False, black=False):
    """Place the logo top-right on the slide.
    dark=True uses the white version (for dark backgrounds).
    black=True uses the all-black version (for the logo sitting over a busy or
    photographic light area, e.g. library slide 6). black takes precedence over
    dark and always uses the content size and position."""
    if black:
        path = LOGO_BLACK
    else:
        path = LOGO_WHITE if dark else LOGO_COLOR
    if dark and not black:
        # Cover/closing: larger logo, slightly inset
        left = SLIDE_W - LOGO_W_COVER - 0.5
        top  = 0.4
        slide.shapes.add_picture(path, Inches(left), Inches(top),
                                 width=Inches(LOGO_W_COVER), height=Inches(LOGO_H_COVER))
    else:
        # Content: smaller logo, top-right
        left = SLIDE_W - LOGO_W_CONTENT - 0.4
        top  = 0.3
        slide.shapes.add_picture(path, Inches(left), Inches(top),
                                 width=Inches(LOGO_W_CONTENT), height=Inches(LOGO_H_CONTENT))

def add_confidential(slide, *, on_dark=False):
    color = AP_COOL_GRAY if on_dark else AP_LIGHT_GRAY
    add_text(slide, text='CONFIDENTIAL', left=PAD_X, top=SLIDE_H - 0.4, width=2.0, height=0.25,
             size=PT_CONFIDENTIAL, bold=True, color=color, all_caps=True, char_spacing=200)

def blank_slide(prs):
    """Add a new slide using the first available blank-ish layout. The library
    PPTX bundles a single DEFAULT layout at index 0, while a fresh Presentation
    has the standard 7 layouts with blank at index 6. Try [6] first, fall back."""
    try:
        return prs.slides.add_slide(prs.slide_layouts[6])
    except IndexError:
        return prs.slides.add_slide(prs.slide_layouts[0])
```

## Part 2: Slide Patterns

Each pattern below is a function that takes the presentation and slide-specific content, and adds one slide. Copy the functions you need into the generated script — they depend on the brand constants and helper functions in Part 1 above. See `../SKILL.md`'s "Quick reference: which slide pattern when" for guidance on which pattern to reach for.

### 1. Cover slide

The cover uses a single text box for the title (main paragraph + optional accent paragraph). This way the accent line flows naturally below the main title even when the main title wraps to multiple lines.

```python
def add_cover_slide(prs, *, eyebrow=None, title_main, title_accent=None, tagline=None, meta=None):
    """
    eyebrow:      optional small label above the title (e.g. "CAPABILITIES OVERVIEW")
    title_main:   main title text (white)
    title_accent: optional accent phrase shown in lime green, flows on next line
    tagline:      optional one-sentence framing under the title
    meta:         optional list of (label, value) pairs for the footer row
    """
    slide = blank_slide(prs)
    set_slide_background(slide, AP_BLACK)
    # Lime accent stripe down the left edge
    add_rect(slide, left=0, top=0, width=LIME_STRIPE_W, height=SLIDE_H, fill=AP_LIME)
    # White logo top-right (no plate)
    add_logo(slide, dark=True)
    # Eyebrow
    if eyebrow:
        add_eyebrow(slide, text=eyebrow, left=COVER_PAD_X, top=COVER_PAD_Y, color=AP_LIME)
    # Title (main + optional accent in one text box, two paragraphs)
    title_top = COVER_PAD_Y + (0.4 if eyebrow else 0)
    title_h = 3.2  # enough vertical space for 2-3 lines at 44pt
    box = slide.shapes.add_textbox(Inches(COVER_PAD_X), Inches(title_top),
                                   Inches(10.5), Inches(title_h))
    tf = box.text_frame
    tf.word_wrap = True
    tf.margin_left = tf.margin_right = tf.margin_top = tf.margin_bottom = 0
    tf.vertical_anchor = MSO_ANCHOR.TOP
    # Main title paragraph (white)
    p1 = tf.paragraphs[0]
    p1.line_spacing = 1.15
    p1.alignment = PP_ALIGN.LEFT
    r1 = p1.add_run()
    _set_run(r1, text=title_main, size=PT_COVER_TITLE, bold=True, color=AP_WHITE)
    # Accent paragraph (lime)
    if title_accent:
        p2 = tf.add_paragraph()
        p2.line_spacing = 1.15
        p2.alignment = PP_ALIGN.LEFT
        r2 = p2.add_run()
        _set_run(r2, text=title_accent, size=PT_COVER_TITLE, bold=True, color=AP_LIME)
    # Tagline (positioned below the title box)
    if tagline:
        tag_top = title_top + title_h + 0.1
        add_lede(slide, text=tagline, left=COVER_PAD_X, top=tag_top,
                 width=10.0, color=RGBColor(0xc6, 0xcc, 0xd0))
    # Meta row at the bottom
    if meta:
        add_rect(slide, left=COVER_PAD_X, top=SLIDE_H - 1.55, width=10.0, height=0.01,
                 fill=AP_DARK_COOL_GRAY)
        col_w = 10.0 / max(len(meta), 1)
        for i, (label, value) in enumerate(meta):
            x = COVER_PAD_X + i * col_w
            add_text(slide, text=label, left=x, top=SLIDE_H - 1.35, width=col_w, height=0.2,
                     size=PT_META_LABEL, bold=True, color=AP_COOL_GRAY,
                     all_caps=True, char_spacing=180)
            add_text(slide, text=value, left=x, top=SLIDE_H - 1.10, width=col_w, height=0.3,
                     size=PT_META_VALUE, bold=True, color=AP_WHITE)
    add_confidential(slide, on_dark=True)
    return slide
```

### 2. Agenda slide

```python
def add_agenda_slide(prs, *, eyebrow="TODAY'S CONVERSATION", title="What We'll Cover", items):
    """
    items: list of dicts like {'title': 'Who we are', 'description': 'Brief framing line.'}
    """
    slide = blank_slide(prs)
    set_slide_background(slide, AP_WHITE)
    add_logo(slide, dark=False)
    add_eyebrow(slide, text=eyebrow, left=PAD_X, top=PAD_Y)
    add_title(slide, text=title, left=PAD_X, top=PAD_Y + 0.35, width=11.5, height=0.8)
    # List, each item is 3 lines: number, h3, description
    list_top = PAD_Y + 1.4
    item_h = (SLIDE_H - list_top - 1.0) / max(len(items), 1)
    for i, item in enumerate(items):
        y = list_top + i * item_h
        # Horizontal divider above each item
        add_rect(slide, left=PAD_X, top=y - 0.05, width=11.5, height=0.01,
                 fill=AP_LIGHT_GRAY)
        # Number (lime)
        add_text(slide, text=f"{i+1:02d}", left=PAD_X, top=y + 0.1, width=0.7, height=0.5,
                 size=20, bold=True, color=AP_DARK_LIME)
        # Title and description
        add_text(slide, text=item['title'], left=PAD_X + 0.9, top=y + 0.1, width=10.5, height=0.4,
                 size=PT_H3, bold=True, color=AP_BLACK)
        add_text(slide, text=item.get('description', ''), left=PAD_X + 0.9, top=y + 0.5,
                 width=10.5, height=0.4, size=PT_BODY, color=AP_DARK_COOL_GRAY)
    add_confidential(slide)
    return slide
```

### 3. Section divider

```python
def add_section_divider(prs, *, number, title, lede=None):
    """A breath between major sections of a longer deck."""
    slide = blank_slide(prs)
    set_slide_background(slide, AP_WHITE)
    add_logo(slide, dark=False)
    # Number, in lime, all caps
    add_eyebrow(slide, text=f"SECTION {number}", left=1.5, top=2.8, color=AP_DARK_LIME)
    # Big title
    add_title(slide, text=title, left=1.5, top=3.1, width=10.5, height=2.0, size=PT_SECTION_DIV)
    if lede:
        add_lede(slide, text=lede, left=1.5, top=5.0, width=9.0)
    add_confidential(slide)
    return slide
```

### 4. Standard content

```python
def add_content_slide(prs, *, eyebrow, title, lede=None, bullets=None, paragraphs=None):
    """
    The workhorse. Most slides are this.
    bullets:    optional list of strings, displayed with lime accent dash
    paragraphs: optional list of strings, displayed as plain prose
    Use one or the other (or both, with paragraphs first).
    """
    slide = blank_slide(prs)
    set_slide_background(slide, AP_WHITE)
    add_logo(slide, dark=False)
    add_eyebrow(slide, text=eyebrow, left=PAD_X, top=PAD_Y)
    add_title(slide, text=title, left=PAD_X, top=PAD_Y + 0.35, width=11.5, height=0.9)
    cursor_y = PAD_Y + 1.4
    if lede:
        add_lede(slide, text=lede, left=PAD_X, top=cursor_y, width=11.0)
        cursor_y += 1.4
    if paragraphs:
        for para in paragraphs:
            add_text(slide, text=para, left=PAD_X, top=cursor_y, width=11.0, height=0.8,
                     size=PT_BODY, color=AP_BLACK)
            cursor_y += 0.8
    if bullets:
        for b in bullets:
            # lime dash
            add_rect(slide, left=PAD_X, top=cursor_y + 0.15, width=0.18, height=0.03,
                     fill=AP_LIME)
            add_text(slide, text=b, left=PAD_X + 0.3, top=cursor_y, width=10.7, height=0.6,
                     size=PT_BODY, color=AP_BLACK)
            cursor_y += 0.45
    add_confidential(slide)
    return slide
```

### 4a. Numbered card grid

Use when a slide has 4–6 short items (key points, findings, criteria) that would leave empty space as plain bullets. Cards fill the slide completely; numbered 01–N with a lime accent stripe.

```python
def add_grid_slide(prs, *, eyebrow, title, lede=None, items, cols=3):
    """
    items: list of str. cols: 2 or 3 (default 3).
    Rows auto-calculated. Cards fill left-to-right, top-to-bottom.
    """
    import math
    rows   = math.ceil(len(items) / cols)
    slide  = blank_slide(prs)
    set_slide_background(slide, AP_WHITE)
    add_logo(slide, dark=False)
    add_eyebrow(slide, text=eyebrow, left=PAD_X, top=PAD_Y)
    add_title(slide, text=title, left=PAD_X, top=PAD_Y + 0.35, width=11.5, height=0.8)
    cursor_y = PAD_Y + 1.35
    if lede:
        add_lede(slide, text=lede, left=PAD_X, top=cursor_y, width=11.0, height=0.6)
        cursor_y += 0.75
    GAP      = 0.20
    grid_top = cursor_y + 0.10
    grid_btm = SLIDE_H - 0.50
    card_w   = (11.5 - GAP * (cols - 1)) / cols
    card_h   = (grid_btm - grid_top - GAP * (rows - 1)) / rows
    for i, text in enumerate(items):
        row = i // cols
        col = i % cols
        x   = PAD_X + col * (card_w + GAP)
        y   = grid_top + row * (card_h + GAP)
        add_rect(slide, left=x, top=y, width=card_w, height=card_h, fill=AP_BG_GRAY)
        add_rect(slide, left=x, top=y, width=card_w, height=0.055, fill=AP_LIME)
        add_text(slide, text=f'{i+1:02d}', left=x + 0.25, top=y + 0.20,
                 width=card_w - 0.5, height=0.28, size=PT_EYEBROW, bold=True,
                 color=AP_DARK_LIME, all_caps=True, char_spacing=180)
        add_text(slide, text=text, left=x + 0.25, top=y + 0.55,
                 width=card_w - 0.5, height=card_h - 0.68, size=PT_H3, bold=True,
                 color=AP_BLACK)
    add_confidential(slide)
    return slide
```

**Quick guidance:**
- 6 items → `cols=3` (3×2 grid)
- 4 items → `cols=2` (2×2 grid)
- 3 items → `cols=3` (3×1 row) — or use `add_kpi_slide` if items have a numeric value

### 5. Two-column comparison

```python
def add_comparison_slide(prs, *, eyebrow, title, lede=None, left_col, right_col, right_accent=True):
    """
    left_col:  dict {'heading': str, 'bullets': [str, ...]}
    right_col: dict {'heading': str, 'bullets': [str, ...]}
    right_accent: True puts a lime border on the right column to suggest "us / our way"
    """
    slide = blank_slide(prs)
    set_slide_background(slide, AP_WHITE)
    add_logo(slide, dark=False)
    add_eyebrow(slide, text=eyebrow, left=PAD_X, top=PAD_Y)
    add_title(slide, text=title, left=PAD_X, top=PAD_Y + 0.35, width=11.5, height=0.8)
    if lede:
        add_lede(slide, text=lede, left=PAD_X, top=PAD_Y + 1.2, width=11.0, height=0.7)
    cols_top = PAD_Y + 2.2
    cols_h = SLIDE_H - cols_top - 0.6
    col_w = (11.5 - 0.3) / 2  # 0.3 gap
    for i, col in enumerate([left_col, right_col]):
        x = PAD_X + i * (col_w + 0.3)
        # Card fill
        fill = AP_CREAM if (i == 1 and right_accent) else AP_BG_GRAY
        add_rect(slide, left=x, top=cols_top, width=col_w, height=cols_h, fill=fill)
        # Lime left edge on the accent column
        if i == 1 and right_accent:
            add_rect(slide, left=x, top=cols_top, width=0.05, height=cols_h, fill=AP_LIME)
        # Heading inside card
        add_text(slide, text=col['heading'], left=x + 0.3, top=cols_top + 0.3, width=col_w - 0.6,
                 height=0.4, size=PT_H3, bold=True, color=AP_BLACK)
        # Bullets
        b_top = cols_top + 0.85
        for b in col['bullets']:
            dash_color = AP_LIME if (i == 1 and right_accent) else AP_COOL_GRAY
            add_rect(slide, left=x + 0.3, top=b_top + 0.13, width=0.16, height=0.03,
                     fill=dash_color)
            add_text(slide, text=b, left=x + 0.55, top=b_top, width=col_w - 0.85, height=0.5,
                     size=PT_BODY, color=AP_BLACK)
            b_top += 0.42
    add_confidential(slide)
    return slide
```

### 6. Pillar cards (3 or 4 across)

```python
def add_pillars_slide(prs, *, eyebrow, title, lede=None, pillars, dark=False):
    """
    pillars: list of dicts {'heading': str, 'description': str}
             Use 3 or 4 items; the function lays them out evenly.
    dark:    True renders on dark background with white title and lighter cards
    """
    slide = blank_slide(prs)
    bg = AP_BLACK if dark else AP_WHITE
    set_slide_background(slide, bg)
    add_logo(slide, dark=dark)
    add_eyebrow(slide, text=eyebrow, left=PAD_X, top=PAD_Y,
                color=AP_LIME if dark else AP_DARK_LIME)
    title_color = AP_WHITE if dark else AP_BLACK
    add_title(slide, text=title, left=PAD_X, top=PAD_Y + 0.35, width=11.5, height=0.8,
              color=title_color)
    if lede:
        lede_color = RGBColor(0xc6, 0xcc, 0xd0) if dark else AP_DARK_COOL_GRAY
        add_lede(slide, text=lede, left=PAD_X, top=PAD_Y + 1.2, width=11.0, height=0.7,
                 color=lede_color)
    n = len(pillars)
    gap = 0.25
    cards_top = PAD_Y + 2.4
    cards_h = SLIDE_H - cards_top - 0.6
    card_w = (11.5 - gap * (n - 1)) / n
    card_fill = RGBColor(0x2e, 0x2f, 0x33) if dark else AP_BG_GRAY
    body_color = RGBColor(0xc6, 0xcc, 0xd0) if dark else AP_DARK_COOL_GRAY
    h3_color = AP_WHITE if dark else AP_BLACK
    for i, p in enumerate(pillars):
        x = PAD_X + i * (card_w + gap)
        add_rect(slide, left=x, top=cards_top, width=card_w, height=cards_h, fill=card_fill)
        # Lime top edge
        add_rect(slide, left=x, top=cards_top, width=card_w, height=0.06, fill=AP_LIME)
        add_text(slide, text=p['heading'], left=x + 0.3, top=cards_top + 0.4,
                 width=card_w - 0.6, height=0.7, size=PT_H3, bold=True, color=h3_color)
        add_text(slide, text=p['description'], left=x + 0.3, top=cards_top + 1.3,
                 width=card_w - 0.6, height=cards_h - 1.5, size=PT_BODY, color=body_color)
    add_confidential(slide, on_dark=dark)
    return slide
```

### 7. Process / method flow

```python
def add_flow_slide(prs, *, eyebrow, title, lede=None, steps):
    """
    steps: list of dicts {'name': str, 'outcome': str}; 3 to 5 steps
    """
    slide = blank_slide(prs)
    set_slide_background(slide, AP_WHITE)
    add_logo(slide, dark=False)
    add_eyebrow(slide, text=eyebrow, left=PAD_X, top=PAD_Y)
    add_title(slide, text=title, left=PAD_X, top=PAD_Y + 0.35, width=11.5, height=0.8)
    if lede:
        add_lede(slide, text=lede, left=PAD_X, top=PAD_Y + 1.2, width=11.0, height=1.0)
    n = len(steps)
    gap = 0.2
    arrow_w = 0.3
    flow_top = PAD_Y + 2.3
    flow_h = 2.9
    # Each step gets a card; arrows between
    total_arrow_space = (n - 1) * arrow_w
    card_w = (11.5 - total_arrow_space - gap * (n - 1)) / n
    for i, step in enumerate(steps):
        x = PAD_X + i * (card_w + gap + arrow_w)
        add_rect(slide, left=x, top=flow_top, width=card_w, height=flow_h,
                 fill=AP_WHITE, line=AP_LIGHT_GRAY, line_width=0.75)
        # Step number (lime, all caps)
        add_text(slide, text=f"{i+1:02d}", left=x + 0.25, top=flow_top + 0.25,
                 width=card_w - 0.5, height=0.3, size=PT_EYEBROW, bold=True,
                 color=AP_DARK_LIME, all_caps=True, char_spacing=180)
        add_text(slide, text=step['name'], left=x + 0.25, top=flow_top + 0.7,
                 width=card_w - 0.5, height=0.8, size=PT_H3, bold=True, color=AP_BLACK)
        add_text(slide, text=step.get('outcome', ''), left=x + 0.25, top=flow_top + 1.55,
                 width=card_w - 0.5, height=flow_h - 1.65, size=PT_BODY,
                 color=AP_DARK_COOL_GRAY)
        # Arrow to the next step
        if i < n - 1:
            arrow_x = x + card_w + gap / 2
            add_text(slide, text='→', left=arrow_x, top=flow_top + flow_h / 2 - 0.2,
                     width=arrow_w, height=0.4, size=20, bold=True, color=AP_LIME,
                     align=PP_ALIGN.CENTER)
    add_confidential(slide)
    return slide
```

### 8. KPI / metrics

```python
def add_kpi_slide(prs, *, eyebrow, title, lede=None, kpis):
    """
    kpis: list of dicts {'value': str, 'label': str, 'description': str}
          Use 3 or 4 cards.
    """
    slide = blank_slide(prs)
    set_slide_background(slide, AP_WHITE)
    add_logo(slide, dark=False)
    add_eyebrow(slide, text=eyebrow, left=PAD_X, top=PAD_Y)
    add_title(slide, text=title, left=PAD_X, top=PAD_Y + 0.35, width=11.5, height=0.8)
    if lede:
        add_lede(slide, text=lede, left=PAD_X, top=PAD_Y + 1.2, width=11.0, height=0.7)
    n = len(kpis)
    gap = 0.25
    cards_top = PAD_Y + 2.6
    cards_h = SLIDE_H - cards_top - 0.6
    card_w = (11.5 - gap * (n - 1)) / n
    for i, k in enumerate(kpis):
        x = PAD_X + i * (card_w + gap)
        add_rect(slide, left=x, top=cards_top, width=card_w, height=cards_h, fill=AP_BG_GRAY)
        # Lime left border
        add_rect(slide, left=x, top=cards_top, width=0.06, height=cards_h, fill=AP_LIME)
        add_text(slide, text=k['value'], left=x + 0.3, top=cards_top + 0.4,
                 width=card_w - 0.6, height=0.9, size=PT_KPI_VALUE, bold=True, color=AP_BLACK)
        add_text(slide, text=k['label'], left=x + 0.3, top=cards_top + 1.3,
                 width=card_w - 0.6, height=0.4, size=PT_KPI_LABEL, bold=True,
                 color=AP_DARK_COOL_GRAY, all_caps=True, char_spacing=150)
        add_text(slide, text=k.get('description', ''), left=x + 0.3, top=cards_top + 1.85,
                 width=card_w - 0.6, height=cards_h - 2.0, size=PT_BODY,
                 color=AP_DARK_COOL_GRAY)
    add_confidential(slide)
    return slide
```

### 9. Quote / testimonial

```python
def add_quote_slide(prs, *, eyebrow='CLIENT VOICE', quote, name, role):
    slide = blank_slide(prs)
    set_slide_background(slide, AP_WHITE)
    add_logo(slide, dark=False)
    # Eyebrow at top of content area
    add_eyebrow(slide, text=eyebrow, left=PAD_X, top=PAD_Y + 0.5)
    # Big lime open quote mark (use Unicode U+201C, never plain ASCII ")
    add_text(slide, text='\u201c', left=PAD_X, top=PAD_Y + 1.1, width=1.2, height=2.0,
             size=120, color=AP_LIME)
    # Quote body
    quote_box = add_text(slide, text=quote, left=PAD_X + 0.9, top=PAD_Y + 2.0,
                         width=11.0, height=2.8, size=PT_QUOTE, color=AP_BLACK)
    quote_box.text_frame.paragraphs[0].line_spacing = 1.4
    # Attribution
    add_text(slide, text=name, left=PAD_X + 0.9, top=PAD_Y + 5.2, width=10.0, height=0.4,
             size=PT_H3, bold=True, color=AP_BLACK)
    add_text(slide, text=role, left=PAD_X + 0.9, top=PAD_Y + 5.6, width=10.0, height=0.3,
             size=PT_BODY, color=AP_DARK_COOL_GRAY)
    add_confidential(slide)
    return slide
```

**Important**: use the Unicode opening quote `\u201c` (left double quotation mark). The plain ASCII `"` renders as two vertical bars at large point sizes in LibreOffice and some PowerPoint configurations.

### 10. Closing

```python
def add_closing_slide(prs, *, eyebrow='THANK YOU', title_main, title_accent=None,
                      tagline=None, meta=None):
    """Same shape as cover but with a different mood. meta is for contact info."""
    return add_cover_slide(prs, eyebrow=eyebrow, title_main=title_main,
                           title_accent=title_accent, tagline=tagline, meta=meta)
```

### 11. Programme timeline (Gantt)

Gantt-style visual for delivery programmes. Rows are phases or tracks; columns are week numbers grouped under month headers. Numbered milestone badges appear at bar endpoints. An "ENGAGEMENT MILESTONES" legend section sits below the Gantt; an optional provider legend appears bottom-right.

```python
def add_timeline_slide(prs, *, eyebrow, title, subtitle=None,
                       phases, milestones, total_weeks,
                       months=None, legend=None):
    """
    phases: list of dicts
        {'label': str,             # row label; use \\n for a second line
         'start': int,             # start week, 1-based
         'end': int,               # end week, 1-based, inclusive
         'bar_color': RGBColor,    # default AP_LIME
         'label_color': RGBColor,  # default same as bar_color
         'milestone': int}         # optional badge number at bar right end

    milestones: list of dicts {'number': int, 'description': str}
        Displayed in two columns below the Gantt.

    total_weeks: int — total chart width in weeks.

    months: None = auto (4 weeks per month starting at W1), or list of dicts
        {'label': str, 'start': int, 'end': int}  # 1-based week numbers.

    legend: None or list of dicts {'label': str, 'color': RGBColor}
        Small colored boxes bottom-right (provider color legend).
    """
    CHART_L  = 0.50
    LABEL_W  = 1.30
    GANTT_L  = CHART_L + LABEL_W
    GANTT_R  = 13.05
    GANTT_W  = GANTT_R - GANTT_L
    week_w   = GANTT_W / total_weeks

    MONTH_H  = 0.26
    WEEK_H   = 0.20
    PHASE_H  = 0.40

    NAVY     = RGBColor(0x1e, 0x2c, 0x3c)   # dark header background
    BADGE_BG = RGBColor(0x55, 0x5f, 0x66)   # milestone and section badge color

    chart_top = 2.10 if subtitle else 1.60

    slide = blank_slide(prs)
    set_slide_background(slide, AP_WHITE)
    add_logo(slide, dark=False)
    add_eyebrow(slide, text=eyebrow, left=PAD_X, top=PAD_Y)
    add_title(slide, text=title, left=PAD_X, top=PAD_Y + 0.35, width=11.5, height=0.65)
    if subtitle:
        add_text(slide, text=subtitle, left=PAD_X, top=PAD_Y + 1.02,
                 width=10.0, height=0.26, size=PT_BODY, color=AP_DARK_COOL_GRAY)

    # Auto-build months at 4 weeks each
    if months is None:
        months = []
        w, m = 1, 1
        while w <= total_weeks:
            end = min(w + 3, total_weeks)
            months.append({'label': f'Month {m}', 'start': w, 'end': end})
            w += 4; m += 1

    # ── Month header row ───────────────────────────────────────────────
    add_rect(slide, left=GANTT_L, top=chart_top, width=GANTT_W, height=MONTH_H, fill=NAVY)
    for mo in months:
        mx = GANTT_L + (mo['start'] - 1) * week_w
        mw = (mo['end'] - mo['start'] + 1) * week_w
        if mo['start'] > 1:
            add_rect(slide, left=mx, top=chart_top, width=0.01, height=MONTH_H,
                     fill=RGBColor(0x3a, 0x4a, 0x5a))
        add_text(slide, text=mo['label'],
                 left=mx + 0.04, top=chart_top + 0.04,
                 width=mw - 0.08, height=MONTH_H - 0.08,
                 size=9, bold=True, color=AP_WHITE, align=PP_ALIGN.CENTER)

    # ── Week sub-header row ─────────────────────────────────────────────
    wk_top = chart_top + MONTH_H
    add_rect(slide, left=GANTT_L, top=wk_top, width=GANTT_W, height=WEEK_H, fill=AP_BG_GRAY)
    for w in range(1, total_weeks + 1):
        wx = GANTT_L + (w - 1) * week_w
        if w > 1:
            add_rect(slide, left=wx, top=wk_top, width=0.006, height=WEEK_H,
                     fill=AP_LIGHT_GRAY)
        add_text(slide, text=f'W{w}',
                 left=wx + 0.01, top=wk_top + 0.02,
                 width=week_w - 0.02, height=WEEK_H - 0.04,
                 size=6, color=AP_DARK_COOL_GRAY, align=PP_ALIGN.CENTER)

    # ── Phase rows ──────────────────────────────────────────────────────
    ph_top = wk_top + WEEK_H
    for idx, phase in enumerate(phases):
        rt  = ph_top + idx * PHASE_H
        rbg = AP_WHITE if idx % 2 == 0 else AP_BG_GRAY
        bc  = phase.get('bar_color', AP_LIME)
        lc  = phase.get('label_color', bc)

        # Row background (gantt area only)
        add_rect(slide, left=GANTT_L, top=rt, width=GANTT_W, height=PHASE_H, fill=rbg)
        add_rect(slide, left=CHART_L, top=rt + PHASE_H - 0.006,
                 width=GANTT_R - CHART_L, height=0.006, fill=AP_BG_GRAY)

        # Label box
        add_rect(slide, left=CHART_L, top=rt, width=LABEL_W, height=PHASE_H, fill=lc)
        lines = phase['label'].split('\n')
        if len(lines) == 1:
            add_text(slide, text=lines[0],
                     left=CHART_L + 0.07, top=rt + 0.07,
                     width=LABEL_W - 0.14, height=PHASE_H - 0.14,
                     size=9, bold=True, color=AP_WHITE,
                     align=PP_ALIGN.CENTER, anchor=MSO_ANCHOR.MIDDLE)
        else:
            add_text(slide, text=lines[0],
                     left=CHART_L + 0.07, top=rt + 0.03,
                     width=LABEL_W - 0.14, height=0.20,
                     size=9, bold=True, color=AP_WHITE, align=PP_ALIGN.CENTER)
            add_text(slide, text=lines[1],
                     left=CHART_L + 0.07, top=rt + 0.19,
                     width=LABEL_W - 0.14, height=0.19,
                     size=8, color=AP_WHITE, align=PP_ALIGN.CENTER)

        # Gantt bar
        PAD = 0.06
        bx  = GANTT_L + (phase['start'] - 1) * week_w
        bw  = (phase['end'] - phase['start'] + 1) * week_w
        bh  = PHASE_H - 2 * PAD
        add_rect(slide, left=bx, top=rt + PAD, width=bw, height=bh, fill=bc)

        # Milestone badge at bar right end
        if 'milestone' in phase:
            bs   = 0.20
            bx2  = bx + bw - bs - 0.03
            by2  = rt + PAD + (bh - bs) / 2
            add_rect(slide, left=bx2, top=by2, width=bs, height=bs, fill=BADGE_BG)
            add_text(slide, text=str(phase['milestone']),
                     left=bx2, top=by2 + 0.015,
                     width=bs, height=bs - 0.03,
                     size=8, bold=True, color=AP_WHITE, align=PP_ALIGN.CENTER)

    # ── Milestones section ──────────────────────────────────────────────
    ms_top = ph_top + len(phases) * PHASE_H + 0.22
    add_text(slide, text='ENGAGEMENT MILESTONES',
             left=CHART_L, top=ms_top, width=7.0, height=0.24,
             size=PT_EYEBROW, bold=True, color=AP_DARK_LIME,
             all_caps=True, char_spacing=180)

    item_top = ms_top + 0.30
    gap_v    = 0.48
    half     = (len(milestones) + 1) // 2
    col_w    = (GANTT_R - CHART_L - 0.35) / 2
    col2_x   = CHART_L + col_w + 0.35
    BDG_SZ   = 0.19

    for i, ms in enumerate(milestones):
        col_x = CHART_L if i < half else col2_x
        iy    = item_top + (i if i < half else i - half) * gap_v
        add_rect(slide, left=col_x, top=iy, width=BDG_SZ, height=BDG_SZ, fill=BADGE_BG)
        add_text(slide, text=str(ms['number']),
                 left=col_x, top=iy + 0.01,
                 width=BDG_SZ, height=BDG_SZ - 0.02,
                 size=7, bold=True, color=AP_WHITE, align=PP_ALIGN.CENTER)
        add_text(slide, text=ms['description'],
                 left=col_x + BDG_SZ + 0.10, top=iy - 0.02,
                 width=col_w - BDG_SZ - 0.16, height=gap_v,
                 size=PT_BODY - 1, color=AP_BLACK)

    # ── Legend ──────────────────────────────────────────────────────────
    if legend:
        leg_w = 1.65
        leg_h = 0.26
        leg_x = GANTT_R - leg_w
        leg_y = SLIDE_H - 0.55 - len(legend) * (leg_h + 0.07)
        for k, item in enumerate(legend):
            ly = leg_y + k * (leg_h + 0.07)
            add_rect(slide, left=leg_x, top=ly, width=leg_w, height=leg_h,
                     fill=item['color'])
            add_text(slide, text=item['label'],
                     left=leg_x, top=ly + 0.04,
                     width=leg_w, height=leg_h - 0.08,
                     size=10, bold=True, color=AP_WHITE, align=PP_ALIGN.CENTER)

    add_confidential(slide)
    return slide
```

## Part 3: Automatic Appendix (Library-Based)

Every Authority Partners deck ends with a 13-slide appendix: an "Appendix" divider followed by all 12 slides from the official Authority Partners slide library, in source order. Because users may want to edit appendix content (correct figures, update years, replace logos, change text) and send the deck back, the appendix slides are **fully editable native PowerPoint shapes**, not embedded images.

This is achieved by starting the entire deck from a copy of the library PPTX. Most of the library's 12 slides are then transformed: each slide's source chrome (title bar, footer, page number) is stripped and replaced with our consistent brand chrome (lime eyebrow + mixed-case title at top, logo top-right, CONFIDENTIAL footer bottom-left). Two slides are exceptions and keep their source design verbatim (slide 7 Technology Map and slide 12 AP Lab), because each is a full-bleed graphic with its own complete headline. The main deck slides (cover, agenda, content, closing) are added afterward, then a slide reorder puts them at the front with the appendix divider between.

### How the deck is built

```python
prs = create_ap_presentation()          # starts as a copy of the library, chrome rebranded
add_cover_slide(prs, ...)
add_agenda_slide(prs, ...)
# ... more slides ...
add_closing_slide(prs, ...)
add_appendix_divider(prs)
reorder_deck(prs)                       # main deck first, then divider, then library
prs.save('/mnt/user-data/outputs/...')
```

### Asset files

The skill bundles:

- `library-base.pptx`: the official Authority Partners slide library, 12 slides, ~9.5 MB. Used as the base for every generated deck.
- `logo-color.png`, `logo-white.png`, `logo-black.png`: brand logos (color for light backgrounds, white for dark backgrounds, black for the logo over a busy light photo on library slide 6).

The `appendix-technology-map.png` PNG from earlier versions of this skill is no longer used. The Technology Map slide (library slide 7) is now preserved verbatim from the source library, which has its own full-bleed graphic and headline.

### Code

```python
import os
import shutil
import tempfile

LIBRARY_PATH = os.path.join(SKILL_DIR, 'assets', 'library-base.pptx')
APPENDIX_DIR = os.path.join(SKILL_DIR, 'assets')

# Per-library-slide brand chrome: (eyebrow, title). Title is None when the
# source design has its own prominent headline (slides 7 and 12). Eyebrows are
# short category labels (1-3 words); titles are Title Case headlines that
# describe the slide's content. The pattern is deliberate: eyebrow names the
# section, title makes the claim.
LIBRARY_CHROME = {
    1: ('OUR FIRM', 'Driving Success for 28+ Years'),
    2: ('GLOBAL DELIVERY', 'An Ideal Global Partner'),
    3: ('WHAT WE DO', 'Software Engineering & Technology Strategy'),
    4: ('OUR METHOD', 'A Proven Software Engineering Method'),
    5: ('OUR CLIENTS', '100+ Client-Partners Over 28 Years'),
    6: ('WHY CLIENTS CHOOSE US', 'Better Outcomes & Lower Cost of Engagement'),
    7: (None, None),  # source is a full-bleed Technology Map graphic with its own header
    8: ('THE RIGHT PRODUCT', 'KPIs that Prove the Work'),
    9: ('AI', 'Productivity, Transformed'),
    10: ('AGILE DELIVERY', 'Product-Centric & Outcome-Driven'),
    11: ('GOVERNANCE', 'Governance for a Strategic Partnership'),
    12: (None, None),  # source design has its own AP LAB headline
}

# Library slides whose source design is preserved verbatim. No chrome strip,
# no eyebrow, no brand title, no logo, no CONFIDENTIAL. The source design has
# its own complete chrome (headline, page elements). Use these when the source
# slide is a full-bleed graphic or a designed layout that should not be touched.
LIBRARY_SLIDES_KEEP_SOURCE = {7, 12}

# Per-slide layout tweaks applied after chrome strip and before brand chrome.
# Each entry can include:
#   - 'title_width':    override brand title width (default 11.5)
#   - 'title_color':    override brand title color (default AP_BLACK)
#   - 'body_shift_y':   shift the largest source body shape down by N inches
#                       to clear the brand title
#   - 'right_photo':    resize a right-side full-height photo + its left-edge
#                       lime stripe. Value is (new_left, new_width). The stripe
#                       (an auto-shape sitting directly left of the photo) is
#                       repositioned to new_left - 0.08
#   - 'logo_black':     use the all-black logo instead of the color logo, for
#                       slides where the logo sits over a busy light photo
LIBRARY_LAYOUT_TWEAKS = {
    6: {
        # Source photo runs from L=7.68 W=5.663 and crowds the title. Push the
        # photo right and narrow it so the title and lede have full breathing room.
        'right_photo': (9.08, 4.263),
        # The logo sits top-right over the photo (whiteboard area); the color
        # mark reads poorly there, so use the all-black logo.
        'logo_black': True,
    },
    9: {
        # Source background is a dark gray photo; black title is unreadable.
        'title_color': AP_WHITE,
    },
}


def strip_source_chrome(slide):
    """Remove source title bar and footer shapes from a library slide.
    Top chrome: shapes with top < 1.5 AND height < 0.9 (catches logo, lime
    stripe, title text, subtitle, including slide 6's lower title bar at 0.9
    and slide 9's slightly taller title at h=0.82).
    Bottom chrome: shapes with top > 7.0 (CONFIDENTIAL/PUBLIC, page number,
    horizontal divider)."""
    shapes_to_remove = []
    for shape in slide.shapes:
        top = shape.top / 914400 if shape.top is not None else 0
        height = shape.height / 914400 if shape.height is not None else 0
        if top < 1.5 and height < 0.9:
            shapes_to_remove.append(shape)
        elif top > 7.0:
            shapes_to_remove.append(shape)
    for shape in shapes_to_remove:
        sp = shape.element
        sp.getparent().remove(sp)


def add_brand_chrome_to_library_slide(slide, *, eyebrow, title=None,
                                       title_width=11.5, title_color=AP_BLACK,
                                       logo_black=False):
    """Add our brand chrome (eyebrow + title + logo + CONFIDENTIAL) to a library
    slide after its source chrome has been stripped. title_width, title_color
    and logo_black can be overridden per slide via LIBRARY_LAYOUT_TWEAKS."""
    add_eyebrow(slide, text=eyebrow, left=PAD_X, top=PAD_Y, width=12.0)
    if title:
        add_title(slide, text=title, left=PAD_X, top=PAD_Y + 0.35,
                  width=title_width, height=1.4, color=title_color)
    add_logo(slide, dark=False, black=logo_black)
    add_confidential(slide)


def _apply_pre_chrome_tweaks(slide, tweaks):
    """Apply layout tweaks to the source shapes BEFORE brand chrome is added.
    Handles right_photo resize and body_shift_y. Called after strip_source_chrome
    so the right-photo + lime-stripe pair survives (they are tall shapes the strip
    spares) and any body groups are intact."""
    if not tweaks:
        return
    # Resize a right-side full-height photo and its left-edge lime stripe.
    if 'right_photo' in tweaks:
        new_left, new_width = tweaks['right_photo']
        photo = None
        stripe = None
        for shp in slide.shapes:
            h = shp.height / 914400 if shp.height is not None else 0
            t = shp.top / 914400 if shp.top is not None else 0
            # Full-height (or close to) right-side shape, starting near the top
            if h >= 6.5 and t < 0.5:
                if shp.shape_type == 13:  # PICTURE
                    photo = shp
                elif shp.shape_type == 1:  # AUTO_SHAPE (the lime stripe)
                    stripe = shp
        if photo is not None:
            photo.left = Inches(new_left)
            photo.width = Inches(new_width)
        if stripe is not None:
            stripe.left = Inches(new_left - 0.08)
    # Shift the largest source body shape (typically a group) downward to make
    # room for a wrapping brand title.
    if 'body_shift_y' in tweaks:
        dy = tweaks['body_shift_y']
        largest = None
        largest_area = 0
        for shp in slide.shapes:
            w = shp.width / 914400 if shp.width is not None else 0
            h = shp.height / 914400 if shp.height is not None else 0
            area = w * h
            if area > largest_area:
                largest_area = area
                largest = shp
        if largest is not None and largest.top is not None:
            largest.top = Emu(largest.top + Inches(dy))


def prepare_appendix_from_library(prs):
    """For each library slide in prs: strip source chrome and add brand chrome,
    applying per-slide layout tweaks where defined.
    Special cases:
    - Slides in LIBRARY_SLIDES_KEEP_SOURCE (7 Technology Map, 12 AP Lab): source
      design is kept verbatim. No chrome strip, no brand chrome.
    - Slides with (None, None) in LIBRARY_CHROME but NOT in KEEP_SOURCE: source
      chrome is stripped, then logo + CONFIDENTIAL only are added (no brand
      eyebrow or title)."""
    for i, slide in enumerate(prs.slides):
        lib_num = i + 1
        if lib_num not in LIBRARY_CHROME:
            continue
        if lib_num in LIBRARY_SLIDES_KEEP_SOURCE:
            # Keep source design verbatim. No strip, no chrome.
            continue
        strip_source_chrome(slide)
        tweaks = LIBRARY_LAYOUT_TWEAKS.get(lib_num, {})
        _apply_pre_chrome_tweaks(slide, tweaks)
        eyebrow, title = LIBRARY_CHROME[lib_num]
        if eyebrow:
            add_brand_chrome_to_library_slide(
                slide,
                eyebrow=eyebrow,
                title=title,
                title_width=tweaks.get('title_width', 11.5),
                title_color=tweaks.get('title_color', AP_BLACK),
                logo_black=tweaks.get('logo_black', False),
            )
        else:
            # Source design has its own headline; just add logo + CONFIDENTIAL.
            add_logo(slide, dark=False, black=tweaks.get('logo_black', False))
            add_confidential(slide)


def add_appendix_divider(prs):
    """Section divider that introduces the appendix."""
    slide = blank_slide(prs)
    set_slide_background(slide, AP_WHITE)
    add_logo(slide, dark=False)
    add_eyebrow(slide, text='APPENDIX', left=1.5, top=2.8, color=AP_DARK_LIME)
    add_title(slide, text='For Your Reference', left=1.5, top=3.1,
              width=10.5, height=2.0, size=PT_SECTION_DIV)
    add_lede(slide, text='The pages that follow are reference slides covering Authority Partners global presence, capabilities, method, clients and outcomes.',
             left=1.5, top=5.0, width=9.0)
    add_confidential(slide)
    return slide


def reorder_deck(prs, num_library=12):
    """Reorder slide XML so main deck slides come first, then the appendix
    divider, then the library slides. Called once at the end of deck building."""
    sldIdLst = prs.slides._sldIdLst
    slide_ids = list(sldIdLst)
    library_ids = slide_ids[:num_library]
    after_library = slide_ids[num_library:]
    # The appendix divider is the LAST slide added, so it is last in after_library.
    divider_id = after_library[-1]
    main_deck_ids = after_library[:-1]
    new_order = main_deck_ids + [divider_id] + library_ids
    for sid in slide_ids:
        sldIdLst.remove(sid)
    for sid in new_order:
        sldIdLst.append(sid)


def create_ap_presentation():
    """Create an Authority Partners presentation that starts from a copy of the
    slide library. Returns a Presentation with 12 library slides at the front,
    each wearing the brand chrome (source chrome stripped and replaced).

    The caller adds main deck slides (cover, agenda, content, closing) using the
    usual add_*_slide functions, then calls add_appendix_divider(prs) and finally
    reorder_deck(prs) before saving."""
    tmp_path = tempfile.mktemp(suffix='.pptx')
    shutil.copy(LIBRARY_PATH, tmp_path)
    prs = Presentation(tmp_path)
    prs.slide_width = Inches(SLIDE_W)
    prs.slide_height = Inches(SLIDE_H)
    prepare_appendix_from_library(prs)
    return prs
```

### Editability

All 12 appendix slides are native PowerPoint shapes and text boxes inherited from the source library. The user can edit any text, move any shape, replace any logo, change colors or fonts directly in PowerPoint. Slide 7 (Technology Map) is a single full-slide image in the source library, so the body itself isn't text-editable, but the user can still replace, resize or remove the image directly in PowerPoint. The remaining 11 slides are fully editable end-to-end.

On the 10 slides where the brand chrome is applied (everything except 7 and 12), the chrome (eyebrow, title, logo, CONFIDENTIAL) is also editable since it's added as native text boxes and a picture.

### Opting out

If a specific deck should not include the appendix, do not call `create_ap_presentation()`. Instead, create a fresh `Presentation()` and add main deck slides as usual:

```python
prs = Presentation()
prs.slide_width = Inches(SLIDE_W)
prs.slide_height = Inches(SLIDE_H)
add_cover_slide(prs, ...)
# ... main deck slides ...
add_closing_slide(prs, ...)
# No appendix divider, no reorder.
prs.save(...)
```

The user must say so explicitly ("no appendix this time" or similar) for the appendix to be skipped.
