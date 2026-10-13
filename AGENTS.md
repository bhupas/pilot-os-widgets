# AGENTS.md

Notes for coding agents, and people, working in this repo. [README.md](README.md) says what the
store is and the rules a widget is held to; [docs/LANGUAGE.md](docs/LANGUAGE.md) is the widget
language.

## What this is

The widget store for Pilot OS, an Android home screen that runs Arch Linux inside (app id
`dev.pilot_os`, debug builds `dev.pilot_os.debug`). The app's source is the private repo
bhupas/pilot-os. This repo is a fork of SimonSchubert/omarchy-widgets, the store of the launcher
Pilot OS was forked from.

On every push to main, CI builds `index.json` from `widgets/` and publishes it, with `widgets/`
and `w/`, to GitHub Pages:

- the app's Store reads https://bhupas.github.io/pilot-os-widgets/index.json
- shared links are https://bhupas.github.io/pilot-os-widgets/w#owl1.<data>, and `w/index.html`
  is the page they land on

The repo used to be bhupas/omarchy-widgets. GitHub redirects the repo after a rename, but not its
Pages site, so the Pages URL changed with it: anything that still points at
`bhupas.github.io/omarchy-widgets` has to move to `bhupas.github.io/pilot-os-widgets`.

## Layout

- `widgets/<author>/<name>/`: one reviewed widget or mini app each, written by its author.
- `order.txt`: the order of the store.
- `docs/LANGUAGE.md`: the language. `w/index.html`: the shared-link page. `art/`: the icon.
- `tools/owl.jar`: the checker and the language's reference implementation, built from the app
  repo with `./gradlew :owl:owlJar` (output `owl/build/cli/owl.jar`). `tools/verify-pr.sh`: what
  the review check runs on a pull request.
- `.github/workflows/`: `review.yml` checks pull requests that touch `widgets/`, `publish.yml`
  publishes main, `runtime.yml` holds `runtimes/quickshell` to `tools/owl.jar`.
- `runtimes/quickshell/`: OWL for Quickshell on Linux.

## Checks

```sh
java -jar tools/owl.jar index .                                      # build index.json (gitignored), as publish.yml does
java -jar tools/owl.jar review . widgets/<author>/<name>/widget.owl  # the rules a pull request is held to
runtimes/quickshell/test/run.sh                                      # Owl.js matches owl.jar, as runtime.yml does; needs a JDK and node
```

`index` prints the widgets it skips and why; a skipped widget drops out of the store.

## Rules that matter most

- Never edit other authors' widgets, except to fix the store (a widget that breaks the index or
  the checks). Say why in the commit message.
- CI publishes on every push to main, straight to the app's users. Work on a branch, run the
  checks, then push to main.
- Keep the review as it is: `review.yml` runs on `pull_request_target` and never checks out or
  runs a pull request's code, and the rules in README.md are what `owl review` enforces.
- Replace `tools/owl.jar` only with one built from the app repo, and run the runtime test after.

## Syncing with the original store

`origin` is https://github.com/bhupas/pilot-os-widgets.git, this store, and `upstream` is
https://github.com/SimonSchubert/omarchy-widgets.git, the original. On a fresh clone, add it with
`git remote add upstream https://github.com/SimonSchubert/omarchy-widgets.git`.

To take in upstream's changes:

```sh
git checkout -b sync-upstream main
git fetch upstream && git merge upstream/main
```

Resolve conflicts by keeping Pilot OS branding in `README.md`, `w/index.html` and `docs/`, and
upstream's widget changes (`widgets/`, `order.txt`). Upstream's links point at
simonschubert.github.io/omarchy-widgets; this finds any that came in with the merge:

```sh
grep -rIn -i "omalauncher\|omarchy-proot\|simonschubert.github.io" --exclude-dir=.git --exclude-dir=widgets .
```

The hits in this file and under `runtimes/quickshell/` are expected: the runtime keeps upstream's
comments, and `dev.omalauncher` is the Kotlin package inside `tools/owl.jar`, which Pilot OS keeps.

Run the checks, then merge the branch into main and `git push origin main`.
