# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow Policy

**IMPORTANT:** Always ask the user before:
- Committing files to git
- Pushing commits to remote repositories
- Creating or modifying git tags
- Running destructive git operations

**Branch Workflow:**
- **NEVER push directly to main** - Always create a branch and PR
- Create feature branches for changes
- Push feature branches and create PRs
- Only merge to main via approved PRs

## Overview

Runtipi provides a user-friendly web interface for Docker container management and self-hosted application deployment:
- Container management and monitoring
- App store with 300+ pre-configured applications
- Web UI accessible on port 80
- Simple orchestration using runtipi-cli

## Package Architecture

This Debian package bundles Runtipi as a Docker Compose stack with systemd management.

**Components:**
- **docker-compose.yml**: Defines the 4-container Runtipi stack (reverse proxy, database, queue, app)
- **runtipi-cli**: Bundled for user convenience and manual operations
- **systemd service**: Manages the Docker Compose stack lifecycle

**Management approach:**
- systemd controls the stack using `docker compose up/down` directly
- runtipi-cli is available for manual operations but not used by systemd
- This provides standard systemd→docker compose integration

**Key characteristics:**
- Built on Docker Compose with 4 containers
- Extensive app catalog (300+ applications)
- Active upstream development and community
- Web UI on port 80/443

## Package Build Process

1. **debian/rules** downloads during build:
   - `runtipi-cli` binary (architecture-specific: x86_64 or aarch64)
   - `docker-compose.prod.yml` from GitHub repo
2. Architecture detection (amd64 → x86_64, arm64 → aarch64)
3. Binary extracted from tar.gz and made executable
4. Files installed to `/opt/runtipi/`:
   - `runtipi-cli` - CLI tool
   - `docker-compose.yml` - Stack definition
5. Systemd service installed to `/lib/systemd/system/runtipi.service`
6. Directory structure created in `.internal/` (media, state, repos, apps, etc.)
7. Default `.env` file created

## Development

### Building the Package

```bash
# Build locally (requires Debian tools)
./run package:deb

# Build using Docker (recommended)
./run package:deb:docker

# Build for CI
./run package:deb:docker:ci
```

### Testing Locally

```bash
# After building, install the package
sudo dpkg -i ../runtipi-docker-service_*.deb

# Check service status
sudo systemctl status runtipi

# View logs
sudo journalctl -u runtipi -f

# Access web UI
open http://localhost
```

### Version Management (Automated)

**Package versioning is fully automated** - you only need to update the `VERSION` file when tracking a new Runtipi release.

**Version Format:**
- Package versions follow Debian convention: `X.Y.Z-N`
- `X.Y.Z`: Upstream Runtipi version (from VERSION file)
- `-N`: Debian packaging revision (auto-incremented by CI)

**How It Works:**

The build workflow automatically:
1. Reads the upstream version from the `VERSION` file (e.g., `v4.5.2`)
2. Checks existing GitHub releases to find the next available revision number
3. Generates `debian/changelog` with the auto-incremented version (e.g., `4.5.2-1`)
4. Builds the package with that version
5. Creates a draft release tagged as `v4.5.2-1`

**When to Update VERSION File:**

Update the `VERSION` file only when tracking a new Runtipi release:

```bash
# Check latest version at https://github.com/runtipi/runtipi/releases
echo "v4.6.0" > VERSION
git add VERSION
git commit -m "Update to Runtipi v4.6.0"
# Create PR, merge → build workflow creates v4.6.0-1
```

**Packaging Changes (No VERSION Update):**

For packaging-only changes (e.g., fix systemd service, update dependencies):
- Don't update the VERSION file
- Just make your changes and create a PR
- Build workflow will auto-increment the revision: `4.5.2-1` → `4.5.2-2`

**Examples:**

