# Ramp derivation

The v2 palette is one ramp read forwards in light and backwards in dark, so
every value in it has to earn its position twice. This file is the arithmetic
behind those positions: the invariant, the inputs, the derived numbers, and the
error left over.

Everything here was computed with the OKLab → linear-sRGB matrix from CSS Color
4, clipped to sRGB, and WCAG relative luminance `Y = 0.2126R + 0.7152G +
0.0722B`. Nothing assumes `Y = L³`; that identity holds only for a truly
achromatic colour and the ramps are not achromatic.

## The invariant

A reversible ramp works when step `n` read against the light ground carries the
same contrast as step `16 − n` read against the dark ground. The grounds are the
ramp's own ends, because `surface.bg.base` resolves to step 1 in light and
step 15 in dark.

Writing `yₙ = Yₙ + 0.05`, light-mode contrast at step `n` is `y₁ / yₙ` and
dark-mode contrast at step `16 − n` is `y₍₁₆₋ₙ₎ / y₁₅`. Setting them equal and
rearranging:

```
yₙ · y₍₁₆₋ₙ₎ = y₁ · y₁₅          for all n in 1…15,  with L₁ > L₂ > … > L₁₅
```

Step 8 is self-paired, so `y₈ = √(y₁ · y₁₅)`. Tolerance is ±2% on the product,
which holds mirrored contrast pairs within 0.1 of each other.

The schema derives this against cybr's `neutral` ramp. Everything below derives
it against Mirror's own values.

## Input: Mirror v1 `grey.solid`, twelve steps

Mirror's greys, not cybr's. Flip rule for a 12-step ramp is `n′ = 13 − n`.

|   n |      L |     C |       Y | light `y₁/yₙ` | 13−n | dark `y₍₁₃₋ₙ₎/y₁₂` |  product | deviation |
| --: | -----: | ----: | ------: | ------------: | ---: | -----------------: | -------: | --------: |
|   1 | 0.9600 | 0.003 | 0.88474 |          1.00 |   12 |               1.00 | 0.054215 |     +0.0% |
|   2 | 0.9200 | 0.005 | 0.77869 |          1.13 |   11 |               1.07 | 0.051517 |     -5.0% |
|   3 | 0.8800 | 0.005 | 0.68147 |          1.28 |   10 |               1.13 | 0.048003 |    -11.5% |
|   4 | 0.8400 | 0.008 | 0.59269 |          1.45 |    9 |               1.97 | 0.073260 |    +35.1% |
|   5 | 0.7600 | 0.008 | 0.43897 |          1.91 |    8 |               2.33 | 0.066095 |    +21.9% |
|   6 | 0.6800 | 0.008 | 0.31442 |          2.56 |    7 |               2.77 | 0.058519 |     +7.9% |
|   7 | 0.4800 | 0.010 | 0.11058 |          5.82 |    6 |               6.28 | 0.058519 |     +7.9% |
|   8 | 0.4400 | 0.010 | 0.08517 |          6.92 |    5 |               8.43 | 0.066095 |    +21.9% |
|   9 | 0.4000 | 0.010 | 0.06399 |          8.20 |    4 |              11.08 | 0.073260 |    +35.1% |
|  10 | 0.2500 | 0.002 | 0.01562 |         14.24 |    3 |              12.61 | 0.048003 |    -11.5% |
|  11 | 0.2300 | 0.002 | 0.01217 |         15.04 |    2 |              14.29 | 0.051517 |     -5.0% |
|  12 | 0.2000 | 0.002 | 0.00800 |         16.12 |    1 |              16.12 | 0.054215 |     +0.0% |

Target `y₁ · y₁₂ = 0.054215`. The ramp fails by **+35% at the 4/9 pair** and
never gets closer than 5% away from the ends.

Note the sign. The schema's table of cybr's 15-step ramp shows the middle
running _short_ on contrast in dark mode — down to 0.34 of the light-mode
figure. Mirror's 12-step ramp fails the other way: the flip lands steps 4, 5
and 6 on 9, 8 and 7, which sit far below the arithmetic middle, so dark mode is
_over_-contrasty exactly where light mode is soft. Both are the same defect —
the ramp spends its top half in the light region and then jumps — but on a
12-step ramp the cliff between step 9 (L 0.40) and step 10 (L 0.25) is crossed
by the mirror rather than straddled by it.

