# Mirror Tokens

> An opinionated design token structure, powering [Mirror](https://github.com/magicasaservice/mirror)

## 💡 Intention

This repository is intended to power our own design system as well as serve as a blueprint for all future design system projects.

## 🚧 Construction

Our design tokens are structured in two hierachical layers:

1. `Application`
2. `Components`

The specificity of each layer topples the one prior. Tokens from each layer can either reference the layer below or include a finite value.

```
         __________
        /          \
       / components \
      /______________\
     /                \
    /      themes      \
   /____________________\
```

## ✨ Customization

Each token serves a purpose and can be updated if needed. Tokens further down the hierarchy have a wider spread influence on the final visual outcome compared to tokens further up the hierarchy. 

## 🎟️ Theming

In addition to existing tokens, themes can be introduced above the application or component layer. A theme holds tokens that overwrite existing tokens with different values. The Mirror CLI then allows to use those themes with i.e. a specific CSS selector.

```
         __________                     __________
        /          \                   /          \
       / components \                 /   themes   \
      /______________\               /______________\
     /                \             /                \
    /      themes      \           /    components    \
   /____________________\         /____________________\
  /                      \       /                      \
 /       application      \     /       application      \
/__________________________\   /__________________________\
```

## 🖇️ Structure

Each layer of tokens fulfills a different function. Therefore, adding, removing or refactoring tokens should be done in consideration of the respective layer’s purpose.

### application

> coming soon

### component

> coming soon


## 🧰 Tooling

The design token structure is currently built with [Tokens Studio](https://tokens.studio/) in Figma. The tokens than act as a source for a custom compiler within our [Design System](https://github.com/magicasaservice/design-system), built with [Style Dictionary](https://github.com/amzn/style-dictionary).

## 🐛 Found a Bug?

If you see something that doesn’t look right, [submit a bug report](https://github.com/magicasaservice/design-tokens/issues/new?assignees=&labels=bug%2Cpending+triage&template=bug_report.yml). See it. Say it. Sorted.
