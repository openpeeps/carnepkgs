<p align="center">
  <img src="https://github.com/openpeeps/carne/blob/main/.github/carne_logo.png" alt="Carne Logo" width="140px" height="140px"><br>
  <strong>carnepkgs</strong>, the official package registry for Carne<br>
  Every package installable with <code>carne pkg install &lt;name&gt;</code> is listed here.
</p>

<p align="center">
  <a href="https://github.com/openpeeps/carne">openpeeps/carne</a> &bullet;
  <a href="https://github.com/openpeeps/carnepkgs/blob/main/packages.json">packages.json</a>
</p>

## What lives here

This repo contains one file that matters: [`packages.json`](packages.json).
It is the registry that Carne reads when you run `carne pkg install <name>`.

Carne is wired to this repo by default:

| Setting | Value |
|---|---|
| Source name | `carne` |
| Registry URL | `https://raw.githubusercontent.com/openpeeps/carnepkgs/main/packages.json` |
| Local cache | `~/.carne/registries/carne.json` |

Each entry in `packages.json` describes one installable package:

```json
{
  "name": "greet",
  "url": "https://github.com/<you>/greet.js",
  "method": "git",
  "tags": ["carne", "js", "greet", "example"],
  "description": "Say hi from Carne",
  "license": "MIT",
  "web": "https://github.com/<you>/greet.js"
}
```

Field notes (all fields are required):

| Field | Meaning |
|---|---|
| `name` | Package name. Lowercase, matches the `name` in the package `package.json`. Scoped names (`@scope/name`) are not supported. |
| `url` | Public git URL of the package repo. This is what gets cloned. |
| `method` | Always `"git"` for now. |
| `tags` | Search keywords. Include at least `carne` or `js`, plus what the package does. |
| `description` | One line summary shown to users browsing the registry. |
| `license` | SPDX identifier, e.g. `MIT`. |
| `web` | Homepage of the package (usually the repo URL). Entries without `web` are skipped by the client. |

## Package index

No packages yet. The first published package lands here, keeping entries sorted by `name`.

## Using packages: `carne pkg install`

