# VISxGenAI 2026 — Flyer Handout

Comprehensive reference for the workshop flyer system. Companion to `visxgenai-workshop.md` (workshop facts) and `flyer/PRINT-NOTES.md` (print pipeline).

_Last updated: 2026-05-08_

---

## 1. The brief

A single-page A4 portrait flyer to advertise the **2nd Workshop on GenAI, Agents, and the Future of VIS** at IEEE VIS 2026 (Nov 9, 2026 · Boston, MA). The workshop has two participation tracks: paper/position presentations and the Agentic VIS Challenge (the live agent challenge that VIS Arena extends). The flyer must surface workshop name + date + location, both tracks, organizers, two QR codes (Website + Discord), and a sponsor placeholder.

We explored eight visual directions in parallel so the team could pick by taste without re-debating content. All eight share identical text content — only typography, palette, and layout vary.

---

## 2. File map

```
flyer/
├── index.html                    Review page — loads all 8 as scaled A4 iframes
├── editorial-tri-tone.html       01 — Editorial Tri-Tone
├── dd-editorial.html             02 — DD Editorial
├── soft-editorial.html           03 — Soft Editorial
├── warm-editorial.html           04 — Warm Editorial
├── bold-poster.html              05 — Bold Poster
├── bold-mono.html                06 — Bold Mono
├── soft-systems.html             08 — Soft Systems
├── future-nostalgia.html         09 — Future Nostalgia
├── qr.png                        Website QR → visxgenai.github.io
├── discord-qr.png                Discord QR → discord.gg/8yeKpBpsCa
├── bg-2026.png                   Painted Boston skyline (used by 09)
├── boston-skyline.svg            (unused alternate line-art skyline)
├── PRINT-NOTES.md                Print pipeline + Pantone codes + bleed instructions
└── archive/                      Discarded variants (peoples-platform, brutalist, neo-grid-bold, grid-mono)
```

The numbering jumps from 06 to 08 to 09 because Bold Mono's variant `grid-mono` was originally numbered 08 then archived. The current 08 (Soft Systems) and 09 (Future Nostalgia) were authored after the renumbering.

---

## 3. Flyer roster

Pairs are grouped by template DNA (font stack + structural rhythm). Each pair has the same skeleton with different palette.

### Pair A — Tri-tone editorial (3 full-bleed color bands)

| # | Name | Background palette | Accent palette |
|---|------|---------------------|-----------------|
| 01 | Editorial Tri-Tone | Burgundy `#7A1F35` / Pink `#F2B6C6` / Cream-yellow `#F2D86A` | Pink + Cream typography |
| 02 | DD Editorial | DD Brown `#653819` / Paper `#FBEDD9` / Cream `#FFF8F0` | DD Orange `#FF671F` + DD Magenta `#DA1884` |

**Type stack:** Instrument Serif (display) · Bricolage Grotesque (body) · JetBrains Mono (eyebrows + meta)
**Layout:** burgundy/brown hero with title + venue + date → middle pink/paper band with section header + 2 tracks side-by-side + deadlines strip → cream footer with `ORGANIZERS` h4 + 4-col organizer grid + QR pair + sponsor placeholder
**Vibe:** magazine spread, intentional, warm

### Pair B — Soft editorial (low-density, single-panel paper, hairline rule between tracks)

| # | Name | Background | Accent |
|---|------|-------------|---------|
| 03 | Soft Editorial | Paper `#F2EEDF` | Sage `#B7C7A8` (underline highlight under "Visualization"), Pink `#E1A4C2`, Blush `#E8C9B6` |
| 04 | Warm Editorial | Off-white `#F8F2EC` | Orange `#DC833A` highlight, Magenta `#C53A87`, Peach `#EAA976` (Track I pill) |

**Type stack:** Cormorant Garamond (display + flourish) · Work Sans (body)
**Layout:** centered serif title with optional underline highlight on a key word → italic venue + date with `✦` flourish below → 2 tracks separated by 1px hairline rule → deadlines strip → centered italic `Organizers` h4 + 4-col organizer grid + QR pair + sponsor placeholder
**Vibe:** Sunday supplement, quiet, literary
**Note:** 03 has no avatars; 04 has dashed-circle avatars (12mm) with peach fill

### Pair C — Bold poster (Shrikhand display, white/cream paper)

| # | Name | Background | Accent |
|---|------|-------------|---------|
| 05 | Bold Poster | White `#FFFFFF` / Light `#F5F2EF` track cards | Fire-engine red `#D8000F` |
| 06 | Bold Mono | Cream `#FFF8F0` / Paper `#FBEDD9` track cards | DD Orange `#FF671F` + DD Magenta `#DA1884` |

