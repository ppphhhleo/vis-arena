# Flyer Print Notes

Quick reference for sending VISxGenAI 2026 flyers to print.

## Trim & bleed

All four flyer HTML files are sized at **A4 trim (210 × 297 mm)** with `@page { size: A4; margin: 0 }`. They were designed so all critical content (text, QRs, sponsor slot, organizers) sits inside a **safe zone of 14 mm from each edge** (12–13 mm on Bold Mono and Bold Poster).

**For pro print with bleed:**
- Export to PDF, open in a vector editor (Illustrator / Affinity), and place on a 216 × 303 mm canvas with 3 mm bleed on each side.
- Extend any full-bleed color blocks (the burgundy / pink / cream bands in Editorial Tri-Tone, the cream wash in Soft Editorial / Bold Mono) past the 210 × 297 mm trim line by 3 mm so trim drift doesn't expose paper.
- Add crop marks.

**For home / office print:**
- Print at 100 % scale, "actual size", no shrink-to-fit.
- Bold Poster is the safest pick — white background, low ink, no bleed.
- Editorial Tri-Tone is the riskiest — three full-bleed color bands will use a lot of toner and may warp inkjet paper. Take it to a copy shop instead.

## Color management — RGB → CMYK

The flyers are authored in sRGB. CMYK conversion will shift several colors. None of these are deal-breakers, but know what to expect.

| Flyer | Color | Hex | CMYK behavior | Pantone equivalent |
|---|---|---|---|---|
| 03 Bold Poster | Red | `#D8000F` | Drops slightly to muddy orange-red. | **PMS 185 C** |
| 04 Bold Mono | DD Orange | `#FF671F` | Loses noticeable saturation — pure RGB orange has no full-strength CMYK match. | **PMS 165 C** (Dunkin's actual brand) |
| 04 Bold Mono | DD Magenta | `#DA1884` | Loses some punch but stays recognizably magenta. | **PMS 219 C** (Dunkin's actual brand) |
| 04 Bold Mono | DD Brown | `#653819` | Reproduces faithfully. | PMS 1545 C |

If brand color accuracy matters (especially for the DD palette in Bold Mono), ask the print shop to use Pantone spot inks for those swatches instead of CMYK process build.

The pastels in Editorial Tri-Tone (burgundy / pink / cream) and Soft Editorial (paper / sage / blush / lemon / pink) all sit comfortably in the CMYK gamut and will reproduce well as process color.

## Recommended printer choice per flyer

| Flyer | Home inkjet | Office laser | Copy shop / pro |
|---|---|---|---|
| 01 Editorial Tri-Tone | Avoid (paper warp) | OK (toner streak risk) | **Best** |
| 02 Soft Editorial | OK | OK | Best |
| 03 Bold Poster | **Best** — white bg | Best | Best |
| 04 Bold Mono | OK | OK | **Best** (for Pantone match) |

## QR codes

Both QRs (`qr.png` Website + `discord-qr.png` Discord) are placed at **25 × 25 mm** with built-in white quiet zone. This is at the lower edge of reliable scan size — if you have room to bump them to 28–30 mm, do it. Don't shrink below 25 mm.

Test-scan both with two different phones before sending to print.

## Other notes

- **Affiliations under organizer names are 9–9.5 pt.** Readable held in hand or up close on a poster board, but not from across a hallway. If the flyer will be wall-posted at >2 m, consider dropping affiliations and just listing organizer names.
- **The sponsor slot is currently a placeholder** (`<div class="sp-slot">sponsor logo</div>`). Drop in the real logo before print:
  ```html
  <div class="sp-slot"><img src="sponsor.png" style="max-width:100%;max-height:100%;object-fit:contain;"></div>
  ```
- **Discord QR currently encodes** `https://discord.gg/visxgenai` as a placeholder. Regenerate `discord-qr.png` once the real invite URL exists:
  ```bash
  python3 -c "import qrcode; qrcode.make('YOUR_REAL_DISCORD_URL').save('flyer/discord-qr.png')"
  ```

## Recent fixes applied (2026-05-06)

- Bumped 0.5 px hairline borders to 1 px on Editorial Tri-Tone footer divider and across Soft Editorial (masthead, deadlines, org-section, QR, sponsor slot) — anything below ~0.75 pt risks disappearing on cheaper print stock.
- Raised the Track I / Track II divider rule on Soft Editorial from 0.4 to 0.6 opacity so the line stays visible after dot gain.
