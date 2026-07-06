# CLAUDE.md

Eugo fork (`eugo-inc/aws-crt-python`, branch `eugo-main`) of
awslabs/aws-crt-python: Python 3 bindings for the AWS Common Runtime
(package `awscrt`, C extension `_awscrt` from `source/*.c` + `crt/` deps).

## What is eugo-only

Entire divergence from the last-merged upstream tip `6587c26` is three
spots (audit anytime: `git diff 6587c26 HEAD --stat`):

- `setup.py` -> adds `using_shared_libs()` / `AWS_CRT_BUILD_USE_SHARED_LIBS=1`
  (currently defined but uncalled: since the 2025-07 merge, upstream's
  `forcing_static_libs()` / `AWS_CRT_BUILD_FORCE_STATIC_LIBS=1` gates the
  static `-l:lib*.a` trick, so default linking is shared). The file is also
  fully reformatted (double quotes, f-strings) vs upstream single quotes.
- `.devcontainer/` -> arm64 eugo toolchain container (clang/cmake checks,
  docker-socket mount for the eugo-kb MCP backend).
- `.mcp.json` -> GitHits (npx) + eugo-kb (docker exec eugo-kb-tools).

This repo has NO `@EUGO_CHANGE` markers (unlike eugo's pytorch fork).
NEVER introduce them here -> keep divergence confined to the files above.

## Build: this checkout is in system-libs mode

`crt/` submodules are NOT initialized (check: `git submodule status` -- every
line starts with `-`). `setup.py::using_system_libs()` is true whenever
`crt/aws-c-common/CMakeLists.txt` is absent, so `python3 -m pip install .`
links system-installed aws-c-* / s2n / libcrypto instead of building
vendored submodules.

- Build fails with missing `aws-c-*` / `s2n` libs -> the host lacks the
  system deps; NEVER reflexively run `git submodule update --init` to fix
  it -> that silently flips the build to vendored mode; install the system
  libs, or flip modes deliberately and say so.
- Need an explicit mode regardless of submodule state -> env vars read in
  `setup.py`: `AWS_CRT_BUILD_USE_SYSTEM_LIBS=1`,
  `AWS_CRT_BUILD_USE_SYSTEM_LIBCRYPTO=1`, `AWS_CRT_BUILD_FORCE_STATIC_LIBS=1`.

## Test

```bash
python3 -m unittest discover --failfast --verbose      # pass = exit 0
python3 -m unittest --failfast --verbose test.test_http_client.TestClient.test_connect_http
```

Keep `--failfast`: per guides/dev/README.md a failed test can leak memory
and cascade later failures. Many tests need AWS-credential env vars and
self-skip without them -- skips are normal, errors are not.

## About to merge upstream

- Policy is merge, not rebase (evidence: merge commits `ca9a828` "Merge
  branch 'awslabs-main'" and `fa007f0`). This checkout has only `origin`
  (eugo-inc) -> add upstream first:
  `git remote add awslabs https://github.com/awslabs/aws-crt-python.git`.
- Expect `setup.py` to conflict wholesale (the eugo-side reformat). Resolve
  hunk-by-hunk keeping eugo's formatting; the prior resolution to imitate
  is `git show fa007f0 -- setup.py`.
- Done resolving -> `git diff <new-upstream-tip> HEAD --stat` must list only
  `setup.py`, `.devcontainer/`, `.mcp.json`, `CLAUDE.md` plus intentional
  new work; anything else is an unexplained divergence -- justify or drop it.
