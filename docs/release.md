# Release process

This guide describes the maintainer workflow for publishing a Daiquiri
release. The normal release path is:

1. prepare the version on `dev`;
2. merge the release change into `main`;
3. build and check the package with `dq-dev`;
4. upload the package to PyPI;
5. test the package installed from PyPI; and
6. tag and publish the GitHub release.

The `dq-dev --build-release` command builds and checks the package. It does
not update `package-lock.json`, upload to PyPI, or create the GitHub release.
The lock file is updated manually on `dev` before the release pull request.

## Prerequisites

You need:

- Docker Engine and Docker Compose v2;
- an installed checkout of `dq-dev`;
- an active `dq-dev` profile;
- Node.js and npm for updating `package-lock.json`;
- the Daiquiri source checkout; and
- the application checkout used by the profile.

For the release build, `dq-dev` must mount the Daiquiri source. In the active
profile's `conf.toml`, configure both `dq_source` entries:

```toml
[folders_on_host]
dq_source = "/path/to/daiquiri"

[docker_volume_mountpoints]
dq_source = "/home/dq/source"
```

The path in `folders_on_host` is the host path. The path in
`docker_volume_mountpoints` is the fixed path inside the container and should
normally remain `/home/dq/source`.

## Prepare the version

Work on the `dev` branch. Update the version in:

- `daiquiri/__init__.py`;
- `package.json`.

With npm installed, synchronize the lock file:

```bash
npm install --package-lock-only
```

Commit and push all three version files to `dev`. Wait for CI to complete,
then open a pull request from `dev` to `main`, for example with the title
`Release 1.3.9`. Merge it after the release change and CI results have been
reviewed.

Update the local `main` checkout after the merge:

```bash
git switch main
git pull --ff-only origin main
```

Confirm that the checked-out version is the intended release version.

## Build and check the package

Make sure the active profile points `dq_source` at the up-to-date Daiquiri
checkout. From the `dq-dev` repository, run:

```bash
python manage.py --build-release
```

The command starts the Daiquiri container using the normal startup path. In
release mode it then:

1. waits for initialization to finish;
2. installs `.[postgres,dev]` in the container;
3. removes the existing contents of `dist/`;
4. runs `daiquiri-admin build`; and
5. runs `twine check dist/*`.

The command streams the logs and stops the release container when it finishes.
The generated package files are written to the mounted source checkout:

```text
/path/to/daiquiri/dist/
```

Inspect the output and confirm that `twine check` succeeds. This command does
not upload anything to PyPI.

If the build fails, do not upload the contents of `dist/`. Fix the problem,
run the command again, and review the new artifacts.

## Optional `.dev` release

If the final release needs separate testing, create a unique development
version such as `1.3.9.dev1`. Keep it synchronized in
`daiquiri/__init__.py` and `package.json`, run
`npm install --package-lock-only`, and build it with the same command:

```bash
python manage.py --build-release
```

Upload the development package using the procedure below and test it from
PyPI before preparing the final version. Do not reuse a `.dev` version that
has already been uploaded because PyPI does not replace existing distribution
files.

## Upload to PyPI

Use an environment with Twine configured for the maintainer's PyPI account.
From the Daiquiri checkout, upload the checked files:

```bash
cd /path/to/daiquiri
twine upload dist/*
```

Use the normal PyPI credential or API-token configuration. Do not put tokens
in the repository or directly in commands that will be retained in shell
history.

If an upload fails because of a temporary network or service problem, resolve
the problem and retry the same artifacts. If PyPI reports that a file already
exists, do not attempt to replace it; use a new version.

## Test the PyPI package

Test the package from PyPI rather than the mounted source checkout:

1. Stop the running profile:

```bash
cd /path/to/dq-dev
python manage.py -p
```

2. Set the application's Daiquiri dependency to the exact PyPI version.
3. Comment out both `dq_source` entries in the active profile's `conf.toml`:
   the entry under `folders_on_host` and the entry under
   `docker_volume_mountpoints`.
4. Start the profile:

```bash
python manage.py -r
```

5. Check the installation logs and confirm that Daiquiri was installed from
   PyPI, not from a mounted local source.
6. Check the web interface and run a representative query.

If the test fails, stop the release process and resolve the problem before
creating the GitHub release. Restore the `dq_source` mount and the normal
application dependency after testing.

## Create the GitHub release

After the final PyPI package has been uploaded and tested, create the tag from
the merged `main` branch:

```bash
cd /path/to/daiquiri
git switch main
git pull --ff-only origin main
git tag 1.3.9
git push origin 1.3.9
```

Replace `1.3.9` with the actual version. Existing tags use the version without
a `v` prefix.

Create a GitHub release for the pushed tag. Use the generated tag-diff notes
as a starting point and organize the final release notes in this order:

1. **Breaking changes and compatibility changes** — put these at the top.
2. **Features** — include every actual feature.
3. **Fixes** — include every actual fix.
4. **Dependabot updates** — keep them in their own separate section.

Omit only irrelevant minor commits, such as internal refactors, typo fixes, or
other changes that do not affect users. Also describe any required updates in
the application repository.

Publish the GitHub release only after the final PyPI installation has been
verified.

## After the release

Restore the normal development configuration:

- restore both `dq_source` entries in `dq-dev`;
- restore the application's normal dependency configuration; and
- stop the profile if it is no longer needed.
