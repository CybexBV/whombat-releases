# Setting up whombat-releases

What the repository needs once (Whombat LNK-24, LNK-25). The files in this folder go to the repository's `main`
branch as they are; the publisher's `tufrepo gh-init` adds `metadata/` and `.gitignore` to the same branch.

## Pages

- Settings → Pages → Build and deployment → Source: **GitHub Actions**.
- Custom domain: `releases.whombat.com`, **Enforce HTTPS** on. DNS (Cloudflare): `releases` CNAME
  `cybexhq.github.io`, DNS only (no proxy), so GitHub can issue the certificate.
- The `github-pages` environment: deployments from `main` only.

## Actions secrets and variables

| Name | Kind | What |
|---|---|---|
| `TUF_SNAPSHOT_KEY` | secret | the snapshot key file (JSON) from `tufrepo gh-continue`'s offline keys folder (`snapshot.json`) |
| `TUF_TIMESTAMP_KEY` | secret | the timestamp key file (JSON) from the same folder (`timestamp.json`) |
| `APPLIANCE_DEPLOY_KEY` | secret | while whombat-appliance is private: the private half of a read-only deploy key on it (the workflow checks it out with it); remove both once the repository is public |
| `APPLIANCE_REPO` | variable | the whombat-appliance repository that holds `tufrepo` (`owner/name`) |
| `APPLIANCE_REF` | variable | the commit of it the workflow builds `tufrepo` from (a full SHA; change it on purpose) |

No other signing key comes here: root, top-level targets and production are offline; development and clients stay on The
Beast. Settings → Actions → General: workflow permissions **read** by default (the workflow asks for what it needs).

## The publisher's token

A fine-grained personal access token (or a GitHub App) for this repository only, with **Contents: read and write**
(release assets and pushes to `main`). It lives where the DEV candidate runs, in `~/.config/whombat/releases.env`
(0600) as `WHOMBAT_RELEASES_TOKEN`, with `WHOMBAT_RELEASES_DIR` naming the publisher's clone (whombat-ops `bin/candidate`
reads that file), never in a file of a repository. Releases it creates are pre-releases and never "latest"; the publisher never touches anything else.

## Branch protection

`main` may be pushed by the publisher's token and the workflow (`github-actions[bot]`); no force pushes, no deletion.

## First run

1. `tufrepo gh-continue --from <copy of DEV's repository> --out <dir> --keys-dir <online keys> --offline-keys-dir <offline>`
2. The two secrets above from `<offline>`; the production key to its offline place.
3. `WHOMBAT_RELEASES_TOKEN=… tufrepo gh-init --dir <dir> --remote https://github.com/CybexHQ/whombat-releases.git`:
   it first uploads and checks every file the metadata names (the carried releases, the AI clients), then pushes the
   metadata. If it stops, run it again: files already on GitHub are skipped.
4. Run the workflow once (workflow_dispatch) and check `https://releases.whombat.com/timestamp.json`.

The whole move, with the freeze and the order of the boxes: `docs/runbooks/updates.md`, “The bridge”.
