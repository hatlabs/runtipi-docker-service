⚠️ **THESE RULES ONLY APPLY TO FILES IN /runtipi-docker-service/** ⚠️

# Runtipi Docker Service - Development Guide

## 🎯 For Agentic Coding: Use the HaLOS Workspace

This repository should be used as part of the halos-distro workspace for AI-assisted development:

```bash
# Clone workspace and all repos
git clone https://github.com/hatlabs/halos-distro.git
cd halos-distro
./run repos:clone
```

See `halos-distro/docs/` for development workflows and guidance.

## About This Project

Debian package bundling Runtipi as a Docker Compose stack with systemd management.

**Local Instructions**: For environment-specific instructions and configurations, see @CLAUDE.local.md (not committed to version control).

## Git Workflow Policy

**IMPORTANT:** Always ask before pushing, creating/pushing tags, or running destructive git operations that affect remote repositories. Local commits and branch operations are fine.

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
