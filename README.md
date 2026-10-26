[README.md](https://github.com/user-attachments/files/33986494/README.md)
# Plug Volume Review

A single-page tool for reviewing abandonment cement plugs: pumped volume against theoretical fill, theoretical top of cement against the actual tag, and the permit's requirements. It runs entirely in the browser. Nothing you enter leaves your computer.

## What it does

Built for the witnessing engineer reviewing **one plug at a time**. **Example (04-029-16337)** loads the E & B Natural Resources Section 9 4-6 plug used to design it.

- **Well strip.** Two compact rows hold the API, permit number, operator, HPZ, date, lease, well number, field, county and Sec-T-R. The API accepts any common form (`04-029-16337`, `0402916337`, `0402916337-00`, `02916337`). Status, district, operator code, spud, directional flag, raw HPZ value and coordinates appear on one line underneath. Every field stays editable.
- **Operator dropdown.** Unique operator names from the well list; case and punctuation variants are collapsed to the most common spelling.
- **HPZ** comes from the well list's `inHPZ` field:

  | inHPZ | HPZ box |
  |---|---|
  | Verified HPZ / Potentially HPZ | Yes |
  | Not Within HPZ / Potentially Not Within HPZ | No |
  | Uncertainty Area | Uncertain |
- **Permit drop** (text PDFs, not scans).
  - The **permit number** goes into the Permit No. box; blank header fields fill from the permit's first paragraph.
  - The app pulls out:
    - critical-well language and H₂S;
    - thermal (silica flour) cement;
    - required cement intervals, the surface plug and the clean-out depth;
    - zone tops;
    - witness points, flagged *shall* or *may*.
  - **Mud vs cement all the way.** If the permit fills the hole between plugs with inert mud, the app shows the mud weight and gel strength, and the gaps between required cement intervals ("Mud spacer 35'–2,143'"). If the required cement leaves no gaps, it shows *Cement all the way*. The risk matrix will use this.
  - The plug's **Permit TOC** is set automatically to the top of the permit interval that contains the plug bottom.
- **Squeeze box.** Lists any squeeze the permit requires. If there is none, it reads *No squeeze required in permit; cement behind pipe = satisfactory (TB verified).* It stops updating once you edit it.
- **Plug.**
  - Enter the plug tag type, placement method (coiled tubing, tubing, drill pipe, dump bailer, bullhead/squeeze), zone, bottom, permit TOC, tagged TOC and ft³ pumped (bbl shown).
  - Also record what it was tagged with, weight on tag and the witness.
- **Pipe across the plug.** Enter one row per pipe section, bottom section first, using calculation depths, so a crossover (e.g. 5½" 20# below 7" 23#) is handled. The plug bottom, tag and section tops chain into the next row automatically.
  - ID = √(OD² − W / 2.667)
  - ft³/ft = ID² / 1029.4 × 5.615

  A toggle switches to the CWC Technical Data Book IDs. Operators usually calculate with table IDs, so the other result is always shown as well.
- **Results.**
  - Volume required from the bottom to the tagged TOC, ft³ pumped, **% of required** (deficient or over), and the theoretical TOC from the volume pumped.
  - A memo-style calc block: Permit requires / TOC tagged / Csg. … (Calculation Depths) / Theoretical Volume Required / Actual Volume Pumped.
  - Findings: tag vs permit top, the volume needed to reach the permit top, and a tag higher than the volume supports (mud-cut top, bridge, under-reported volume). A tag lower than theoretical points to losses, a strung-out plug or channeling.
- **Wellbore diagram** to depth scale, with Plug and Whole-well views.
  - Shows pipe sections and crossovers, cement for the volume pumped, and a hatch where the tag is above the theoretical TOC.
  - Draws permit TOC, theoretical TOC and zone-top lines.
  - Shows mud spacer fill when the permit calls for mud.
- **Notes, Copy summary, Export/Import JSON, Print / PDF.**

## Assessment (next step)

The Assessment panel is a placeholder for the risk matrix. It will set allowable % deficient or over by plug type, HPZ, critical well, H₂S, and mud spacer vs cement all the way, then give a pass/fail recommendation. Later phases cover full memos and WellSTAR JSON export.

## Well list data

The well list is `data/wells_20261022.tsv.gz`, about 5 MB, well under GitHub's 25 MB web-upload limit. It is the CalGEM AllWells export (59 MB) cut down to the 20 columns the app uses, tab-separated and gzip-compressed. `data/manifest.json` names the file:

```json
{ "file": "wells_20261022.tsv.gz", "asOf": "2026-10-22" }
```

The first visit downloads and indexes it (a few seconds) and caches it in the browser. Later visits are instant. When the file named in the manifest changes, the app downloads it again.

**To update the well list (no Python needed):**
1. Open the app, click **Load CSV**, and pick the new AllWells CSV export.
2. Click **Save slim file for GitHub**. This downloads `wells_YYYYMMDD.tsv.gz`.
3. Upload that file to `data/` in the repository, change `file` and `asOf` in `data/manifest.json`, and delete the old `.gz`.

Alternatively, run `python tools/make_wells_gz.py AllWells_YYYYMMDD.csv`.

Columns kept: `API, LeaseName, WellNumber, WellStatus, WellTypeLa, OperatorCo, OperatorNa, FieldName, AreaName, CountyName, District, Section, Township, Range, BaseMeridi, Latitude, Longitude, SpudDate, isDirectio, inHPZ`.

**Offline or local use:** open `index.html` directly and click **Load CSV**. It takes either the full CSV export or the slim `.gz`.

## Hosting on GitHub Pages

1. Push this folder to a new repository, for example `plug-volume-review`.
2. In the repository, go to **Settings → Pages → Build and deployment**, set **Source: Deploy from a branch**, choose `main` and `/ (root)`, and save.
3. The site appears at `https://<your-account>.github.io/plug-volume-review/` within a minute or two.

`.nojekyll` is included so Pages serves the files as they are.

## Sources

- Casing OD/weight/ID: Cudd Well Control *Technical Data Book*, pp. 47–48.
- ID and capacity formulas: `Calculate_ID.xlsx` (Central District).
- Well list: CalGEM AllWells export, 2026-10-22.
- PDF text extraction: pdf.js 3.11.174 (cdnjs).
