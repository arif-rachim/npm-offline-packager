# npm-offline-packager

A CLI tool to download and publish NPM packages tarboll (with all dependencies) <br>
for offline npm registry ([verdaccio](https://github.com/verdaccio/verdaccio), [artifactory](https://jfrog.com/artifactory/), etc.) 

## Install

```
$ npm install -g npm-offline-packager
```

## Usage


### npo fetch - Fetch packages tarball from npm registry

```bash
 $ npo fetch <list of packages or a path to package-json/package-lock file>
```

```
  Options:

    -p, --package-json <packageJson>  The path to package.json file
    -l, --package-lock <packageLock>  The path to package-lock.json file (npm lockfileVersion 2 or 3).
                                      Uses exact pinned versions and the full installed tree;
                                      bypasses the recursive resolver.
    --top <top>                       Fetch top packages from npm registry api. <max: 5250>
    -d, --dest <dest>                 Packages destination folder
    --no-tar                          Whether to create tar file from all packages
    --no-cache                        Whether to save download packages in cache
    --dev                             Whether to resolved dev dependencies
    --peer                            Whether to resolved peer dependencies
    --optional                        Whether to resolved optional dependencies
    -r, --registry <registry>         The registry url,Defaults to https://registry.npmjs.org/
    -h, --help                        output usage information
```

#### Examples

To fetch a list of packages
```bash
 $ npo fetch express @types/express bluebird
```

To fetch dependencies from package.json file
```bash
 $ npo fetch -p ./package.json
```

To fetch the exact installed tree from package-lock.json (recommended for reproducible offline bundles)
```bash
 $ npo fetch -l ./package-lock.json --dev --peer --optional
```

To fetch top n packages from npm registry api
```bash
 $ npo fetch --top n
```

#### `-p` vs `-l`: which to use?

| | `-p, --package-json` | `-l, --package-lock` |
|---|---|---|
| Versions | Semver ranges, resolved against the registry at fetch time | Exact pinned versions from the lockfile |
| Transitive deps | Resolved recursively via `pacote.manifest()` | Already flat in the lockfile — no extra network calls |
| Platform-specific optional deps (`@esbuild/*`, `@rollup/*`, etc.) | Only those matching the current OS, and only with `--optional` | All platforms that npm recorded at install time |
| Aliased / git / file deps | Not supported reliably (semver coerce strips `^`/`~` only) | Honored via the lockfile's `version` field |
| Reproducibility | Different runs may resolve to different versions | Bit-for-bit matches the project's lockfile |

Use `-l` when you want the offline bundle to match exactly what was installed on the source machine (typical for mirroring a project to a private registry). Use `-p` for quick one-off fetches or when you don't have a lockfile.

`--package-lock` requires npm lockfileVersion 2 or 3 (npm >= 7). For older lockfiles, regenerate with `rm package-lock.json && npm install` on a modern npm.

### npo publish - Publish packages tarball to private npm registry

```bash
$  npo publish <path to tarball file or folder>
```

```
  Options:

    -r, --registry <registry>      The private registry url
    -s, --skip-login               Whether to skip npm login command
    -f, --force                    Whether to publish with --force flag
    -c, --concurrent <concurrent>  How many packages to publish concurrently (default: 20)
    -h, --help                     output usage information
    --del-package                  After successful publication package deleting the package file (.tgz) 
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details