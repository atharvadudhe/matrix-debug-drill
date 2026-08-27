# Matrix CI Audit

## Failure Summary

| OS | Node Version | Failing Step | Failure Type |
|---|---:|---|---|
| ubuntu-latest | 22 | Run npm test | Runtime version |
| windows-latest | 18 | Run npm test | OS-specific |
| windows-latest | 20 | Run npm test | OS-specific |
| windows-latest | 22 | Run npm test | OS-specific + Runtime version |

## Detailed Findings

### 1. Ubuntu + Node 22

- **OS:** ubuntu-latest
- **Node version:** 22
- **Failing step:** Run npm test
- **Error:** `TypeError: crypto.createCipher is not a function`
- **Classification:** Runtime version
- **Root cause:** `crypto.createCipher` is no longer available in Node 22.
- **Planned fix:** Replace the deprecated `crypto.createCipher` implementation with the modern `crypto.createCipheriv` approach.

### 2. Windows + Node 18

- **OS:** windows-latest
- **Node version:** 18
- **Failing step:** Run npm test
- **Errors:**
  - `getOutputPath returns correct path` fails because the generated path uses `/` while the expected Windows path uses `\`.
  - `readTextFile returns file content with expected line endings` fails because Windows uses CRLF (`\r\n`) line endings while the test expects LF (`\n`).
- **Classification:** OS-specific
- **Planned fix:** Use `path.join()` for platform-independent path construction and normalize line endings before comparing file contents.

### 3. Windows + Node 20

- **OS:** windows-latest
- **Node version:** 20
- **Failing step:** Run npm test
- **Errors:**
  - `getOutputPath returns correct path` fails because the generated path uses `/` instead of the Windows path separator.
  - `readTextFile returns file content with expected line endings` fails because Windows returns CRLF (`\r\n`) while the test expects LF (`\n`).
- **Classification:** OS-specific
- **Planned fix:** Replace hardcoded path separators with `path.join()` and normalize line endings before comparison.

### 4. Windows + Node 22

- **OS:** windows-latest
- **Node version:** 22
- **Failing step:** Run npm test
- **Errors:**
  - `TypeError: crypto.createCipher is not a function`
  - `getOutputPath returns correct path` fails because of platform-specific path separators.
  - `readTextFile returns file content with expected line endings` fails because of CRLF vs LF line endings.
- **Classification:** OS-specific + Runtime version
- **Planned fix:** Replace `crypto.createCipher` with the modern crypto API, use `path.join()` for paths, and normalize line endings before comparison.

## Overall Diagnosis

The matrix pattern reveals two independent categories of problems:

1. **Operating-system-specific failures on Windows**
   - Hardcoded path separators
   - Hardcoded line-ending expectations

2. **Node 22 runtime compatibility failure**
   - Removal of `crypto.createCipher`

The Ubuntu Node 18 and Node 20 combinations pass, confirming that the existing implementation works on those runtime versions. The Windows failures occur across all Node versions, confirming that they are primarily operating-system compatibility issues.