## To use

- `yarn`
- `yarn build`
- open `index.html` (no server required)

# My ideal BuckleScript monorepo

The goal is to run a single `bsb` from the project root and have everything Just Work (tm). To achieve this today, the only viable method is a single `bsconfig.json` at the root of the project with all packages in the `sources` list. That has multiple major downsides:

- the entire monorepo is flat, there's no ability to use the `namespace` setting to support separate packages
- a simple `bsb -clean` deletes _everything_, so rebuilds are unnecessarily slow
- `bsb` has to search the entire project for dependencies between files, whereas package dependencies more clearly show the bottlenecks and what to compile first (I'm not sure whether there is much performance to be gained here, but if dependencies are ever compiled in parallel it has potential)

This repository shows my best attempt at a single `bsb` root command, and with a few tweaks to `bsb` it could meet my goal. The setup:

- yarn workspace-based monorepo (this ensures there are no secondary `node_modules` folders in the packages which might confuse `bsb`)
- packages are listed under `bs-dependencies`, which works because the yarn workspace creates symlinks to each source package in `node_modules`
- `bsb -make-world` compiles everything

Everything _looks_ sweet at this point, in particular by using `bs-dependencies` for source packages `namespace` works just fine, but there are three critical flaws which prevent this from being viable:

- Warning and error settings in `bs-dependencies` are ignored. This is understandable in a normal project, you don't want to see warnings for code you can't control, but because monorepo source packages are modelled in this way no warning messages are ever produced.
  - the file `packages/b/src/Examples.re` has an unused parameter error, but it only shows up when running `bsb` in the `packages/b` folder
- To make `yarn watch` work, instead of using `bsb -w`, `watchman` is required (there are many watch tools, I chose it because I use fastpack and it already requires watchman). This isn't too bad, but running `bsb -make-world` is definitely slower than normal `bsb -w` rebuilds.
- reason-language-server seems to struggle to deal with multiple `bsconfig.json` files. I'm not sure if this is a specific bug or just general RLS instability I tend to experience.

In my ideal scenario, `bsb` would have some support for modeling `sources` as `bs-dependencies` (or perhaps a new config field such as `bs-source-dependencies`) such that they support their own config but are rebuilt in watch mode and make use of the appropriate warning/error flags.

### Source
Heavily modified from the ReasonReact Template.