# Raine Local Install

Build and link the local `opencode` binary from this checkout:

```bash
./raine/install-local.sh
```

The script runs `packages/opencode/script/install-bin --build`, which:

- builds the standalone binary when needed
- links it to `~/.local/bin/opencode` by default

Verify the install:

```bash
which opencode
opencode --version
```
