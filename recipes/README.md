# Recipes

One folder per package, in the Redox cookbook format:
`recipes/<group>/<name>/recipe.toml` and its patches, as under
`recipes/` in Redox's cookbook. A folder here with the same path as an
upstream recipe replaces it. List each package in `../config.toml`.

`scripts/build.sh` builds them against the redox-rpi release in
`../os-release`; see the [README](../README.md#building-against-an-os-release).