| Scenario | VERSION file | Last Release | New Version | Description |
|----------|--------------|--------------|-------------|-------------|
| New Runtipi version | `v4.6.0` | `v4.5.2-1` | `v4.6.0-1` | First packaging of 4.6.0 |
| Packaging fix | `v4.5.2` | `v4.5.2-1` | `v4.5.2-2` | Fixed systemd service |
| Another fix | `v4.5.2` | `v4.5.2-2` | `v4.5.2-3` | Updated dependencies |
| New upstream again | `v4.7.0` | `v4.5.2-3` | `v4.7.0-1` | Track new release |

**Notes:**
- `debian/changelog` is auto-generated during CI builds (not tracked in git)
- `.bumpversion.cfg` only tracks the `VERSION` file
- Revision numbers ensure no conflicts even with multiple builds of same upstream version

### Testing on Remote Systems

```bash
# Build the package
./run package:deb:docker:ci

# Copy to remote system
scp runtipi-docker-service_*.deb pi@halpi2.local:

# Install on remote system
ssh pi@halpi2.local "sudo dpkg -i runtipi-docker-service_*.deb"

# Reload systemd and restart service
ssh pi@halpi2.local "sudo systemctl daemon-reload && sudo systemctl restart runtipi"

# Check service status
ssh pi@halpi2.local "sudo systemctl status runtipi --no-pager"

# Monitor logs for version information
ssh pi@halpi2.local "sudo journalctl -u runtipi -f"
```

### Verifying Installation

```bash
# Check that the correct version is running
ssh pi@halpi2.local "sudo journalctl -u runtipi -n 100 --no-pager | grep 'Running version'"
# Expected output: info > Running version: v4.5.2

# Verify docker-compose.yml has correct image and TIPI_VERSION
ssh pi@halpi2.local "grep -E 'image: ghcr.io|TIPI_VERSION' /opt/runtipi/docker-compose.yml"
# Expected output:
#   image: ghcr.io/runtipi/runtipi:v4.5.2
#   TIPI_VERSION: v4.5.2

# Check all containers are running
ssh pi@halpi2.local "docker ps | grep runtipi"
# Expected: 4 containers (runtipi, runtipi-db, runtipi-queue, runtipi-reverse-proxy)

# Access web UI
# Navigate to http://halpi2.local in browser
```

### Debugging Issues

**Local debugging:**
```bash
# Check runtipi-cli status
/opt/runtipi/runtipi-cli --version

# Check Docker containers
docker ps -a

# Manual start (for debugging)
cd /opt/runtipi
sudo ./runtipi-cli start

# Check configuration
cat /opt/runtipi/.env

# Verify docker-compose.yml transformation was applied correctly
grep -A 2 "runtipi:" /opt/runtipi/docker-compose.yml
# Should show: image: ghcr.io/runtipi/runtipi:vX.Y.Z
# NOT: build: ...
```

**Remote debugging:**
```bash
# Check service status and recent logs
ssh pi@halpi2.local "sudo systemctl status runtipi --no-pager -l"

# Follow logs in real-time
ssh pi@halpi2.local "sudo journalctl -u runtipi -f"

# Check for errors in logs
ssh pi@halpi2.local "sudo journalctl -u runtipi -n 200 --no-pager | grep -i error"

# Verify directory structure
ssh pi@halpi2.local "find /opt/runtipi -type d | head -20"

# Check container health
ssh pi@halpi2.local "docker ps --filter 'name=runtipi' --format 'table {{.Names}}\t{{.Status}}'"

# Restart service if needed
ssh pi@halpi2.local "sudo systemctl restart runtipi"

# View container logs directly
ssh pi@halpi2.local "docker logs runtipi -f"
```

**Common issues:**
- **Version shows 0.0.0**: TIPI_VERSION not set correctly in docker-compose.yml (check debian/rules regex)
- **"Update available" notification**: Version mismatch - ensure VERSION file matches latest Runtipi release
- **Build section error**: docker-compose.yml transformation failed - verify Python regex in debian/rules
- **App installation fails with version requirement**: Check actual version with `journalctl -u runtipi | grep "Running version"`