The two endpoints are worth stating plainly, because they are what the whole
derivation is anchored to: Mirror's `grey.solid.1` is `L 0.96` and its
`grey.solid.12` is `L 0.20`. Those are the same two values cybr anchors its
`neutral` ramp to, which is why the derived numbers below land where the schema
said they would.

## Chroma is symmetric before L is derived

Authoring rule 3 wants `Cₙ = C₍₁₆₋ₙ₎`, so that a light-mode tint does not
become a dark-mode saturated fill. Chroma also moves luminance, so it has to be
fixed before `L` is solved for.

For `grey` and `stone` the envelope is Mirror's own light half, mirrored:

```
0.003 0.005 0.005 0.008 0.008 0.008 0.010 0.010 0.010 0.008 0.008 0.008 0.005 0.005 0.003
```

`neutral` stays at `C 0`. The four chromatic hues take
`Cₙ = k · min(Cmax(Lₙ), Cmax(L₍₁₆₋ₙ₎))` with `k = 0.85` against the Display P3
hull, which is the schema's formula and its constant.

## Derived: `grey.solid`, fifteen steps

Steps 1–6 are Mirror's, unchanged. Step 7 is seeded at `L 0.62` to remove the
cliff. Step 8 is forced to the geometric mean. Steps 9–15 are solved from the
invariant — `yₙ = (y₁ · y₁₅) / y₍₁₆₋ₙ₎` — numerically, against each step's own
chroma.

|   n |      L |     C | Δ to next | light |  dark |  product | deviation |
| --: | -----: | ----: | --------: | ----: | ----: | -------: | --------: |
|   1 |   0.96 | 0.003 |    0.0400 |  1.00 |  1.00 | 0.054214 |   +0.000% |
|   2 |   0.92 | 0.005 |    0.0400 |  1.13 |  1.13 | 0.054211 |   -0.006% |
|   3 |   0.88 | 0.005 |    0.0400 |  1.28 |  1.28 | 0.054210 |   -0.008% |
|   4 |   0.84 | 0.008 |    0.0800 |  1.45 |  1.45 | 0.054214 |   -0.001% |
|   5 |   0.76 | 0.008 |    0.0800 |  1.91 |  1.91 | 0.054215 |   +0.002% |
|   6 |   0.68 | 0.008 |    0.0600 |  2.56 |  2.57 | 0.054225 |   +0.019% |
|   7 |   0.62 |  0.01 |    0.0524 |  3.24 |  3.24 | 0.054207 |   -0.014% |
|   8 | 0.5676 |  0.01 |    0.0508 |  4.01 |  4.01 | 0.054219 |   +0.008% |
|   9 | 0.5168 |  0.01 |    0.0545 |  4.97 |  4.97 | 0.054207 |   -0.014% |
|  10 | 0.4623 | 0.008 |    0.0689 |  6.28 |  6.28 | 0.054225 |   +0.019% |
|  11 | 0.3934 | 0.008 |    0.0683 |  8.43 |  8.43 | 0.054215 |   +0.002% |
|  12 | 0.3251 | 0.008 |    0.0362 | 11.08 | 11.08 | 0.054214 |   -0.001% |
|  13 | 0.2889 | 0.005 |    0.0400 | 12.61 | 12.61 | 0.054210 |   -0.008% |
|  14 | 0.2489 | 0.005 |    0.0489 | 14.29 | 14.29 | 0.054211 |   -0.006% |
|  15 |    0.2 | 0.003 |         — | 16.12 | 16.12 | 0.054214 |   +0.000% |

Maximum deviation on the product: **0.019%**. Maximum mirrored-pair contrast
delta: **0.0001**. Both are two orders of magnitude inside the ±2% tolerance.

WCAG landmarks, and they hold in both modes: step 7 crosses 3:1, step 9 crosses
4.5:1, step 11 crosses 7:1.

**These are the schema's Option A numbers to four decimal places.** That is a
result, not a coincidence, and it is worth being explicit about why. Mirror's
`grey` ramp and cybr's `neutral` ramp share both anchors — `L 0.96` and
`L 0.20` — and Mirror's grey carries at most `C 0.010`, which moves relative
luminance by less than `2 × 10⁻⁶`. The invariant is anchor-sensitive and
chroma-insensitive at that magnitude, so re-deriving against Mirror's greys
reproduces cybr's derivation. Step 10 differs by 0.0001 from the schema's
0.4622, which is rounding.

