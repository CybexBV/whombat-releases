# whombat-releases

The update files of [Whombat](https://whombat.com) boxes. Every box takes the newest release of its channel
(Production or Development) from here:

- **The signed metadata** (TUF) is in `metadata/` and served by GitHub Pages at `https://releases.whombat.com/`.
- **The files** are this repository's release assets: a release's tuple and closure indexes in the release named
  after its id (`whombat-YYYYMMDD.N`), the store paths' compressed blobs in 256 bucket releases `nar-00` … `nar-ff`,
  the AI client bundles in `clients`.

A box trusts nothing here unless the metadata signed it: the release keys are not on GitHub. The workflow in
`.github/workflows/metadata.yml` only re-signs the snapshot and timestamp, every 6 hours and whenever the
metadata changes. The tooling is `tufrepo` in the whombat-appliance repository (`release-tools/`).
