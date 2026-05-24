# Modifications from upstream rism-digital/verovio

## Base commit

**Upstream:** `rism-digital/verovio@1d164d253fcca8a7c80a8912fd98205057cc0874` (develop branch, post-version-2.7.1)

**Modified branch:** `probe/uniform-width-probe`

## Changes made

### SightEven layout option (uniform measure widths)

Added two new options to Verovio's layout system:

- `uniformMeasureWidth` (boolean) — when true, all measures on a system are forced to the same width
- `uniformMeasureSpacing` (leftAlign | scaled | time) — controls how notes/rests are distributed within the now-wider measures

#### Implementation

**Files modified:**

- `include/vrv/options.h`
  - Added `OptionBool m_uniformMeasureWidth`
  - Added `OptionIntMap m_uniformMeasureSpacing`
  - Added enum `option_UNIFORM_SPACING` with values leftAlign, scaled, time

- `src/options.cpp`
  - Registered `uniformMeasureWidth` and `uniformMeasureSpacing` options
  - Defined static string map for spacing mode values

- `include/vrv/doc.h`
  - Added `PrepareUniformMeasureWidth(Page*)` method declaration
  - Added private members `m_uniformTargetWidth` and `m_uniformBarlineXRel`

- `src/doc.cpp`
  - Implemented `PrepareUniformMeasureWidth(Page*)`
  - Called from `CastOffDocBase` after the cache block (post-line 1082) to redistribute alignments after system cast-off

- `include/vrv/page.h`
  - Added `ApplyUniformMeasureWidth()` public method
  - Added `RedistributeMeasureAlignments(Measure*, int, int, int)` private method

- `src/page.cpp`
  - Implemented `ApplyUniformMeasureWidth()` — computes uniform target width from all measures on a page and calls `RedistributeMeasureAlignments` for each
  - Implemented `RedistributeMeasureAlignments()` — redistributes Alignment x-positions based on the chosen spacing mode:
    - **leftAlign**: natural Verovio spacing, content hugs the left
    - **scaled**: per-measure proportional scaling — preserves Verovio's collision avoidance while stretching to fill uniform width
    - **time**: pure time-proportional placement — each note onset placed at `(note_time / max_measure_time) * usable_width`, respecting natural left padding

#### Purpose

SightPiano's SightEven feature provides time-proportional score layout to help beginners read rhythm spatially. Uniform measure widths combined with time-proportional note positioning make visual distance on the page correspond directly to elapsed musical time, supporting sight-reading pedagogy by making rhythm "visible" on the staff.

#### Diff

To see the exact changes against the upstream base commit:

```bash
git diff 1d164d253fcca8a7c80a8912fd98205057cc0874..probe/uniform-width-probe
```

## License

This modification is released under the same **LGPL-3.0** license as the upstream Verovio project. The original Verovio copyright and license notices are preserved in `COPYING` at the root of this repository.

## Build

To build the WASM module including these modifications:

```bash
cd emscripten
./buildNpmPackage
```

The output in `emscripten/npm/` is the package that SightPiano consumes via `pnpm link:`.

## Integration in SightPiano

SightPiano conditionally emits these options only when the user enables the "SightEven" feature toggle. When SightEven is disabled, both options are omitted and Verovio behaves identically to stock.

- See: `/Users/yasuo/Documents/sightpiano/packages/notation-web/src/verovio-layout.ts:122-125` for the option emission logic
- See: `/Users/yasuo/Documents/sightpiano/website/legal.htm#third-party` for the in-app LGPL notice directing users to this source

## Contact

For questions about these modifications, contact: Original Device LLC — <contact@sightpiano.net>
