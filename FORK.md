# Maintained deferred-focus patch

This fork publishes `@alexgorbatchev/base-ui@1.9.1`. It is based on upstream
[`@base-ui/react@1.9.0`](https://github.com/mui/base-ui/commit/ad7ecda5296b0295b41a51ecb01fce6204acd099)
and retains the upstream MIT license and MUI attribution.

The runtime patch ports upstream
[PR #6015](https://github.com/mui/base-ui/pull/6015), commit
[`b45439f`](https://github.com/mui/base-ui/commit/b45439f800dbece8d5b5e587252c81b6fb8f5b3c).
Deferred focus restoration checks the current active element again when the
animation-frame callback executes. It restores genuinely lost focus, but preserves
focus that has already moved to another element. Native tests cover removed and
disabled-then-removed elements, including the positive lost-focus case.

The fork does not incorporate other upstream development changes. Public exports,
peer dependencies, and the package's ESM, CommonJS, and TypeScript entrypoints retain
the upstream package structure. The workspace name remains `@base-ui/react`;
pnpm's `publishConfig.name` sets the published scoped identity.

Consumers use npm's native package alias to retain existing imports:

```json
{
  "dependencies": {
    "@base-ui/react": "npm:@alexgorbatchev/base-ui@1.9.1"
  }
}
```

## Release procedure

1. Install the pinned pnpm version and run `pnpm install --frozen-lockfile`.
2. Run `pnpm typescript`, `pnpm eslint`, `pnpm stylelint`, and `pnpm prettier`.
3. Run `pnpm test:chromium FloatingFocusManager Dialog Popover --no-watch` and
   `pnpm test:jsdom FloatingFocusManager Dialog Popover --no-watch`. Verify the new
   focus tests fail with the unpatched runtime and pass after exact restoration.
4. Run `pnpm --filter '@base-ui/react...' build` and
   `pnpm --filter @base-ui/react test:package`.
5. Pack only `@base-ui/react` with `pnpm --filter @base-ui/react pack`. Inspect the
   packed identity, license, dependencies, and all exports. Qualify ESM, CommonJS,
   and TypeScript consumers against that artifact.
6. Commit and push the reviewed source. Verify the scoped version is absent from
   npm, then publish the inspected tarball using `pnpm publish <tarball> --access public`.
   Do not run the upstream multi-package publishing workflow.
7. Verify registry metadata and downloaded tarball integrity, install the exact
   native alias in a consumer, and repeat entrypoint checks before updating the
   component library dependency. Create an immutable release tag at the reviewed
   commit and retain the published artifact checksum in its release notes.

The upstream package build removes package-local build output. Diagnostic receipts
are retained outside that output and independently backed up before release builds.