**Type stack:** Shrikhand (poster title + section title + numerals) · Libre Baskerville (kicker italic + h3 + organizer names) · Space Grotesk (body + meta)
**Layout:** Shrikhand title with italic "of" + accent-colored "VIS." → italic kicker below dark hairline rule + bold accent-colored date → 2 light track cards with grid layout (numeral + h3 inline) → 4 deadline columns top/bottom rules → centered Shrikhand `Organizers.` + 4-col grid (with avatars) + QR pair
**Vibe:** poster, dramatic, magazine cover energy

### Pair D — Studio (Soft Systems family, gradient + dot-pattern background)

| # | Name | Background | Accent |
|---|------|-------------|---------|
| 08 | Soft Systems | Gradient warm cream + radial pink/orange glows + 1.8mm dot pattern | DD Magenta `#e11383` + DD Orange `#f5821f` + Peach `#f8a45a` |
| 09 | Future Nostalgia | Same gradient + faint Boston skyline (`bg-2026.png` at 38% opacity) in hero | Same DD palette + Brown `#683817` |

**Type stack:**
- 08: Shrikhand (numerals + section title) + Libre Baskerville (h2 + names) + Space Grotesk (body)
- 09: Instrument Serif (display + numerals) + Bricolage Grotesque (body)

**Layout:** title at top → venue/date row with hairline rule above → 2 track cards with corner-pin decoration → border-top-bottom deadlines strip → `Organizers` heading → bottom row: 4-col grid + 56–57mm QR/sponsor panel
**Vibe:** studio brand deck, painterly, design-led
**Note:** 09 uses the painted Boston skyline as faint hero background; 08 uses pure radial-gradient glow only

---

## 4. Common content (identical across all 8)

### Workshop title
> **2nd GenAI, Agents, & the Future of VIS** _(or "Visualization" on Soft / Warm Editorial)_

The "Workshop on" wording is intentionally dropped from the title — the venue line below carries it.

### Venue / date
- `@ IEEE VIS 2026 Workshop`
- `Nov 9, 2026 · Boston, MA`

### Two tracks (text identical across all flyers)

**Track I — Papers & Position Talks**
> Research papers, position pieces, and demos on GenAI, agents, and visualization. Short and long formats — bring early ideas, finished systems, or thoughtful provocations.

**Track II — Agentic VIS Challenge**
> Build, adapt, and submit AI agents into the **VIS Arena** — agents that automatically analyze datasets and generate visual data reports.

The challenge copy intentionally does NOT mention peer-review or rubric pluralism — kept simple and concrete for a recruitment flyer. The richer pitch lives in `notes/vis-arena-notes.md`.

### Deadlines (strip)
- CfP opens: **May 30, 2026**
- Submissions: **Aug 15, 2026**
- Decisions: **Sept 10, 2026**
- Camera-ready: **Oct 1, 2026**

### Organizers (7 total)
| Avatar | Name | Affiliation |
|--------|------|-------------|
| ZC | Zhu-Tian Chen | U. Minnesota–Twin Cities |
| NK | Nam Wook Kim | Boston College |
| SB | Saeed Boorboor | U. Illinois Chicago |
| SR | Shivam Raval | Harvard University |
| PH | Pan Hao | U. Minnesota–Twin Cities |
| QW | Qianwen Wang | U. Minnesota–Twin Cities |
| VS | Vidya Setlur | Tableau Research |

Layout convention: 4-column grid → 4 organizers in row 1, 3 organizers + empty cell in row 2; QR pair (Website + Discord) + sponsor placeholder span the right-most column across both rows.

### QR codes
- **Website QR** (`qr.png`) → `https://visxgenai.github.io/`
- **Discord QR** (`discord-qr.png`) → `https://discord.gg/8yeKpBpsCa`

Both rendered at 25mm × 25mm, 1.5mm white padding, 1px ink-colored border. PNGs generated with the Python `qrcode` lib, default ECC level (~Medium), default 4-module quiet zone built into the PNG.

### Sponsor placeholder
A `13mm × full-width` dashed-border box labelled "sponsor logo" sits above the QR pair on every flyer. To swap in a real logo:
```html
<div class="sp-slot"><img src="sponsor.png" style="max-width:100%;max-height:100%;object-fit:contain;"></div>
```
And remove the dashed border (change `1px dashed → none` or `1px solid`).

---

## 5. Shared structural conventions

### A4 sizing
Every flyer uses:
```css
@page { size: A4; margin: 0; }
.flyer { width: 210mm; height: 297mm; overflow: hidden; }
@media print { body { padding: 0 !important; margin: 0 !important; } }
```
The `@media print` rule strips the screen-only 20px body padding so the 297mm flyer doesn't get pushed onto a 2nd page when printing.

**Bleed:** the canvas sits exactly at trim (210×297mm) with no bleed extension. For pro print, place the exported PDF on a 216×303mm canvas with crop marks (3mm bleed each side) before sending — see `flyer/PRINT-NOTES.md`.