The interesting re-tune is not here. It is in the four chromatic hues, where
chroma moves luminance by tens of percent.

## The chromatic hues do not share one L ramp

Authoring rule 1 says every hue uses the same fifteen `L` values, and that this
is what makes the invariant hold for every hue at once. That reasoning depends
on `Y = L³`, which is only true at `C = 0`.

Put the four chromatic hues on grey's L ramp with the symmetric envelope above
and measure the worst pair product:

| hue      | worst \|deviation\| on the pair product |
| -------- | --------------------------------------: |
| `blue`   |                                    9.0% |
| `red`    |                                   17.0% |
| `green`  |                                   23.7% |
| `yellow` |                                    0.8% |

Yellow squeaks through. Blue, red and green do not, and green misses by twelve
times the tolerance. A shared L ramp reintroduces, per hue, most of the error
the exercise exists to remove.

### The decision

**Mirror shares the contrast ladder, not the L ramp.**

The ladder is grey's, from the table above — `1.00 1.13 1.28 1.45 1.91 2.56
3.24 4.01 4.97 6.28 8.43 11.08 12.61 14.29 16.12` — and each hue's `Lₙ` is
solved so that its own step `n` sits on that rung. Chroma and L are solved
together to a fixed point, since the envelope depends on L and L depends on the
envelope.

The justification is that the shared L ramp was never the requirement. It is a
proxy for the requirement, and it is exact only where chroma is zero. What the
schema actually wants from rule 1 is that one verification stands for seven
ramps and that step `n` means the same thing in every hue — and holding the
ladder delivers both, where holding L delivers neither for a saturated hue.

Three things follow, and all three are worth having.

**Cross-hue composites become predictable.** Step 7 on `surface.bg.base` reads
3.24:1 whichever hue it is drawn from. Under a shared L ramp the same step
reads 3.24:1 in grey, 3.34:1 in blue, 3.51:1 in red and 3.02:1 in green — and
green, the one that has to clear 3:1 for a non-text UI element, is the one that
comes closest to missing. Nothing in the token names would say so.

**The ramps still look like one family.** Holding the ladder rather than L
moves the L values by at most 0.025, and for blue and yellow by less than 0.01.
The result reads as one ramp with per-hue correction, not as seven independent
ramps.

**It is not cybr's shape arriving by the back door.** cybr's hues carry
their own L ramps because each was hand-fitted to its own gamut hull — `lime`
has three identical top steps and drops 0.40 in L between steps 12 and 13.
These diverge only by the amount the hue's luminance demands, and they diverge
smoothly.

## The derived palette

Hue angles are Mirror's: `blue` 264, `red` 33.84, `green` 151, `yellow` 90.
`grey` and `stone` hold the shared achromatic L ramp — rule 1 survives for the
three near-neutral hues, where it is exact.

### L

| hue       |     H | 1 … 15                                                                                                   |
| --------- | ----: | -------------------------------------------------------------------------------------------------------- |
| `grey`    |   264 | 0.96 0.92 0.88 0.84 0.76 0.68 0.62 0.5676 0.5168 0.4623 0.3934 0.3251 0.2889 0.2489 0.2                  |
| `neutral` |     0 | 0.96 0.92 0.88 0.84 0.76 0.68 0.62 0.5676 0.5168 0.4623 0.3934 0.3251 0.2889 0.2489 0.2                  |
| `stone`   |   106 | 0.96 0.92 0.88 0.84 0.76 0.68 0.62 0.5676 0.5168 0.4623 0.3934 0.3251 0.2889 0.2489 0.2                  |
| `blue`    |   264 | 0.96 0.9201 0.8804 0.8407 0.762 0.6843 0.627 0.578 0.5254 0.4689 0.3975 0.3271 0.2901 0.2495 0.2002      |
| `red`     | 33.84 | 0.9633 0.9259 0.8895 0.8531 0.7813 0.6982 0.6407 0.5909 0.5379 0.4813 0.4095 0.3349 0.296 0.2533 0.2022  |
| `green`   |   151 | 0.953 0.9113 0.87 0.8288 0.7465 0.6644 0.6028 0.5428 0.4944 0.4421 0.3762 0.3109 0.2763 0.238 0.1913     |
| `yellow`  |    90 | 0.9597 0.9197 0.8797 0.8397 0.7599 0.6802 0.6205 0.5686 0.5177 0.4631 0.3941 0.3256 0.2894 0.2493 0.2003 |

### C

| hue       | 1 … 15                                                                                    |
| --------- | ----------------------------------------------------------------------------------------- |
| `grey`    | 0.003 0.005 0.005 0.008 0.008 0.008 0.01 0.01 0.01 0.008 0.008 0.008 0.005 0.005 0.003    |
| `neutral` | 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0                                                             |
| `stone`   | 0.003 0.005 0.005 0.008 0.008 0.008 0.01 0.01 0.01 0.008 0.008 0.008 0.005 0.005 0.003    |
| `blue`    | 0.018 0.036 0.054 0.073 0.112 0.152 0.183 0.21 0.183 0.152 0.112 0.073 0.054 0.036 0.018  |
| `red`     | 0.021 0.042 0.065 0.089 0.141 0.166 0.185 0.204 0.185 0.166 0.141 0.089 0.065 0.042 0.021 |
| `green`   | 0.061 0.076 0.088 0.099 0.12 0.141 0.157 0.173 0.157 0.141 0.12 0.099 0.088 0.076 0.061   |
| `yellow`  | 0.04 0.05 0.058 0.065 0.079 0.093 0.104 0.114 0.104 0.093 0.079 0.065 0.058 0.05 0.04     |

## Residual error

Deviation of `yₙ · y₍₁₆₋ₙ₎` from the target `y₁ · y₁₅ = 0.054214`, per mirrored
pair, per hue. Tolerance is ±2%.

| pair   |    grey | neutral |   stone |    blue |     red |   green |  yellow |
| ------ | ------: | ------: | ------: | ------: | ------: | ------: | ------: |
| 1 / 15 | +0.000% | +0.001% | +0.071% | -0.005% | +0.018% | +0.018% | -0.002% |
| 2 / 14 | -0.006% | -0.004% | +0.132% | -0.025% | -0.030% | +0.002% | -0.004% |
| 3 / 13 | -0.008% | -0.006% | +0.145% | -0.009% | +0.010% | +0.002% | -0.003% |
| 4 / 12 | -0.001% | +0.007% | +0.264% | -0.022% | -0.020% | +0.003% | -0.026% |
| 5 / 11 | +0.002% | +0.010% | +0.292% | +0.011% | -0.006% | -0.003% | +0.015% |
| 6 / 10 | +0.019% | +0.026% | +0.324% | +0.057% | +0.045% | +0.032% | +0.021% |
| 7 / 9  | -0.014% | -0.001% | +0.375% | -0.001% | -0.014% | -0.025% | -0.026% |
| 8 / 8  | +0.008% | +0.020% | +0.399% | +0.037% | -0.007% | -0.018% | +0.034% |

Worst case across the whole palette is **+0.399%**, on `stone` at the 8/8 pair.
`stone` is the only ramp that carries a residual worth naming, and it is there
by choice: it holds grey's L ramp verbatim, and at hue 106 that costs a little
luminance. Solving stone's own L would take it to 0.02%, at the price of
breaking the achromatic three apart for no visible gain.

Every other hue lands inside ±0.06%. The mirrored-pair contrast delta never
exceeds 0.002.

Verified on the built output rather than on the arithmetic: `application.css`
against `theme/dark/application.css`, foreground on `surface.bg.base`.

| token                        | light |  dark |
| ---------------------------- | ----: | ----: |
| `surface.fg.default`         | 16.12 | 16.12 |
| `surface.fg.link.default`    |  3.24 |  3.24 |
| `secondary.fg.solid.default` |  3.24 |  3.24 |
| `accent.bg.solid.default`    |  3.24 |  3.24 |
| `danger.fg.solid.default`    |  3.24 |  3.24 |
| `success.fg.solid.default`   |  3.24 |  3.24 |
| `warning.fg.solid.default`   |  3.24 |  3.24 |
| `primary.bg.solid.default`   | 16.12 | 16.12 |

## What the ladder costs

Holding the ladder is not free, and one group pays most of it.

**`warning` loses its brightness.** v1 puts `warning.bg.solid.default` at
`yellow.solid.7` = `oklch(0.87 0.175 90)`, a bright yellow. On the ladder,
step 7 is whatever reads 3.24:1, and for yellow that is `oklch(0.6205 0.104
90)` — a mustard. There is no way around this: brightness and contrast are the
same axis. It is also the fix for a real defect, because v1 authors
`warning.fg.solid.default` at that same bright yellow, which reads **1.33:1**
against `surface.bg.base`. A foreground token at 1.36:1 is not legible in either
mode; the ladder makes it 3.24:1 in both.

**Chroma comes down.** The symmetric envelope is bounded by the _darker_ half
of each pair, where the hull is narrow. `blue` peaks at `C 0.210` against v1's
0.26; `yellow` peaks at `C 0.114` against v1's 0.175. The schema predicts this
and calls the envelope the only fix on the table.

**Two hues leave sRGB.** `red` steps 1–5 and `green` steps 8–15 sit inside
Display P3 but outside sRGB, and browsers gamut-map them on sRGB displays.
Mapping reduces chroma while holding L and H, which perturbs luminance
slightly, so the invariant is exact on a P3 display and approximate on an sRGB
one. Authoring the envelope against the sRGB hull instead would remove that at
the cost of a further chroma drop — `green` from 0.173 to 0.130 at its peak.
Not taken; flagged.

## The alpha ladder

Solid ramps reverse, translucent ramps do not, so a translucent ramp needs an
ink that reads in both modes.

**The ladder is Mirror's, unchanged:** twelve steps, `0.12` to `0.78` in steps
of `0.06`. cybr's irregular `0.04 … 0.96` is not adopted. On an even ladder the
three overlay rungs keep cybr's relationship exactly, because the relationship
is an index offset: `bg.subtle` at 1/2/3, `bg.muted` at 3/4/5 and
`border.subtle` at 4/5/6 put `border.subtle.default` on the same step as
`bg.muted.hover` in both ladders. On an irregular ladder that only works by
accident.

**Every translucent ramp is inked from its own solid step 8.** Step 8 is the
fixed point of the flip — the one step that maps to itself — so it is the one
ink whose relationship to both grounds is symmetric. That is exactly what a
ramp which does not flip needs. It also disposes of the deleted `base` marker
without a marker: the ink is a convention, not an annotation.

For `grey` that moves the base from v1's `oklch(0.40 0.01 264)` to
`oklch(0.5676 0.01 264)`. cybr independently sits its `neutral.translucent`
base at `L 0.56`.

`black.translucent` and `white.translucent` keep `oklch(0 0 0 / α)` and
`oklch(1 0 0 / α)` and carry `mirror` markers pointing at each other.

## The ground ramp

`config.color.palette.ground.solid` is `#fff` at 1 and `#000` at 2. With
`N = 2` the reversal rule `n′ = (N + 1) − n` gives `1 ↔ 2`. It satisfies the
invariant trivially, being its own mirror, and it is exempt from the OKLCH rule
and from the shared ladder.

