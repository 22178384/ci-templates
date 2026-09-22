# usage

Copy-paste instructions for each template, plus the things that bit me.

## 1. Get the files into place

The workflows live in `workflows/` on purpose (this repo's checkout dir is
excluded from Actions, so nothing runs here). To use them in a real repo:

```bash
mkdir -p .github/workflows
cp workflows/build.yml   .github/workflows/
cp workflows/test.yml    .github/workflows/
cp workflows/release.yml .github/workflows/
cp workflows/docker.yml  .github/workflows/   # only if you have a Dockerfile
```

Then replace the placeholders. The only ones you must touch are:

- `build.yml`: nothing, unless your source isn't under `src/`.
- `test.yml`: the coverage path `--cov=src`. Change `src` to your package dir.
- `release.yml`: nothing. It reads the name from the repo.
- `docker.yml`: nothing required. Docker Hub login is optional, drop those
  two steps if you only push to GHCR.

## 2. Release flow

Tag and push:

```bash
git tag v0.3.0
git push origin v0.3.0
```

That fires `release.yml`. It will:

1. Build sdist + wheel.
2. Check `v0.3.0` == `project.version` in `pyproject.toml`. Mismatch = fail.
3. Publish to PyPI via trusted publishing (no token).
4. Create a GitHub Release with auto-generated notes and attach the artifacts.

### Setting up trusted publishing

No secret to create. On PyPI:

1. Go to your project → *Publishing* → *Add a new pending publisher*.
2. Owner: your GitHub user/org. Repository: the repo name.
3. Workflow name: `release.yml`. Environment: `pypi`.

That's it. The `id-token: write` permission in the workflow is what makes it
work. If you see `invalid-publisher` in the logs it's almost always the
environment name not matching.

## 3. Docker

GHCR push uses `GITHUB_TOKEN`, nothing to configure. Docker Hub (if you keep
it) needs two repo secrets:

- `DOCKER_USERNAME`
- `DOCKER_TOKEN` — a Docker Hub *access token*, not your password.

Build-only (no push) is the default on manual dispatch. Handy for checking
the Dockerfile still builds on a branch:

Actions → docker → Run workflow → leave "Actually push" unchecked.

## 4. Caching notes

`setup-python` with `cache: pip` caches the pip download dir keyed on your
dependency files. If you use a `requirements*.txt` set instead of
`pyproject.toml`, pass `cache-dependency-path: requirements*.txt`.

Poetry users: `cache: poetry`, and install Poetry before `setup-python` runs
the cache restore, or it warns that it can't find the Poetry cache dir.

## Troubleshooting

| Symptom | Cause |
| --- | --- |
| `denied: installation not allowed to Create organization package` | GHCR package already exists under a different owner. Check the package settings. |
| `invalid-publisher` | Trusted publisher config doesn't match (env name, workflow filename, repo). |
| Release job fails on the version check | Tag and `pyproject.toml` version drifted. Bump the version, re-tag. |
| macOS leg queued forever | Free-tier concurrency. Comment out the macOS `include` entry. |
| `error: invalid command 'bdist_wheel'` | You skipped `pip install build`. `python -m build` needs it. |

## Notes

These are pinned to action majors as of 2024. I don't maintain a Dependabot
config here. Bump the versions yourself when GitHub deprecates a runtime.
