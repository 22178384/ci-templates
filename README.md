# ci-templates

I got tired of copy-pasting the same four workflows into every new Python repo. So this is where they live now. Copy, paste, tweak the name, move on.

## Why the files aren't in `.github/workflows/`

Because I keep this repo checked out inside a directory where I don't want GitHub actually running the jobs on my scratch commits. The YAML sits in `workflows/` and you copy it into place yourself. Annoying, but it also means nothing fires until you deliberately opt in.

If you clone this and want it to self-test, just:

```bash
mkdir -p .github/workflows
cp workflows/*.yml .github/workflows/
```

## What's in here

| File | What it does |
| --- | --- |
| `workflows/build.yml` | Builds a wheel + sdist on every push to `main` and PRs. Uploads artifacts. |
| `workflows/test.yml` | Matrix across Python 3.9–3.12 on Ubuntu + macOS. Runs pytest with coverage. |
| `workflows/release.yml` | On a `v*` tag: builds, checks the version matches the tag, publishes to PyPI via trusted publishing. |
| `workflows/docker.yml` | Builds a multi-arch image and pushes to GHCR. Only runs on tags and manual dispatch. |
| `docs/usage.md` | Step-by-step copy-paste instructions + the gotchas I hit. |

## When each one fires

| Workflow | push to main | pull request | tag `v*` | manual |
| --- | --- | --- | --- | --- |
| build.yml | yes | yes | — | yes |
| test.yml | yes | yes | — | no (weekly cron instead) |
| release.yml | — | — | yes | — |
| docker.yml | — | — | yes | yes (build-only by default) |

`test.yml` also has a Monday 06:00 cron. That's the "a dependency released a breaking minor and nothing in my repo changed" detector. It has caught two of those for me. If you don't want the emails, delete the `schedule:` block.

## Assumptions these templates make

- Python project with `pyproject.toml` at the repo root.
- Tests runnable with `pytest`, config in `pyproject.toml` or `pytest.ini`.
- You want a wheel, not just a `pip install -e .`.
- Docker image, if you use it, is built from `Dockerfile` at the root.

If your layout is different, most of the changes are one-line edits to the `working-directory` or the build backend. Not going to write a config system for this.

## Quick start

1. Copy the ones you want into `.github/workflows/`.
2. Replace the obvious placeholders: `your-org/your-repo`, the package name in `build.yml`.
3. For `release.yml` set up a PyPI trusted publisher (no token needed) — see `docs/usage.md`.
4. For `docker.yml` the token is `${{ secrets.GITHUB_TOKEN }}`, nothing to add unless you push elsewhere.

## Gotchas

- **Caching**: `actions/setup-python` caches pip by default when you pass `cache: pip`. It keys off the lockfile / `pyproject.toml`. If you use Poetry, switch to `cache: poetry` and install Poetry first.
- **`fail-fast: false`** in the test matrix. Without it one red 3.9 job cancels the others and you lose the whole picture. Cost me an afternoon once.
- **macOS runners are slow to boot.** The matrix is nice-to-have. If your CI minutes are tight, comment out the macOS entries.
- **`release.yml` checks the tag against the package version.** If they don't match it fails on purpose. I've tagged the wrong commit too many times.
- **GHCR image names must be lowercase.** `github.repository` preserves case. The docker workflow lowercases it with a shell step because I hit this.

## Notes

Only tested on GitHub Actions. These are not portable to GitLab as-is. If you need GitLab, see my `yaml-configs` repo, there's a `.gitlab-ci.yml` there.

Last touched: 2024. Pinned action versions are the majors as of then (`v4`/`v5`). Bump them yourself, I'm not chasing Dependabot here.
