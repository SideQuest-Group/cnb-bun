# cnb-bun

A [Cloud Native Buildpack](https://buildpacks.io/) that installs [Bun](https://bun.sh) and builds your app. Works with any CNB-compatible platform — Heroku, Ancla, `pack` CLI, kpack, whatever.

## What it does

1. Detects `bun.lock` or `bun.lockb` in your repo
2. Downloads the Bun binary (cached between builds)
3. Runs `bun install --frozen-lockfile`
4. Runs your build scripts if they exist in `package.json`: `heroku-prebuild` → `build` → `heroku-postbuild`

If there's no `Procfile` and you have a `start` script, it sets `bun start` as the default web process. Otherwise it stays out of the way — useful when another buildpack (like Python) owns the process.

## Usage

With `pack` CLI and `heroku/builder:24`:

```sh
pack build my-app \
  --builder heroku/builder:24 \
  --buildpack https://github.com/SideQuest-Group/cnb-bun \
  --buildpack heroku/python
```

Or specify ordering in a `project.toml` at your repo root:

```toml
[_]
schema-version = "0.2"

[[io.buildpacks.group]]
uri = "https://github.com/SideQuest-Group/cnb-bun"

[[io.buildpacks.group]]
id = "heroku/python"
```

That second group entry is optional — only needed if your app has a Python backend too (like ours does).

## Pinning a Bun version

By default it grabs the latest release. To pin a specific verison:

**Option A** — `.bun-version` file in your repo root:
```
1.2.2
```

**Option B** — `BUN_VERSION` build-time env var:
```sh
pack build my-app --env BUN_VERSION=1.2.2 ...
```

## Multi-language builds

This buildpack is designed to play nice with others. The typical setup for a JS frontend + Python backend:

1. **cnb-bun** runs first — installs deps, builds static assets (e.g. `astro build` → `dist/`)
2. **heroku/python** runs second — installs Python + dependencies via uv/pip/poetry
3. Your `Procfile` starts the Python server, which serves the built static files

cnb-bun won't set a default process when it sees a `Procfile` or `Procfile.ancla`.

## Supported platforms

- Linux amd64 and arm64
- Any CNB builder that supports the [Buildpack API 0.10](https://github.com/buildpacks/spec/blob/main/buildpack.md)
- Tested with `heroku/builder:24`

## Links

- Repo: [github.com/SideQuest-Group/cnb-bun](https://github.com/SideQuest-Group/cnb-bun)
- Quick link: [get.ancla.dev/cnb-bun](https://get.ancla.dev/cnb-bun)

## Thats it

Three files, ~150 lines of bash. Read the source if somethings unclear — its short.
