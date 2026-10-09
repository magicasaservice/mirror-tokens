# Mirror Tokens

This repository holds the design tokens
[Mirror](https://github.com/magicasaservice/mirror) compiles by default.
`mirror init` points new projects at it, and `mirror tokens` downloads it and
compiles it into CSS custom properties and JS, TS and JSON files under
`.maas/tokens/`.

It is also meant as a starting point. If you want a token set of your own, copy
this one into your project with `mirror init --tokens` and edit it there.

The set covers color, radius, shadow and typography, plus border width and
outline width. There are no spacing tokens.

## Files

Every token is an object with a `$value` and a `$type`. A value in braces, such
as `{config.color.palette.grey.solid.15}`, references another token by its
path.

```
tokens/
  config.json          palette, radius base, font weights
  application.json     semantic tokens, the base set
  theme/dark.json      the dark color mode
  theme/mono.json      the mono theme
  breakpoint/sm.json   larger type from 640px up
  figma/type.json      composite typography for Figma text styles
```

### `config.json`

`config.json` holds the primitives under a `config` root, and the semantic
tokens in `application.json` reference them by path.

The palette has seven hues: `grey`, `neutral`, `stone`, `blue`, `red`, `green`
and `yellow`. Each hue has a 15-step `solid` ramp, running from its lightest
color at step 1 to its darkest at step 15, and a 15-step `translucent` ramp,
which is a single ink at alpha 0.12 up to 0.96 in steps of 0.06. The hue ramps
are written in OKLCH.

`black` and `white` hold one solid value each plus a translucent ramp, and
`ground` is a two-step ramp with white at 1 and black at 2. Next to the palette
sit `dimension.radius.base` (`0.25rem`), which the radius scales multiply, and
ten named font weights from `thin` (100) to `black` (900).

The `grey`, `neutral` and `stone` ramps are tuned so that step `n` read against
step 1 has the same contrast, to within half a percent, as step `16 - n` read
against step 15. That is what lets the dark mode mirror a step index and keep
the contrast the light mode had.

### `application.json`

`application.json` holds the semantic tokens under an `app` root. It is the
layer your application reads, through the generated classes and the `--app-*`
custom properties.

Every color in it references the palette, and the radii and font weights are
computed from the config scales. Font families, sizes, line heights, letter
spacing and text case are written as plain values.

Colors sit in two places. `app.color.surface` holds page and container colors,
and the intent groups (`primary`, `secondary`, `accent`, `danger`, `success`,
`warning`, `neutral`, `disabled`, `focus` and `shadow`) sit under
`app.color.component`. An intent color is addressed by role, fill and state, so
`app.color.component.primary.bg.solid.hover` is the hover fill of a primary
control.

The `component` and `default` segments are dropped from every generated name,
so that token compiles to `--app-color-primary-bg-solid-hover`. The
[token overview](https://github.com/magicasaservice/mirror/blob/main/apps/docs/content/1.tokens/0.overview.md)
explains every segment of a path.

### Themes and breakpoints

The other three JSON files restate some of the `app` tokens with different
values. Mirror’s stock config compiles each of them into a file of its own,
under a selector that outranks the base one, so the restated values win
wherever that selector matches. A token a file leaves out keeps its base value.

| File                 | Stock target                | Applies under                                                                       |
| -------------------- | --------------------------- | ----------------------------------------------------------------------------------- |
| `theme/dark.json`    | `theme/dark/application`    | `[data-color-mode="dark"][data-color-mode]`                                         |
| `theme/mono.json`    | `theme/mono/application`    | `[data-theme="mono"][data-theme][data-theme]`                                       |
| `breakpoint/sm.json` | `breakpoint/sm/application` | `:root:root, [data-color-mode][data-color-mode]` inside `@media (min-width: 640px)` |

`theme/dark.json` is the dark color mode, and it restates color tokens only.
Set `data-color-mode="dark"` on an element and the color tokens resolve to
their dark values on that element and everything inside it. Both modes read the
same palette, and most entries in the file follow one of two rules.

- A reference into a `solid` ramp mirrors its step, so step `n` becomes step
  `16 - n`. On the two-step `ground` ramp, 1 and 2 swap.
- A reference into the `black` or `white` translucent ramp keeps its step and
  moves to the other ink.

The remaining entries are tuned by hand, the link colors and the muted
foregrounds among them. A token that looks the same in both modes stays out of
the file.

`theme/mono.json` is a second presentation of the same set. It sets the title,
callout, body, caption and footnote roles and component text in Index, and
restates the weights, line heights and letter spacing that change with it. Set
`data-theme="mono"` on an element to apply it there.

`breakpoint/sm.json` raises the display, title and subtitle sizes and tightens
the display letter spacing from 640px up. It needs no attribute.

Brand themes are kept in the repository of the product that uses them, so this
set stops at `dark` and `mono`. The
[theming guide](https://github.com/magicasaservice/mirror/blob/main/apps/docs/content/1.tokens/2.theming.md)
covers loading the compiled files and setting the attributes.

### `figma/type.json`

`figma/type.json` holds one composite typography token per Figma text style.
Each one combines a font family, weight, size, line height and letter spacing,
and every one of those five members is a reference into `app`.

No build target compiles this file. `mirror tokens` copies it unchanged to
`.maas/tokens/figma/type.json`, and `mirror figma` turns each token into a text
style named after its path, with the family, weight and size bound to the
exported typography variables.

If your own copy has no such file, set `figma.typography` to `false` in
`mirror.config.ts` and the export skips text styles. The
[Figma styles guide](https://github.com/magicasaservice/mirror/blob/main/apps/docs/content/2.figma/1.styles.md)
covers the plugin that writes them into a Figma file.

## Typefaces

The text roles name two typefaces, and loading the font files is your app’s
job. Mirage is the standard family, used by every surface role except `code`
and by component text. It is a commercial typeface, licensed through Dreamtype
at [dreamtype.xyz/typefaces/mirage](https://dreamtype.xyz/typefaces/mirage).

Index sets `code`, numbers and the mono theme. It is free and open source, and
published at
[github.com/magicasaservice/index](https://github.com/magicasaservice/index).

## Branches

`main` holds the set for Mirror 2. It is the ref Mirror 2 fetches when `source`
names none, and the tree `mirror init --tokens` copies.

The `v1` branch keeps the previous tree for apps that have not moved yet.

## Using it with Mirror

Run `mirror init` in your project.

```bash
pnpm mirror init
```

It writes a `mirror.config.ts` that extends Mirror’s stock config and points
`source` at this repository without a ref, so every `mirror tokens` run
downloads the current `main`. That `source` is all a config needs to build this
set.

```ts
import { extendMirrorConfig } from '@maas/mirror'

export default extendMirrorConfig({
  source: { type: 'git', repo: 'magicasaservice/mirror-tokens' },
})
```

To hold a project on one state of the set, add a `ref` to `source`. It takes a
branch, tag or commit, and defaults to `main`.

```ts
source: {
  type: 'git',
  repo: 'magicasaservice/mirror-tokens',
  ref: '<commit>',
},
```

To build from another ref for a single run without editing the file, pass it to
`mirror tokens` as `--source github:magicasaservice/mirror-tokens#<ref>`. The
[CLI reference](https://github.com/magicasaservice/mirror/blob/main/apps/docs/content/0.overview/4.cli.md)
lists every argument.

## Starting your own set

To edit the tokens, run `mirror init --tokens`. It copies this repository’s
`tokens/` directory into `./tokens` and writes a `mirror.config.ts` whose
`source` reads that directory.

```bash
pnpm mirror init --tokens
```

```ts
source: { type: 'local', path: './tokens' },
```

Check `./tokens` in with the rest of the project. Nothing records where the copy
came from, so later changes to this repository do not reach it. If a
`mirror.config.ts` or a non-empty `./tokens` already exists, the command stops
before downloading anything; pass `--force` to write over both.

The stock targets read the files above by name, so keep the names if you want
the stock build. To add a theme, write a file under `tokens/theme/` that
restates only the tokens it changes, and add a target for it to `targets` in
`mirror.config.ts`.

```ts
targets: {
  'theme/brand/application': {
    include: [{ src: 'config' }, { src: 'application' }],
    selector: '[data-theme="brand"][data-theme][data-theme]',
    files: [{ src: 'theme/brand' }],
  },
},
```

That compiles `tokens/theme/brand.json` into
`.maas/tokens/css/theme/brand/application.css`, which applies under
`data-theme="brand"`. The
[configuration guide](https://github.com/magicasaservice/mirror/blob/main/apps/docs/content/1.tokens/1.configuration.md)
documents every key of a target.

## Found a bug?

If you see something that doesn’t look right,
[submit a bug report](https://github.com/magicasaservice/mirror/issues/new) on
the Mirror repository. See it. Say it. Sorted.