## File Structure

```
runtipi-docker-service/
├── VERSION                    # Runtipi version to install
├── debian/
│   ├── changelog              # Debian package changelog
│   ├── control                # Package metadata and dependencies
│   ├── rules                  # Build rules (downloads runtipi-cli)
│   ├── install                # File installation mappings
│   ├── postinst               # Post-installation script
│   ├── postrm                 # Post-removal script
│   └── source/format          # Package format
├── runtipi.service            # Systemd service definition
├── run                        # Build script
└── docker/                    # Docker build tools
    ├── Dockerfile.debtools
    └── docker-compose.debtools.yml
```

## Systemd Service

The service:
- Runs `docker compose up` directly from `/opt/runtipi/`
- Stops with `docker compose down`
- Depends on Docker service
- Restarts automatically on failure
- WorkingDirectory is `/opt/runtipi/`

**Why not use runtipi-cli for systemd?**
- runtipi-cli is just a wrapper around docker compose
- Direct docker compose integration is more standard and debuggable
- runtipi-cli remains available for manual user operations

## Important Notes

- **Version tracking**: VERSION file must match a valid GitHub release tag
- **Architecture support**: amd64 (x86_64) and arm64 (aarch64) only
- **Downloads during build**: Both runtipi-cli and docker-compose.yml downloaded at package build time
- **Container architecture**: Runtipi itself runs in containers (4-container stack)
- **CLI purpose**: Bundled for user convenience, not used by systemd
- **Docker requirement**: Full Docker Compose stack (reverse proxy, database, queue, app)

### Path Configuration

**Critical:** Runtipi's upstream docker-compose uses `.internal/` subdirectories for all data storage. This package transforms those to use a flat directory structure directly under `/opt/runtipi/` to match the official runtipi-cli behavior.

**How it works (matching official runtipi-cli):**

1. `debian/rules` transforms `docker-compose.prod.yml` at build time:
   - Changes `${RUNTIPI_*_PATH:-.internal}` to `${RUNTIPI_*_PATH:-.}` in volume mounts
   - Changes `./.internal/` paths to `./`
   - Result: Volume mounts like `${RUNTIPI_APP_DATA_PATH:-.}/app-data:/app-data`

2. `debian/postinst` sets minimal environment variable in `.env`:
   - Only sets `RUNTIPI_APP_DATA_PATH=/opt/runtipi` (the root folder)
   - Other PATH variables (`RUNTIPI_MEDIA_PATH`, etc.) are left unset
   - When unset, they default to `.` (current directory) in docker-compose

3. Volume mount expansion (with WorkingDirectory=/opt/runtipi):
   - `${RUNTIPI_APP_DATA_PATH:-.}/app-data` → `/opt/runtipi/app-data` (absolute path)
   - `${RUNTIPI_MEDIA_PATH:-.}/media` → `./media` → `/opt/runtipi/media` (relative path)
   - All paths resolve to subdirectories under `/opt/runtipi/`

**Result:** All data is stored in flat `/opt/runtipi/` subdirectories:
- `/opt/runtipi/app-data/`
- `/opt/runtipi/media/`
- `/opt/runtipi/state/`
- `/opt/runtipi/repos/`
- etc.

This exactly matches the official runtipi-cli behavior, minimizing friction with upstream.

## Workflow Integration

### GitHub Actions

When implemented, workflows should:
1. **build.yml**: Build packages on every push
2. **release.yml**: Create releases with built packages
3. **update-runtipi-version.yml**: Check for new Runtipi releases (nightly)

### Automated Version Updates

A nightly workflow can check https://api.github.com/repos/runtipi/runtipi/releases/latest and create a PR when new versions are available.

## Related Documentation

- **Runtipi upstream**: https://github.com/runtipi/runtipi
- **Runtipi documentation**: https://runtipi.io/docs
- **Installation script**: https://github.com/runtipi/runtipi/blob/master/scripts/install.sh
