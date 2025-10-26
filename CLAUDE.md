# Runtipi Docker Service

Debian package bundling Runtipi as a Docker Compose stack with systemd management.

## Git Workflow Policy

**IMPORTANT:** Always ask before committing, pushing, tagging, or running destructive git operations.

**Branch Workflow:** Never push to main directly - always use feature branches and PRs.

## Building

```bash
# Build using Docker (recommended)
./run package:deb:docker

# Build for CI
./run package:deb:docker:ci
```

## Version Management

**Automated** - only update `VERSION` file when tracking new Runtipi release:

```bash
echo "v4.6.0" > VERSION
git commit -m "Update to Runtipi v4.6.0"
# CI auto-generates debian/changelog and increments revision
```

**Version format:** `X.Y.Z-N` where X.Y.Z is upstream Runtipi version, N is packaging revision (auto-incremented by CI).

## Testing Locally

See [CLAUDE.local.md](CLAUDE.local.md) for remote testing procedures.

**Local debugging:**
```bash
/opt/runtipi/runtipi-cli --version
docker ps -a
cat /opt/runtipi/.env
```

**Common issues:**
- **Version shows 0.0.0**: TIPI_VERSION not set in docker-compose.yml
- **Update notification**: VERSION file doesn't match latest release
- **App install fails**: Check version with `journalctl -u runtipi | grep "Running version"`

## Architecture

- **Package**: Debian package for amd64/arm64
- **Stack**: 4 Docker containers (app, db, queue, reverse-proxy)
- **Management**: systemd runs `docker compose up/down` directly
- **Paths**: Flat structure under `/opt/runtipi/` (matches official runtipi-cli)
- **Downloads**: runtipi-cli binary and docker-compose.yml fetched at build time
