# rush add bug?

If there are >1 contributors in package.json and there is no dependencies yet, running `rush add --package <pkg-name>` will cause the package.json to be malformed.

Check `pkg-a/package.json`:

```json
{
  "name": "pkg-a",
  "private": true,
  "version": "0.0.0",
  "contributors": [
    {
      "name": "person-a",
      "email": "example-a@example.org"
    },
    {
      "name": "person-b",
      "email": "example-b@example.org"
    }
  ]
}
```

Repro:

```sh
cd pkg-a
rush add --package meow
```

Expected:

- A valid JSON in package.json
- rush add runs successfully

```json
{
  "name": "pkg-a",
  "private": true,
  "version": "0.0.0",
  "contributors": [
    {
      "name": "person-a",
      "email": "example-a@example.org"
    },
    {
      "name": "person-b",
      "email": "example-b@example.org"
    }
  ],
  "dependencies": {
    "meow": "~13.2.0"
  }
}
```

Actual:


- Malformed JSON in package.json
- rush add failed

```json
{
  "name": "pkg-a",
  "private": true,
  "version": "0.0.0",
  "contributors": [
    {
      "name": "person-a",
      "email": "example-a@example.org"
    },
    {
      "name": "person-b",
      "email": "example-b@example.org"
    }
  ],
  "dependencies": {
    "meow": "~13.2.0",
  }
}
```

```log
➜  pkg-a git:(master) ✗ rush add --package meow
Found configuration in /path/to/repo/rush-repro-rush-add/rush.json


Rush Multi-Project Build Tool 5.124.6 - https://rushjs.io
Node.js version is 18.20.3 (LTS)

Found configuration in /path/to/repo/rush-repro-rush-add/rush.json

Starting "rush add"


Determining new version for dependency: meow
No version selector was specified, so the version will be determined automatically.

Trying to acquire lock for pnpm-8.15.8
Acquired lock for pnpm-8.15.8
Found pnpm version 8.15.8 in /path/to/user/.rush/node-v18.20.3/pnpm-8.15.8

Symlinking "/path/to/repo/rush-repro-rush-add/common/temp/pnpm-local"
  --> "/path/to/user/.rush/node-v18.20.3/pnpm-8.15.8"
The "ensureConsistentVersions" policy is NOT active, so we will assign the latest version.

Querying NPM registry for latest version of "meow"...

Found latest version: 13.2.0

Assigning version "~13.2.0" for "meow".
Updating projects to use meow@~13.2.0

Wrote/path/to/repo/rush-repro-rush-add/pkg-a/package.json

Running "rush update"

Checking Git policy for this repository.

Trying to acquire lock for pnpm-8.15.8
Acquired lock for pnpm-8.15.8
Found pnpm version 8.15.8 in /path/to/user/.rush/node-v18.20.3/pnpm-8.15.8

Symlinking "/path/to/repo/rush-repro-rush-add/common/temp/pnpm-local"
  --> "/path/to/user/.rush/node-v18.20.3/pnpm-8.15.8"
Transforming /path/to/repo/rush-repro-rush-add/common/config/rush/.npmrc
  --> "/path/to/repo/rush-repro-rush-add/common/temp/.npmrc"

Updating workspace files in /path/to/repo/rush-repro-rush-add/common/temp
Copying "/path/to/repo/rush-repro-rush-add/common/config/rush/pnpm-lock.yaml"
  --> "/path/to/repo/rush-repro-rush-add/common/temp/pnpm-lock.yaml"
Copying "/path/to/repo/rush-repro-rush-add/common/config/rush/pnpm-lock.yaml"
  --> "/path/to/repo/rush-repro-rush-add/common/temp/pnpm-lock-preinstall.yaml"

Checking installation in "/path/to/repo/rush-repro-rush-add/common/temp"

Deleting files from /path/to/repo/rush-repro-rush-add/common/temp/node_modules

Running "pnpm install" in /path/to/repo/rush-repro-rush-add/common/temp

 ERR_PNPM_JSON_PARSE  Unexpected token "}" (0x7D) in JSON at position 287 while parsing near "...meow\": \"~13.2.0\",\n  }\n}\n" in /path/to/repo/rush-repro-rush-add/pkg-a/package.json 

  15 |   "dependencies": {
  16 |     "meow": "~13.2.0",
> 17 |   }
     |   ^
  18 | }
  19 |


The command failed:
 /path/to/repo/rush-repro-rush-add/common/temp/pnpm-local/node_modules/.bin/pnpm install --store /path/to/repo/rush-repro-rush-add/common/temp/pnpm-store --config.cacheDir=/path/to/repo/rush-repro-rush-add/common/temp/pnpm-store --config.stateDir=/path/to/repo/rush-repro-rush-add/common/temp/pnpm-store --no-prefer-frozen-lockfile --no-strict-peer-dependencies --config.auto-install-peers=false --config.resolutionMode=highest --config.ignoreCompatibilityDb --recursive --link-workspace-packages false --reporter default
ERROR: Error: The command failed with exit code 1
Giving up after 1 attempts


ERROR: The command failed with exit code 1
```