### Organizers section font spec (unified across all 8)
- **Name** — `10.5pt` (9.5pt on 05/06 because Libre Baskerville renders wider), `font-weight` matched to family bold (600 sans / 600 serif / 700 LB), `letter-spacing: -0.01em`, `white-space: nowrap`
- **Affiliation** — `6.5pt`, `font-weight: 400`, `letter-spacing: 0.01em`, `white-space: nowrap`, `margin-top: 0.5mm`
- **Grid** — `repeat(4, minmax(0, 1fr)) 56mm`, `gap: 4mm 5mm`, `align-items: start`
- **Cell** — `min-width: 0` (so `nowrap` content doesn't push column wider), `text-align: center` on flyers with avatars

### Avatar pattern (where used: 01, 04, 05, 09)
Dashed-border circle with 2-letter initials, scoped to each flyer's accent palette:
```css
width: 12mm; height: 12mm; border-radius: 50%;
background: <flyer accent fill>;
border: 1px dashed <flyer ink>;
font-family: <flyer name font>; font-size: 8.5–11pt; font-weight: 600–700;
margin: 0 auto 2mm;
```
To swap to real photos:
```html
<div class="avatar"><img src="photos/zhutian.jpg"
  style="width:100%;height:100%;object-fit:cover;border-radius:50%;"></div>
```
And drop the dashed border (`1px dashed → 1px solid` or remove).

### Print-readiness tweaks already applied
- All hairline borders bumped from `0.5px` → `1px` so they survive cheap print stock.
- Soft Editorial track-divider opacity raised from `0.4` → `0.6` to survive dot gain.
- Body text bumped to weight 400+ (no Work Sans Light 300) for legibility on paper.

---

## 6. Per-flyer distinctive features

### 01 Editorial Tri-Tone
- Three full-bleed color bands; the only flyer with a yellow footer band
- "Two ways to participate" h2 in Instrument Serif inside the pink middle
- Italic Roman numerals (`i.` / `ii.`) at 32pt in burgundy

### 02 DD Editorial
- Same 3-band structure as 01 but with DD palette
- Brown hero with cream title + DD Orange "of VIS" italic accent
- Magenta italic on `VIS Arena`
- Orange Roman numerals + Orange deadline values

### 03 Soft Editorial
- Centered serif title with sage underline behind "Visualization"
- 1px hairline rule separates Track I from Track II
- Centered `Organizers` italic heading; no avatars
- `✦` flourish below date

### 04 Warm Editorial
- Same Soft Editorial skeleton + DD-warm palette
- Orange underline behind "Visualization" instead of sage
- Peach pill on Track I, magenta pill on Track II
- Has dashed-border avatars with peach fill

### 05 Bold Poster
- White background — best for home/office printing (lowest ink coverage)
- Single fire-engine red accent on `VIS.` and Track II border
- Track grid: numeral inline-baseline with title via grid-template-columns: auto 1fr
- Shrikhand `Organizers.` title

### 06 Bold Mono
- Bold Poster's structure recolored with DD palette
- DD Orange replaces red as primary accent; DD Magenta on `VIS Arena`
- Cream paper background; track cards in slightly darker peach paper
- Same inline numeral+title grid layout as 05

### 08 Soft Systems
- Layered radial gradients (pink + orange glows) + 1.8mm dot pattern overlay (~28% opacity)
- Track cards with rotated empty square decoration (-10mm offset, 12° rotation)
- Right-side QR/sponsor panel separated from organizer grid via `bottom { grid: 1fr 55mm }`
- DD Magenta + DD Orange + Peach palette
- Header strip removed (was `08 / VISxGENAI 2026`)

### 09 Future Nostalgia
- Painted Boston skyline (`bg-2026.png`) as hero background at 38% opacity, blend-mode multiply
- Strong cream gradient overlay so title remains legible
- Decorative "L-bracket" corner pin on each track card (`::before` pseudo)
- Hero `padding: 14mm 14mm 7mm` + middle `flex: 0 0 105mm` to avoid dead space
- Same 4-col organizer grid as the editorial family
- Aff font reduced to 6.5pt to fit narrow cells with `white-space: nowrap`

---

## 7. Update operations

### Replace the Discord QR
```bash
python3 -c "import qrcode; qrcode.make('NEW_DISCORD_URL').save('flyer/discord-qr.png')"
```
All 8 flyers reference the same file — they all update.

### Replace the Website QR
```bash
python3 -c "import qrcode; qrcode.make('https://visxgenai.github.io/').save('flyer/qr.png')"
```

### Drop in real organizer photos
1. Save each photo as `flyer/photos/<lastname>.jpg` (or `.png`)
2. In flyers 01, 04, 05, 09: replace each `<div class="avatar">XX</div>` with:
   ```html
   <div class="avatar"><img src="photos/<lastname>.jpg" style="width:100%;height:100%;object-fit:cover;border-radius:50%;"></div>
   ```
3. Change the avatar's CSS `border: 1px dashed → 1px solid` (or remove)

### Drop in the sponsor logo
Replace `<div class="sp-slot">sponsor logo</div>` with:
```html
<div class="sp-slot"><img src="sponsor.png" style="max-width:100%;max-height:100%;object-fit:contain;"></div>
```
And drop the dashed border in CSS.

### Export to PDF for the printer
Open each flyer in Chrome → Cmd-P → "Save as PDF" → 100% scale, no headers/footers. Or batch via headless Chrome:
```bash
CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
cd flyer
for f in editorial-tri-tone dd-editorial soft-editorial warm-editorial bold-poster bold-mono soft-systems future-nostalgia; do
  "$CHROME" --headless --disable-gpu --no-sandbox \
    --print-to-pdf="$f.pdf" --no-pdf-header-footer \
    --virtual-time-budget=6000 "file://$PWD/$f.html"
done
```
For pro print: import the PDF into a vector editor and place on a 216×303mm canvas with crop marks (3mm bleed). See `flyer/PRINT-NOTES.md` for Pantone equivalents.

---

## 8. Review workflow

`flyer/index.html` loads all 8 flyers as scaled A4 iframes in a 2-column grid. Pairs sit side-by-side:

| Row | Left | Right |
|-----|------|-------|
| 1 | 01 Editorial Tri-Tone | 02 DD Editorial |
| 2 | 03 Soft Editorial | 04 Warm Editorial |
| 3 | 05 Bold Poster | 06 Bold Mono |
| 4 | 08 Soft Systems | 09 Future Nostalgia |

Toggle 1/2/4-column density via the buttons in the header. Click "open ↗" on any card to view the flyer at full A4 size in a new tab.

Card head shows numbered title (`01 Editorial Tri-Tone`) — number is bold (700) muted-grey, theme name is regular (400). Removed the palette/font tag line that used to sit below.

---

## 9. Decisions log (key iterations)

- **Initial set:** 4 directions (Editorial Tri-Tone, People's Platform, Soft Editorial, Brutalist).
- **Added:** Bold Poster + Neo-Grid Bold, then Bold Mono + Grid Mono (DD palette variants).
- **Discarded** (now in `flyer/archive/`): People's Platform, Brutalist, Neo-Grid Bold, Grid Mono — too loud / too technical / too neon for the workshop voice.
- **Title rewrite:** Originally "2nd Workshop on GenAI...". Dropped "Workshop on" since the venue line below already says "@ IEEE VIS 2026 Workshop". Title now leads with "2nd GenAI, Agents, & the Future of VIS."
- **Track-2 copy rewrite:** Removed the rubric-pluralism / peer-review wording. Final copy is recruitment-friendly: "Build, adapt, and submit AI agents into the VIS Arena — agents that automatically analyze datasets and generate visual data reports."
- **No header strips:** Top eyebrow stamps (`VISxGenAI · No. 02`, `// VISxGenAI [02]`, etc.) removed across all flyers — the title carries the workshop identity.
- **No contact footer:** `visxgenai@ieeevis.org · visxgenai.github.io` line removed across all flyers — contact lives in the QR codes.
- **Numbering jump 06 → 08 → 09:** original 08 (Grid Mono) was archived after we picked the DD-palette direction; user-authored Soft Systems took 08, Future Nostalgia took 09. Renumbering would touch too many files for cosmetic gain.
- **Organizers section unification:** all 8 flyers now share the same name/aff font sizes and `nowrap` rules so the organizer grid reads consistently across the set.
- **Spacing tightening:** trimmed dead space between deadlines and organizers across the editorial pairs (`.middle { flex: 1 → 0 0 96mm }` on 01/02, `.org-section { margin-top: auto → 8mm }` on 03/04). Bumped breathing room between deadlines and organizers on the bold pairs (`9mm → 14mm`) and Soft Systems (`10mm → 14mm`).

---

## 10. Open items

- [ ] Real sponsor logo file → drop into `flyer/sponsor.png` and swap into all 8.
- [ ] Real organizer photos → `flyer/photos/<lastname>.jpg` × 7 → swap into 01, 04, 05, 09.
- [ ] Pick the final 1–2 directions from the 8 candidates.
- [ ] Pro print: prepare 216×303mm bleed-extended PDFs with crop marks.
- [ ] Test-scan both QR codes with two phones before sending to print.
- [ ] Decide whether to add Pantone spot-color overrides for DD palette (PMS 165 C / 219 C / 1545 C) on flyers 02, 04, 06, 08, 09.
