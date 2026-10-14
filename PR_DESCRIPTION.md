# Fix: Support partial wildcard in Windows proxy bypass rules

## Problem

Reader fails to load Windows system proxy settings when proxy software (like BetterBox, Clash Verge) sets bypass rules with partial wildcards like `172.2*`.

### Root Cause

The current `bypassEntries()` function in `windows-proxy.ts` only handles these wildcard formats:
- `*.example.com` (domain wildcards)
- `10.*` or `192.168.*.*` (complete IP wildcards where `*` is a standalone octet)

However, many proxy tools use **partial wildcards** like:
- `172.2*` (meaning `172.20.*.*` to `172.29.*.*`)
- `10.1*` (meaning `10.10.*.*` to `10.19.*.*`)

When encountering `172.2*`, the parser splits it into `['172', '2*']`, but `indexOf('*')` returns `-1` because there's no standalone `*` element. This triggers the error: **"Windows 系统代理包含不支持的直连例外"**

The error is then silently caught in `resolveSystemProxyEnvironment()` with only a generic warning, making it impossible to debug.

## Solution

1. **Add support for partial wildcards**: Detect patterns like `\d+.\d+\*` and expand them into complete ranges
   - `172.2*` → `172.20.*.*`, `172.21.*.*`, ..., `172.29.*.*`

2. **Improve error logging**: Log the actual error message in the catch block instead of swallowing it completely

3. **Add test coverage**: Include test cases for partial wildcard expansion

## Changes

### 1. Enhanced `bypassEntries()` in `windows-proxy.ts`

```typescript
// Handle partial wildcard like "172.2*" (matches 172.20.x.x to 172.29.x.x)
const partialMatch = /^(\d+)\.(\d+)\*$/.exec(entry);
if (partialMatch) {
  const [, first, secondPrefix] = partialMatch;
  const results: string[] = [];
  for (let i = 0; i <= 9; i++) {
    const secondOctet = `${secondPrefix}${i}`;
    if (Number(secondOctet) <= 255) {
      results.push(`${first}.${secondOctet}.*.*`);
    }
  }
  return results;
}
```

### 2. Better error logging in `proxy.ts`

```typescript
} catch (error) {
  const message = error instanceof Error ? error.message : String(error);
  console.warn("无法应用系统代理，Reader 将沿用启动环境。", message);
  // ...
}
```

### 3. New test cases in `windows-proxy.test.ts`

- Test partial wildcard expansion for `172.2*`
- Test mixed bypass lists with partial wildcards

## Testing

Manual test with BetterBox proxy settings:
```
Bypass: *jd.com;localhost;172.2*;192.168.*;127.0.0.1
```

Before: Reader shows "无法读取或转换系统代理"
After: Reader successfully loads system proxy with expanded ranges

## Breaking Changes

None. This is purely additive - existing valid configurations continue to work.

## Related Issues

Fixes users unable to use system proxy when using BetterBox, Clash Verge, or other proxy tools that generate partial wildcard bypass rules.
