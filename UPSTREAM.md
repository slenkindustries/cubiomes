# Upstream sync procedure

This repository is a pinned fork of [xpple/cubiomes](https://github.com/xpple/cubiomes)
(itself a fork of [Cubitect/cubiomes](https://github.com/Cubitect/cubiomes), MIT).
It is consumed by `seedmap-engine` as a git submodule at a fixed commit.

We never develop features here. The only reasons this fork exists:

- An upstream force-push or deletion cannot break our builds.
- We can cherry-pick a new Minecraft version before upstream ships it, if ever
  needed.

## Remotes

- `origin`   — this fork (`slenkindustries/cubiomes`)
- `upstream` — https://github.com/xpple/cubiomes.git (the mirror source)
- `cubitect` — https://github.com/Cubitect/cubiomes.git (reference only)

## Syncing with upstream

### 1. Fetch upstream

```sh
git fetch origin
git fetch upstream
```

### 2. See what changed

```sh
git log --oneline master..upstream/master
git diff --stat master...upstream/master
```

### 3. Merge upstream into master

Merge, never rebase: this is a shared mirror and rewriting history would break
every consumer that has pinned the current SHA.

```sh
git checkout master
git merge upstream/master
```

If there are conflicts, resolve them in favour of upstream unless the change is
one of our own additions (there should normally be none besides the README
banner and this file).

### 4. Verify the version enum after merging

Check that `biomes.h` still contains the expected versions and that `MC_NEWEST`
resolves to the newest entry:

```sh
grep -nE 'MC_26_1|MC_26_2|MC_NEWEST' biomes.h
```

Optionally build and run the test suite via CMake from a scratch directory
(never add build output or configuration to this repo):

```sh
cmake -S . -B /tmp/cubiomes-build && cmake --build /tmp/cubiomes-build
```

### 5. Push

```sh
git push origin master
```

## Effect on consumers

`seedmap-engine` pins a specific SHA of this repository in its submodule
pointer. A sync performed here has **no effect** on downstream builds until
`seedmap-engine` explicitly bumps its submodule reference to the new SHA.