Four tokens read it: `surface.bg.low`, and `fg.onSolid.default` on `primary`,
`secondary` and `neutral`.

## What did not survive authoring

Four things in the schema did not make it through in the form they were
written.

**Authoring rule 1 is narrowed.** "One shared L ramp" holds for `grey`,
`neutral` and `stone` and is replaced by a shared _contrast ladder_ for the
four chromatic hues, for the reason above.

**`bg.muted` and `border.subtle` land heavier than v1's borders.** The rungs
sit at cybr's indices on Mirror's ladder, so `border.subtle.default` is a 30%
overlay where v1's `border.translucent.default` was 12% on `primary` and
`secondary`. The stack is internally consistent and the absolute weight is a
design call that has not been made. Check it on a real screen before release.

**`fg.onMuted` takes `fg.onSubtle`'s value, not `<hue>.solid.1`.** The schema
says to copy cybr, which sets both to step 1. cybr is dark-first, so its step 1
is light ink on a dark ground. Mirror is light-first, so the equivalent is the
group's own `onSubtle` value — step 15 for the groups whose overlay sits on the
page ground, step 8 for `secondary`. Copying the literal step number would have
put white ink on a white page.

**`chromaScale` is still inert.** It is declared on the dark source entry
because the config accepts it. Nothing reads it, because the chroma envelope is
authored into the ramps. Either drop the property or find it a home.
