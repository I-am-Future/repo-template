# repo-template

A universal Python repo template (uv + hatchling + ruff + mypy + pytest).

## TODO (onboarding checklist)

- [ ] Rename `src/mypackage/` to your actual package name.
- [ ] Update `[project].name`, `description`, `authors`, `version` in `pyproject.toml`.
- [ ] Run `uv sync --extra dev`.
- [ ] Replace `README.md` with your project's real description.
- [ ] (Optional) Add PyTorch CUDA extras per the section below.
- [ ] (Optional) Add a `tests/` dir — `pytest` is already configured to look there.

## Quickstart

```bash
uv sync --extra dev        # install runtime + dev deps into .venv
```

## Layout

```
src/mypackage/       # rename this dir + update [project].name and
                     # [tool.hatch.build.targets.wheel].packages
tests/
pyproject.toml
```

## ⭐ Adding program entry point

To expose a CLI command, uncomment and edit `[project.scripts]` in `pyproject.toml`:

```toml
[project.scripts]
mycli = "mypackage.cli:main"   # `mycli` in PATH → calls mypackage/cli.py::main()
```

Then create `src/mypackage/cli.py` with a `def main() -> None:` function. Re-run `uv sync` so the script is installed into `.venv/bin/`.

## ⭐ Remove uv.lock from .gitignore

For applications, commit `uv.lock` so builds are reproducible. Delete line 210 of `.gitignore`:

```
uv.lock
```

Then `git add uv.lock && git commit`. (For libraries published to PyPI, leave it ignored.)

## ⭐ Adding PyTorch with a specific CUDA version

Under `OPTIONAL DEPENDENCIES` in `pyproject.toml`:
```toml
# Users install with: uv sync --extra cu128
cu128 = ["torch>=2.6", "torchvision", "torchaudio"]   # RTX 50 series, CUDA 12.8+
cu126 = ["torch>=2.6", "torchvision", "torchaudio"]   # RTX 40 series, CUDA 12.6
```

Under `UV — PYTORCH INDEX SOURCES`:
```toml
[[tool.uv.index]]
name = "pytorch-cu128"
url  = "https://download.pytorch.org/whl/cu128"
explicit = true

[[tool.uv.index]]
name = "pytorch-cu126"
url  = "https://download.pytorch.org/whl/cu126"
explicit = true

# Add this in the same time for mutual exclusion
[tool.uv]
conflicts = [
    [
        { extra = "cu128" },
        { extra = "cu126" },
    ],
]
```

Under `UV — DEPENDENCY SOURCES`:
```toml
torch = [
    { index = "pytorch-cu128", extra = "cu128" },
    { index = "pytorch-cu126", extra = "cu126" },
]
torchvision = [
    { index = "pytorch-cu128", extra = "cu128" },
    { index = "pytorch-cu126", extra = "cu126" },
]
torchaudio = [
    { index = "pytorch-cu128", extra = "cu128" },
    { index = "pytorch-cu126", extra = "cu126" },
]
```

## ⭐ Installing local packages (sibling dirs, submodules)

To depend on a package that lives next to this repo (or inside `submodules/`) instead of PyPI, declare it as a normal dependency and route it to a local path via `[tool.uv.sources]`:

```toml
[project]
dependencies = [
    "myutils",
    "privlib",
]

[tool.uv.sources]
myutils = { path = "../myutils",           editable = true }   # sibling dir
privlib = { path = "./submodules/privlib", editable = true }   # git submodule
```

`editable = true` means changes in the source dir are picked up without reinstall. Re-run `uv sync` after adding. Each target must itself be a valid Python project (has its own `pyproject.toml`).
