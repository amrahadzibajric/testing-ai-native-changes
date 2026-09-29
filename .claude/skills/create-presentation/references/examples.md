# Full Worked Example: Complete Deck Script

Referenced from `../SKILL.md`. A complete script for an 8-slide capabilities deck looks like:

```python
#!/usr/bin/env python3
import os
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR

# [paste brand constants block from code-library.md Part 1]
# [paste helper functions block from code-library.md Part 1]
# [paste pattern functions you need from code-library.md Part 2]

# Resolve logo paths
SKILL_DIR = '/path/to/authority-partners-slides-pptx'  # adjust at runtime
LOGO_COLOR = os.path.join(SKILL_DIR, 'assets', 'logo-color.png')
LOGO_WHITE = os.path.join(SKILL_DIR, 'assets', 'logo-white.png')
LOGO_BLACK = os.path.join(SKILL_DIR, 'assets', 'logo-black.png')

# [paste the appendix-library functions from code-library.md Part 3 — LIBRARY_PATH
#  depends on SKILL_DIR above, so this block comes after logo path resolution]

# Build the deck. create_ap_presentation() must be the first call: it returns a
# presentation pre-loaded with the 12 rebranded library slides, sets slide_width/
# slide_height, and everything added below lands after those 12 slides until
# reorder_deck() puts the main deck first. Use a bare Presentation() instead only
# if the user has explicitly opted out of the appendix (see code-library.md Part 3
# "Opting out").
prs = create_ap_presentation()

add_cover_slide(prs,
    eyebrow='CAPABILITIES OVERVIEW',
    title_main='A Software Engineering Partner',
    title_accent='For Healthcare Payers',
    tagline='A focused look at how Authority Partners delivers regulated, member-facing and core platform software for health plans.',
    meta=[('PREPARED FOR', 'Office of the CIO'), ('DATE', 'May 2026'), ('AUTHORITY PARTNERS', 'authoritypartners.com')])

add_agenda_slide(prs, items=[
    {'title': 'Who We Are', 'description': 'Positioning and the kinds of clients we serve.'},
    {'title': 'Why Payers Choose Us', 'description': 'Where our healthcare experience runs deepest.'},
    {'title': 'Capabilities & Delivery', 'description': 'What we build and how we run engagements.'},
    {'title': 'Outcomes & Next Steps', 'description': 'Results from comparable work and a path forward.'},
])

# ... more slides ...

add_closing_slide(prs,
    title_main="Let's Design the First Slice",
    title_accent='Together',
    tagline="We'd welcome a working session with your engineering and digital leads.",
    meta=[('CONTACT', '[Name, Title]'), ('EMAIL', 'hello@authoritypartners.com'), ('WEB', 'authoritypartners.com')])

# Complete the standard Authority Partners appendix: an appendix divider, then
# reorder so the main deck comes first, the divider next, then the 12 library
# reference slides already at the front of prs (driving success, global map, what
# we do, method, client logos, why clients choose, technology map, right product,
# AI, product-centric agile, governance, AP Lab). Ten wear the brand chrome
# (eyebrow, title, logo, CONFIDENTIAL); two (technology map and AP Lab) keep their
# source design verbatim. All twelve are editable native PowerPoint shapes. Omit
# both calls only if the user explicitly says the deck shouldn't include the
# appendix — and use a bare Presentation() at the top of the script instead.
add_appendix_divider(prs)
reorder_deck(prs)

prs.save('/mnt/user-data/outputs/ap-deck-healthcare-payer-capabilities-20260510.pptx')
print('Deck saved.')
```
