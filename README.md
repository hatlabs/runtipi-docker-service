# runtipi-docker-service

Debian packaging for Runtipi, a homeserver orchestrator for managing Docker containers through a user-friendly web interface. Runtipi provides an app store with 300+ pre-configured applications, making it easy to deploy and manage self-hosted services.

## Overview

Runtipi is a homeserver orchestrator built on Docker Compose. This package provides Debian packaging and systemd integration for easy installation and management on Debian-based systems.

The package includes:
- **docker-compose.yml**: The Runtipi stack (4 containers: reverse proxy, database, queue, and main app)
- **runtipi-cli**: Command-line tool for manual operations and management
- **systemd service**: Automatic startup and lifecycle management via systemd

## Files

- **docker-compose.yml**: Docker Compose configuration for Runtipi stack
- **runtipi-cli**: Command-line interface for manual management
- **runtipi.service**: Systemd service file for automatic lifecycle management
- **VERSION**: Runtipi version to install
- **CLAUDE.md**: Documentation for Claude Code

## Installation

The Runtipi service is installed via a Debian package that:
1. Downloads the architecture-appropriate `runtipi-cli` binary from GitHub releases
2. Downloads the `docker-compose.yml` file for the Runtipi stack
3. Installs both files to `/opt/runtipi/`
4. Installs the systemd service file to `/lib/systemd/system/runtipi.service`
5. Creates the complete directory structure (`media`, `state`, `repos`, `apps`, etc.)
6. Creates a default `.env` configuration file
7. Enables and starts the runtipi service

## Service Management

```bash
# Start Runtipi
sudo systemctl start runtipi

# Stop Runtipi
sudo systemctl stop runtipi

# Restart Runtipi
sudo systemctl restart runtipi

# Check status
sudo systemctl status runtipi

# View logs
sudo journalctl -u runtipi -f
```

## Access

Once running, Runtipi is accessible at:
- **Web UI**: http://device-ip:80 or http://tipi.local

## Data Location

Runtipi stores its data in `/opt/runtipi/`, using a flat directory structure:
- `media/` - Media files shared across apps
- `state/` - Application state and settings
- `repos/` - App store repository cache
- `apps/` - Installed application configurations
- `app-data/` - Application data (organized by store and app)
- `logs/` - Service logs
- `traefik/` - Reverse proxy configuration and SSL certificates
- `user-config/` - User-customized app configurations
- `backups/` - Backup files in .tar.gz format
- `cache/` - Build and runtime cache
- `data/` - Runtipi's own data (PostgreSQL, Redis)
- `.env` - Environment configuration

## Architecture

Runtipi runs as a Docker Compose stack with 4 containers:
- **runtipi-reverse-proxy** (Traefik): Handles HTTP/HTTPS traffic on ports 80/443
- **runtipi-db** (PostgreSQL): Database for Runtipi state
- **runtipi-queue** (LavinMQ): Message queue for background jobs
- **runtipi** (NestJS app): Main application server

**Management:**
- **Systemd service**: Controls the Docker Compose stack (`docker compose up/down`)
- **runtipi-cli**: Available for manual operations and troubleshooting
- **Installation Path**: `/opt/runtipi/`
- **Configuration**: `/opt/runtipi/.env`

## CLI Usage

While systemd manages the service automatically, you can use `runtipi-cli` for manual operations:

```bash
# Manual start (not recommended - use systemctl instead)
cd /opt/runtipi
sudo ./runtipi-cli start

# Manual stop
sudo ./runtipi-cli stop

# Use with custom env file
sudo ./runtipi-cli start --env-file /path/to/.env
```

## Building the Package

### Local Build

```bash
# Build package locally (requires Debian packaging tools)
./run package:deb

# Build using Docker container (recommended)
./run package:deb:docker
```

### CI Build

```bash
# Build in CI mode (uses existing changelog)
./run package:deb:docker:ci
```

## Version Management

The package version is managed in the `VERSION` file. The version should match a valid Runtipi release tag (e.g., `v3.8.1`).

To update to a new version:
1. Update the `VERSION` file with the new tag
2. Update `debian/changelog` with the new version
3. Update `.bumpversion.cfg` if needed
4. Rebuild the package

## Requirements

- Docker CE CLI
- Docker Compose plugin
- 64-bit ARM or x86 architecture

## License

This packaging repository (build scripts, Debian packaging files, systemd service, documentation) is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

**Important:** Runtipi itself is licensed under the **GPL-3.0 License**. When you install this package, you receive:
- The packaging infrastructure (MIT) - created by Hat Labs
- Runtipi software (GPL-3.0) - created by the Runtipi project

Users of the installed software are bound by Runtipi's GPL-3.0 license for the Runtipi application itself.

## Links

- **Runtipi Project**: https://github.com/runtipi/runtipi
- **Runtipi Documentation**: https://runtipi.io/docs
- **This Packaging Repository**: https://github.com/hatlabs/runtipi-docker-service
