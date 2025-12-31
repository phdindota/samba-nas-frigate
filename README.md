# Samba NAS for Frigate - Home Assistant Add-on Repository

A Home Assistant add-on repository providing a Samba NAS solution optimized for Frigate NVR storage on internal drives or NAS-mounted storage. This add-on exposes storage over SMB/CIFS, making it accessible from Windows, macOS, and Linux devices.

## Overview

This is a **Home Assistant add-on repository** that contains the Samba NAS add-on, specifically configured to work seamlessly with Frigate NVR on Home Assistant OS (HAOS). The add-on provides automated mounting and sharing of internal drives or NAS-mounted storage for efficient storage of Frigate recordings, clips, and snapshots.

The add-on also handles HAOS mount-name quirks by automatically creating a `/media/frigate → /media/FRIGATE` symlink when safe, ensuring Frigate can consistently access storage regardless of whether HAOS creates uppercase or lowercase mount names.

## Features

- **Cross-platform file sharing** - Access your files from Windows, macOS, and Linux
- **SMB/CIFS protocol** - Industry-standard network file sharing
- **Home Assistant integration** - Designed as a Home Assistant add-on
- **Flexible configuration** - Support for multiple shares and user access controls
- **Flexible storage options** - Supports both internal drives (SSD, HDD, NVMe) and NAS-mounted storage approaches
- **Auto-mount internal drives** - Automatically mount and share labeled internal drives
- **Automatic /media/frigate handling** - Auto-creates `/media/frigate → /media/FRIGATE` symlink when safe to handle HAOS mount-name quirks
- **External disk support** - Mount and share additional storage devices
- **Network discovery** - WSDD (Web Services Dynamic Discovery) support for easy network browsing
- **Recycle bin** - Optional trash/recycle bin functionality
- **Access control** - Fine-grained user and share permissions
- **Frigate optimization** - Pre-configured for Frigate NVR storage with proper directory structure
- **Media library integration** - Seamless integration with Home Assistant's media system

## Requirements

- **Home Assistant OS (HAOS)** - This add-on is specifically designed for Home Assistant Operating System
- Network connectivity to your home network
- (Optional) An internal drive (SSD, HDD, or NVMe) for Frigate storage
- (Optional) External storage devices if you want to share additional disks

**Important:** This add-on has been designed, built, and tested to work with HAOS (Home Assistant Operating System). Use on other types of installations is not recommended.

## Installation

### Adding the Repository

This is a **custom add-on repository**. To use it, you need to add it to your Home Assistant installation:

1. Navigate to your Home Assistant frontend
2. Go to **Settings** → **Add-ons** → **Add-on Store**
3. Click the **three dots menu** (⋮) in the top right corner
4. Select **Repositories**
5. Add the following URL:
   ```
   https://github.com/phdindota/samba-nas-frigate
   ```
6. Click **Add**

The repository will now appear in your add-on store, and you can install the Samba NAS add-on.

### Installing the Add-on

After adding the repository:

1. Find **Samba NAS** in the add-on store (it may be in the "Community Add-ons" section)
2. Click on the add-on
3. Click **INSTALL**
4. Wait for the installation to complete (the add-on will be built locally from source, which may take a few minutes)

**Note:** This add-on builds locally on your Home Assistant system from the included Dockerfile. The first installation may take 5-10 minutes depending on your hardware.

## Configuration

### Quick Start for Frigate with Internal Drive

If you're setting up this add-on to use with Frigate on an internal drive:

1. **Prepare your internal drive** (see [FRIGATE_HAOS_INTERNAL_DRIVE.md](FRIGATE_HAOS_INTERNAL_DRIVE.md) for detailed instructions)
   - Format the drive with a supported filesystem (ext4, btrfs, xfs, etc.)
   - Label the drive (e.g., `frigate`)

2. **Configure the add-on** with these recommended settings:
   ```yaml
   workgroup: WORKGROUP
   username: homeassistant
   password: changeme  # Change this to a strong password!
   automount: true
   moredisks:
     - frigate
   medialibrary:
     enable: true
   ```
   See [sambanas/haos_samba_nas_frigate_internal_example.yaml](sambanas/haos_samba_nas_frigate_internal_example.yaml) for a complete example configuration.

3. **Disable Protection Mode** in the add-on settings (required for automount and moredisks features)

4. **Start the add-on**

### Basic Configuration

For general use (not Frigate-specific):

1. In the configuration section, set a username and password
2. Review the enabled shares and disable any you don't plan to use
3. Configure network access by specifying allowed hosts
4. (Optional) Add additional disks to mount and share

For detailed configuration options and examples, see the [sambanas/DOCS.md](sambanas/DOCS.md) file.

## Connecting to the NAS

Once the add-on is running and configured:

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

### Adding as Network Storage in Home Assistant

For Frigate integration, you should add the Samba share as network storage:

1. Go to **Settings** → **System** → **Storage**
2. Click **Add Network Storage**
3. Configure:
   - **Type**: CIFS (Samba)
   - **Server**: `core` (or your HAOS hostname/IP)
   - **Share**: `frigate` (or the name of your mounted disk)
   - **Username**: `homeassistant` (or as configured)
   - **Password**: (your configured password)
   - **Media directory path**: Leave default (e.g., `frigate_media`)
4. Click **Submit**

The share will be mounted at `/media/<directory_path>` and accessible to all add-ons including Frigate.

### Available Shares

The add-on exposes multiple shares by default:

| Share | Description |
|-------|-------------|
| `config` | Home Assistant configuration files |
| `addons` | Local add-ons |
| `backup` | Backup/snapshot files |
| `media` | Local media files |
| `share` | Shared data between add-ons and Home Assistant |
| `ssl` | SSL certificates |
| `addon_configs` | Add-on configuration directories |

**Additional shares** will be automatically created for any drives listed in the `moredisks` configuration option. For example, if you have a drive labeled `frigate`, it will be shared as `\\<IP_ADDRESS>\frigate`.

## Frigate Integration

This add-on is optimized for use with Frigate NVR. Here's how to set it up:

### Step 1: Prepare Internal Drive

Follow the comprehensive guide in [FRIGATE_HAOS_INTERNAL_DRIVE.md](FRIGATE_HAOS_INTERNAL_DRIVE.md) to:
- Format and label your internal drive (e.g., as `frigate`)
- Understand the drive preparation requirements
- Set up proper filesystem and permissions

### Step 2: Configure Samba NAS Add-on

Use the example configuration provided in [sambanas/haos_samba_nas_frigate_internal_example.yaml](sambanas/haos_samba_nas_frigate_internal_example.yaml):

```yaml
workgroup: WORKGROUP
username: homeassistant
password: changeme  # Replace with a strong password!
automount: true
moredisks:
  - frigate
medialibrary:
  enable: true
```

**Important:** Disable Protection Mode in the add-on settings to enable automount functionality.

### Step 3: Add Network Storage

In Home Assistant:
1. **Settings** → **System** → **Storage** → **Add Network Storage**
2. Configure the CIFS share pointing to your `frigate` share
3. The share will be mounted at `/media/<mount_name>`

### Step 4: Configure Frigate

Update your Frigate configuration to use the mounted path for storage:

```yaml
# In your Frigate configuration
mqtt:
  enabled: true

ffmpeg:
  hwaccel_args: preset-rpi-64-h264  # Adjust for your hardware

record:
  enabled: true
  retain:
    days: 7
    mode: motion

snapshots:
  enabled: true
  retain:
    default: 30

# If needed, explicitly set the base directory (optional in most cases):
# media:
#   base_dir: /media/frigate
```

Frigate will automatically use the `/media/<mount_name>` path for recordings, clips, and snapshots.

### Automatic FRIGATE/frigate Symlink

**Important:** If you have a disk or partition labeled `FRIGATE` (uppercase) that gets mounted as `/media/FRIGATE`, this add-on will automatically create a symlink from `/media/frigate` (lowercase) to `/media/FRIGATE` when:

- `/media/FRIGATE` exists and is a directory
- `/media/frigate` does not already exist (to prevent data loss)

This symlink creation happens automatically during add-on startup, ensuring both Samba and Frigate can access the storage using either `/media/FRIGATE` or `/media/frigate` paths. The add-on will also automatically create the standard Frigate directory structure:
- `/media/FRIGATE/recordings`
- `/media/FRIGATE/clips`
- `/media/FRIGATE/snapshots`

**Note:** The add-on will never delete or overwrite an existing `/media/frigate` directory or file to prevent accidental data loss.

For complete step-by-step instructions, see [FRIGATE_HAOS_INTERNAL_DRIVE.md](FRIGATE_HAOS_INTERNAL_DRIVE.md).

## Updating

The add-on can be updated through the Home Assistant Supervisor interface when updates become available.

## Documentation

Detailed documentation, configuration examples, and advanced features can be found in the `sambanas/` directory:

- **[Frigate with Internal Drive Setup](FRIGATE_HAOS_INTERNAL_DRIVE.md)** - Complete guide for using Frigate NVR with an internal drive on Home Assistant OS
  - How to prepare and label an internal drive
  - Auto-mount configuration with the Samba NAS add-on
  - Integration with Frigate for video recording storage
  - Troubleshooting and best practices
- **[Example Configuration for Frigate](sambanas/haos_samba_nas_frigate_internal_example.yaml)** - Ready-to-use configuration template for Frigate storage
- **[Full Documentation](sambanas/DOCS.md)** - Comprehensive guide and configuration reference
- **[Changelog](sambanas/CHANGELOG.md)** - Version history and changes

## Repository Structure

This repository is organized as a Home Assistant add-on repository:

```
/
├── repository.json              # Repository metadata
├── README.md                    # This file
├── FRIGATE_HAOS_INTERNAL_DRIVE.md  # Frigate setup guide
└── sambanas/                    # Samba NAS add-on
    ├── config.yaml              # Add-on configuration schema
    ├── Dockerfile               # Container build instructions
    ├── build.yaml               # Build configuration
    ├── DOCS.md                  # Detailed documentation
    ├── CHANGELOG.md             # Version history
    ├── rootfs/                  # Add-on filesystem
    └── haos_samba_nas_frigate_internal_example.yaml
```

## Support

For issues, questions, or contributions, please use the GitHub issue tracker for this repository:
- **Issues**: https://github.com/phdindota/samba-nas-frigate/issues
- **Discussions**: https://github.com/phdindota/samba-nas-frigate/discussions

## License

This project maintains the license from the original source. Please refer to the specific license file in the repository for details.

---

**About This Repository**: This is a Home Assistant add-on repository containing the Samba NAS add-on, optimized for use with Frigate NVR on internal drives. The add-on is based on the excellent work from the Home Assistant community and has been configured specifically for Frigate media storage use cases.