Prerequisites: the Carne CLI. See [openpeeps/carne](https://github.com/openpeeps/carne) for install instructions.

### 1. Install a package by name

```bash
carne pkg install greet
```

This resolves `greet` in the `carne` source, clones it into the local
cache (`~/.carne/packages/_cache/greet`), picks the newest tagged version
(or `HEAD` when the repo has no tags yet), and copies a clean checkout to
`~/.carne/packages/greet/<version>`.

### 2. Install a specific branch or tag

```bash
carne pkg install greet#0.1.0   # a tag
carne pkg install greet#main     # a branch
```

### 3. Install straight from a git URL (no registry needed)

```bash
carne pkg install https://github.com/<you>/greet.js
carne pkg install https://github.com/<you>/greet.js#main
carne pkg install gh:<you>/greet.js
```

Any git transport works (`https://`, `git+`, `git://`, `git@`, `.git`
URLs), plus forge aliases (`gh:`/`github:`, `gl:`, `cb:`, `srht:`).
Handy for trying a package before it is published here, or for private repos.

### 4. Use the package in your scripts

Installed packages load through `require("pkg/<name>")` as CommonJS:

```js
const greet = require("pkg/greet");
console.log(greet.hello("ada")); // hi ada
```

A `@version` suffix pins an installed version
(`require("pkg/greet@0.1.0")`), and `pkg/<name>/sub/path` reaches
inside a package. Only CommonJS: ESM (`import`/`export`) packages fail
with a clear error.

### 5. Declare dependencies in `package.json`

List what your project or package needs so others get the full tree on install:

```json
{
  "name": "my-app",
  "version": "0.1.0",
  "description": "My Carne app",
  "license": "MIT",
  "main": "index.js",
  "dependencies": {
    "greet": "^0.1.0"
  }
}
```

Supported constraint operators: `=`, `==`, `>=`, `>`, `<=`, `<`, `^` (caret), `~` and `~>` (tilde).

### 6. Manage installed packages

```bash
carne pkg develop                # link the current directory as an editable package
carne pkg develop ./my-package   # link a specific directory
carne pkg list                   # list installed packages
carne pkg update greet           # update one package
carne pkg update                 # update everything
carne pkg remove greet           # remove all installed versions
carne pkg remove greet 0.1.0     # remove one version
carne pkg prune                  # remove orphaned packages
carne pkg refresh                # re-download registry data
```

### 7. Where things live on disk

| Path | Contents |
|---|---|
| `~/.carne/packages/_cache/<name>` | Bare git clone used for version discovery |
| `~/.carne/packages/<name>/<version>` | Clean installed copy your scripts resolve against |
| `~/.carne/develop/<name>` | Symlink created by `carne pkg develop` (editable, wins over registry copies) |
| `~/.carne/registries/carne.json` | Downloaded copy of this repo `packages.json` |
| `~/.carne/sources.json` | Configured sources (defaults to the `carne` source above) |

Set `CARNE_HOME` to move the whole store somewhere else.

### Troubleshooting

| Symptom | Fix |
|---|---|
| `Could not download registry for carne: curl failed ... 404` | Your cached registry predates this repo having a `packages.json`, or the file is unreachable. Check the Registry URL in the table above loads in a browser, then delete `~/.carne/registries/carne.json` and retry so Carne re-downloads it. |
| `No registry data seeded, run source.fetch` | Same cause: no registry was ever downloaded successfully. Fix connectivity to the Registry URL, clear the cached file above, and retry. |
| `Package not found in registry: <name>` | The name is not in `packages.json` (check spelling and the package index), or your local registry copy is stale (delete the cached file and retry). As a workaround, install by git URL (step 3). |
| Wrong version installed | The resolver picks the newest git tag satisfying your constraint. Push the missing tag on the package repo, or pin with `carne pkg install <name>#<ref>`. |

## Publishing a package

Publishing is a pull request to this repo. There is no `carne publish`
command; the manual route below is the whole process.

### Step 1. Put your package in a public git repo

Any public host that serves git over HTTPS works (GitHub recommended).

### Step 2. Add a `package.json` manifest at the repo root

Minimal example:

```json
{
  "name": "my-utils",
  "version": "0.1.0",
  "description": "Handy string helpers",
  "license": "MIT",
  "main": "index.js",
  "dependencies": {
    "greet": "^0.1.0"
  }
}
```

Rules:

- `name` must match the name you request in the registry (lowercase, no leading `.`/`_`, no spaces; scoped names are rejected).
- `version` is exact semver (`0.1.0`). It is used for `HEAD` installs until you push tags.
- `main` (or the `exports` map root, which wins over `main`) points at the entry file. Without either, Carne falls back to `index.js`, `src/index.js`, then `<name>.js`. Packages with no loadable entry fail at require time, so test step 4 first.
- `dependencies` maps names to semver ranges (see step 5 of the usage section for operators). Omit it when you have no dependencies.
- Only CommonJS (`module.exports`) loads in v1. ESM (`import`/`export`) packages fail with a clear error, so do not publish those yet.
- Keep generated output, vendored code, and editor files out of the repo. Installs copy the tree minus `.git`, tests, examples, and docs.

### Step 3. Tag your releases

```bash
git tag 0.1.0 && git push origin 0.1.0
```

Both `0.1.0` and `v0.1.0` tag styles are recognized. The resolver installs the
highest tag satisfying the requested constraint. Untagged repos still install
(the manifest `version` is used and recorded as `HEAD`), but tags are strongly
recommended so users can pin versions.

### Step 4. Test a local install before submitting

```bash
# from inside your package directory
carne pkg develop

# from anywhere else, install by URL to prove the repo is consumable
carne pkg install https://github.com/<you>/<repo>.git
```

Then require it from a script (`require("pkg/<name>")`, step 4 of the
usage section). If both work, the package is ready for the registry.

### Step 5. Submit the entry via a pull request

1. Fork this repo and add one JSON object to the array in `packages.json`, keeping entries sorted by `name`.
2. Double check every required field from the table above (`web` included, otherwise the client skips your entry).
3. In the PR description include: package repo URL, one line on what it does, and confirmation that `carne pkg install <url>` works.
4. A maintainer reviews, merges, and your package becomes installable by name (`carne pkg install <name>`) once the raw file updates (usually within minutes; users with a stale cache may need to delete `~/.carne/registries/carne.json` once).

Example diff:

```diff
   [
+    {
+      "name": "my-utils",
+      "url": "https://github.com/<you>/my-utils.js",
+      "method": "git",
+      "tags": ["carne", "js", "utils"],
+      "description": "Handy string helpers",
+      "license": "MIT",
+      "web": "https://github.com/<you>/my-utils.js"
+    }
   ]
```

Validation checklist for reviewers:

- [ ] Entry is valid JSON and keeps alphabetical order by `name`.
- [ ] `name` matches the `name` in the package `package.json`.
- [ ] `url` clones anonymously (`git ls-remote <url> HEAD` succeeds).
- [ ] Repo root contains `package.json` with `name`, `version`, `description`, `license`.
- [ ] The package loads via `require("pkg/<name>")` (has a reachable CommonJS entry).
- [ ] At least one semver tag exists (or the package is intentionally untagged and installs from the default branch).
- [ ] No name squatting or trademark conflicts.

### Updating or removing your package

- New versions need no registry change. Just push a new semver tag; Carne discovers tags on install.
- To change the description, tags, or repo URL, open a PR editing your entry.
- To unpublish, open a PR removing your entry and explain why. Installed copies keep working; only new installs stop resolving.

## Contributing

Issues and PRs are welcome. For registry entries, follow the publishing steps
above. For anything else (docs, tooling), open an issue first so maintainers
can point you in the right direction.

## License

MIT. See [LICENSE](LICENSE).
