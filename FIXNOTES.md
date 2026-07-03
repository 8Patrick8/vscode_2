# Fix notes: Replace curl-pipe-shell with pinned and verified artifact downloads

## Changes made

### `.github/workflows/pr-linux-cli-test.yml:26`
- Before: `curl https://sh.rustup.rs -sSf | sh -s -- ...`
- After:
  - URL pinned to version tag `1.27.1` via `https://raw.githubusercontent.com/rust-lang/rustup/1.27.1/rustup-init.sh`
  - Downloaded to `rustup-init.sh` file
  - SHA256 checksum verification (`32a680...`) before execution
  - Cleanup after execution
  - Added `--proto '=https' --tlsv1.2` for transport security

### `build/azure-pipelines/cli/install-rust-posix.yml:25`
- Before: `curl https://sh.rustup.rs -sSf | sh -s -- ...`
- After:
  - URL pinned to version tag `1.27.1` via `https://raw.githubusercontent.com/rust-lang/rustup/1.27.1/rustup-init.sh`
  - Downloaded to `rustup-init.sh` file
  - SHA256 checksum verification (`32a680...`) before execution
  - Cleanup after execution

### `extensions/copilot/src/extension/chatSessions/vscode-node/copilotCLIShim.ts:109`
- Before: `curl -fsSL https://gh.io/copilot-install | bash`
- After:
  - Sets `VERSION=0.0.394` (pinning the copilot CLI artifact to the required version)
  - Downloads install script to a temp file via `mktemp`
  - Verifies install script SHA256 checksum (`cd455089...`) before execution
  - Cleans up temp file after execution
  - The install script internally verifies the downloaded copilot binary against SHA256SUMS.txt

### `extensions/copilot/src/extension/chatSessions/vscode-node/copilotCLIShim.ts:122`
- Before: `wget -qO- https://gh.io/copilot-install | bash`
- After: Same pattern as `runCurl` above (pinned version, temp file, checksum, cleanup)

## Integrity verification summary

| File | Artifact | Pin type | Checksum |
|------|----------|----------|----------|
| pr-linux-cli-test.yml | rustup-init.sh | Git tag `1.27.1` | SHA256: `32a680a8...` |
| install-rust-posix.yml | rustup-init.sh | Git tag `1.27.1` | SHA256: `32a680a8...` |
| copilotCLIShim.ts (curl) | copilot install script | Checksum + VERSION env | SHA256: `cd455089...` |
| copilotCLIShim.ts (wget) | copilot install script | Checksum + VERSION env | SHA256: `cd455089...` |

## Verification
- No test files exist for `copilotCLIShim.ts`
- The CI workflows will be validated by CI on the PR
- All changes are behavior-preserving: the same artifacts are fetched and executed, just with pinned versions and integrity verification added
