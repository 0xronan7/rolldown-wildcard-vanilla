# Rolldown Wildcard Export Bug

**UPDATE:** This affects **both Vite 7 (Rollup) AND Vite 8 (rolldown)**.

## Reproduction

Minimal reproduction of wildcard exports resolution failure in @o7/icon package.

## Error

### Vite 7 (Rollup)
```
[commonjs--resolver] No known conditions for "./lucide/check.svelte" specifier 
in "@o7/icon" package
```

### Vite 8 (rolldown)
```
"./lucide/check" is not exported under the conditions ["module", "browser", "production", "import"]
from package @o7/icon (see exports field in package.json)
```

## Root Cause

The `@o7/icon` package uses wildcard exports:

```json
{
  "exports": {
    "./lucide/*": {
      "types": "./dist/lucide/*.svelte.d.ts",
      "svelte": "./dist/lucide/*.svelte"
    }
  }
}
```

This pattern is **not resolved correctly by either bundler**.

## Test Case

```js
// Both of these fail:
import CheckIcon from '@o7/icon/lucide/check';
import CheckIcon from '@o7/icon/lucide/check.svelte';
```

## Workaround

Use explicit paths without wildcard pattern:

```js
// This works but defeats the purpose of exports wildcards
import CheckIcon from '@o7/icon/dist/lucide/check.svelte';
```

Or the @o7/icon Vite plugin transforms barrel imports in dev mode, but production builds still fail.

## Reproduction Steps

```bash
# Vite 7 test
cd rolldown-wildcard-vite7-test
pnpm install
pnpm build  # FAILS

# Vite 8 test  
cd rolldown-wildcard-vanilla
pnpm install
pnpm build  # FAILS
```

## Expected Behavior

Wildcard exports should resolve:
- `@o7/icon/lucide/*` → `./dist/lucide/*.svelte`

## Actual Behavior

Both Rollup (Vite 7) and rolldown (Vite 8) fail to resolve wildcard export patterns.

## Related

- @o7/icon: https://github.com/ottomated/o7-icon
- Rolldown: https://github.com/rolldown/rolldown
- Rollup: https://github.com/rollup/rollup
