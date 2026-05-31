# deterministically-undaunted

Mirror of rulebooks, scenarios, errata, and card images for the *Undaunted*
boardgame series (Normandy, North Africa, Stalingrad, Battle of Britain,
Reinforcements), tracked with [git-annex](https://git-annex.branchable.com/).

The git repository itself only stores symlinks to content-addressed
(SHA256E) keys. The actual PDFs and images live behind URLs registered
against git-annex's built-in `web` special remote — primarily the
publisher's CDN (Osprey Publishing), with Google Drive copies as backup
mirrors where available.

## Requirements

- `git`
- `git-annex` ([install instructions](https://git-annex.branchable.com/install/))

## Getting all the files

```sh
git clone git@github.com:cognivore/deterministically-undaunted.git
cd deterministically-undaunted
git annex init
git annex get .
```

`git annex get .` walks every annexed path, downloads the bytes from one
of the registered URLs, and verifies the SHA256 before materializing the
symlink target. If a URL is dead, try another remote or register a new
mirror (see below).

To fetch a single file:

```sh
git annex get rulebooks/normandy/undaunted_normandy_rulebook.pdf
```

## Inspecting where content lives

```sh
git annex whereis rulebooks/normandy/undaunted_normandy_rulebook.pdf
git annex info
git annex list
```

## Adding a new mirror URL for an existing file

If a registered URL goes down, register an additional source (e.g. a
Google Drive direct-download link) for the same key:

```sh
git annex addurl --file=rulebooks/normandy/undaunted_normandy_rulebook.pdf \
  'https://drive.google.com/uc?export=download&id=<FILE_ID>'
git annex sync
git push origin main git-annex
```

The SHA256E backend will reject the new URL if the bytes don't match the
existing key, so mirrors are safe to add without risk of silent
corruption.

## Adding a new file

```sh
git annex addurl --file=rulebooks/<set>/<name>.pdf '<source URL>'
git commit -m "Annex <name>"
git annex sync
git push origin main git-annex
```

## Releasing local disk space

`git annex` keeps full file contents under `.git/annex/objects/`. To free
space while keeping the symlinks:

```sh
git annex drop .
```

This only drops content if git-annex can confirm another remote still
holds a copy (here that means a reachable `web` URL).
