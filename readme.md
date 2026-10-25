# pnpm patch bug repro

Demonstrates a bug in `pnpm patch` / `pnpm patch-commit` where the generated patch file is **corrupted**: instead of containing only the diff of the change made, the patch records every file in the package as `deleted file mode 100644`, producing a ~4000-line patch that does nothing useful.

## Environment

- `pnpm` 10.33.2
- Package patched: `react-aria-components@1.17.0`

## Steps to reproduce

```sh
pnpm install react-aria-components
pnpm patch react-aria-components
# pnpm opens a temporary working directory with the package contents
# edit a file — e.g. dist/private/utils.js — then run:
pnpm patch-commit <path printed by the previous command>
```

## Expected behavior

`patches/react-aria-components.patch` contains only a diff of the file(s) that were edited.

## Actual behavior

`patches/react-aria-components.patch` contains **no diff of the actual change**. Instead, every file in the package appears as a deletion:

```diff
diff --git a/.gitignore b/.gitignore
deleted file mode 100644
index 3023673dc096e96def8857aff9bc845de3846d98..0000000000000000000000000000000000000000
diff --git a/Autocomplete/package.json b/Autocomplete/package.json
deleted file mode 100644
...
```

The patch is ~4 000 lines long and marks every file as deleted, which means applying the patch would wipe out the entire package rather than apply the intended fix.

## Notes

- `pnpm-workspace.yaml` registers the patch via `patchedDependencies`.
- The edited file (`dist/private/utils.js`) is visible under `node_modules/.pnpm_patches/`, but the change is absent from the generated patch.
- Minimal reproduction repo: https://github.com/cyberuni/pnpm-patch-issue
