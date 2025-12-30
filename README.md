# Samba NAS

A Samba-based NAS (Network Attached Storage) configuration for exposing storage over SMB/CIFS on your home network.

## Overview

This repository contains a Samba NAS solution designed to share storage across different operating systems over a network. It provides a simple way to expose your local storage via the SMB (Server Message Block) protocol, making it accessible from Windows, macOS, and Linux devices.

**Note:** This repository previously contained Frigate-related content, which has been deprecated. The repository now focuses exclusively on the Samba NAS configuration.

## Features

- **Cross-platform file sharing** - Access your files from Windows, macOS, and Linux
- **SMB/CIFS protocol** - Industry-standard network file sharing
- **Home Assistant integration** - Designed as a Home Assistant add-on
- **Flexible configuration** - Support for multiple shares and user access controls
- **External disk support** - Mount and share additional storage devices
- **Network discovery** - WSDD (Web Services Dynamic Discovery) support for easy network browsing
- **Recycle bin** - Optional trash/recycle bin functionality
- **Access control** - Fine-grained user and share permissions

## Requirements

- **Home Assistant OS (HAOS)** - This addon is specifically designed for Home Assistant Operating System
- Network connectivity to your home network
- (Optional) External storage devices if you want to share additional disks

**Important:** This add-on has been designed, built, and tested to work with HAOS (Home Assistant Operating System). Use on other types of installations is not recommended.

## Setup

### Installation

1. Navigate to your Home Assistant frontend
2. Go to **Supervisor** → **Add-on Store**
3. Find the "Samba NAS" add-on
4. Click **INSTALL**

### Basic Configuration

1. In the configuration section, set a username and password
2. Review the enabled shares and disable any you don't plan to use
3. Configure network access by specifying allowed hosts
4. (Optional) Add additional disks to mount and share

For detailed configuration options and examples, see the [sambanas/DOCS.md](sambanas/DOCS.md) file.

## Connecting to the NAS

### From Windows

Use the UNC path format:
```
\\<IP_ADDRESS>\
```

Example: `\\192.168.1.100\`

### From macOS

Use the SMB URL format:
```
smb://<IP_ADDRESS>
```

Example: `smb://192.168.1.100`

### From Linux

Use your file manager's "Connect to Server" option or mount via command line:
```bash
mount -t cifs //<IP_ADDRESS>/<share_name> /mnt/point -o username=<username>
```

### Available Shares

By default, the following directories are exposed:

| Share | Description |
|-------|-------------|
| `config` | Home Assistant configuration files |
| `addons` | Local add-ons |
| `backup` | Backup/snapshot files |
| `media` | Local media files |
| `share` | Shared data between add-ons and Home Assistant |
| `ssl` | SSL certificates |
| `addon_configs` | Add-on configuration directories |

## Maintenance

### Maintenance Mode Notice

The Samba NAS add-on is currently in **maintenance mode**. This means:
- No new features will be implemented
- Critical bug fixes will continue to be provided
- Development focus has shifted to SambaNas2 (a complete rewrite in Go)

For more information about the future of this project, see the [sambanas/README.md](sambanas/README.md) file.

### Updating

The add-on can be updated through the Home Assistant Supervisor interface when updates become available.

## Documentation

Detailed documentation, configuration examples, and advanced features can be found in the `sambanas/` directory:

- [Full Documentation](sambanas/DOCS.md) - Comprehensive guide and configuration reference
- [Configuration File](sambanas/config.yaml) - Add-on configuration schema
- [Changelog](sambanas/CHANGELOG.md) - Version history and changes

## License

This project maintains the license from the original source. Please refer to the specific license file in the repository for details.

## Support

For issues, questions, or contributions, please use the GitHub issue tracker for this repository.

---

**Historical Note:** This repository was previously named to include Frigate-related content. All Frigate functionality has been removed, and the repository now exclusively contains the Samba NAS configuration and add-on files.
