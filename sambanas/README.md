# Home Assistant Add-on: Samba NAS for Frigate

Frigate-optimized Samba NAS for Home Assistant OS - store recordings on internal or NAS-mounted storage with automatic /media/frigate handling.

![Supports aarch64 Architecture][aarch64-shield] ![Supports amd64 Architecture][amd64-shield] ![Supports armv7 Architecture][armv7-shield]


## About

**Samba NAS for Frigate** is a Home Assistant add-on optimized for storing Frigate NVR recordings on internal drives or NAS-mounted storage in Home Assistant OS (HAOS). This add-on provides SMB/CIFS file sharing that works seamlessly with Frigate's storage requirements.

### Key Features for Frigate Users

- **Flexible Storage Options**: Supports both internal drives (SSD, HDD, NVMe) and NAS-mounted storage approaches
- **Automatic /media/frigate Handling**: When a disk labeled `FRIGATE` (uppercase) is mounted at `/media/FRIGATE`, the add-on automatically creates a symlink `/media/frigate → /media/FRIGATE` for compatibility. This symlink is **only created when safe** - the add-on will never overwrite an existing `/media/frigate` directory or file.
- **Frigate Integration**: Pre-configured to work with Frigate's storage requirements, supporting paths like `/media/frigate` or `/media/frigate_media`
- **Auto-mount Internal Drives**: Automatically mount and share labeled internal drives with the `moredisks` configuration option
- **Cross-platform Access**: Access your Frigate recordings from Windows, macOS, and Linux devices over SMB/CIFS
- **Media Library Integration**: Optional integration with Home Assistant's media system for easy access to recordings

### How It Works

This add-on can be used in two primary ways for Frigate storage:

1. **Internal Drive Approach**: Format and label an internal drive (e.g., as `frigate`), configure the add-on to auto-mount it, then add it as network storage in Home Assistant for Frigate to use
2. **NAS-Mounted Approach**: Add a CIFS/SMB network storage mount in Home Assistant pointing to external NAS storage, and configure Frigate to use that mounted path

The add-on handles HAOS mount-name quirks by automatically creating a `/media/frigate → /media/FRIGATE` symlink when safe, ensuring Frigate can consistently access storage regardless of whether HAOS creates uppercase or lowercase mount names.

For detailed setup instructions, see the repository's [FRIGATE_HAOS_INTERNAL_DRIVE.md](https://github.com/phdindota/samba-nas-frigate/blob/master/FRIGATE_HAOS_INTERNAL_DRIVE.md) guide.


[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armhf-shield]: https://img.shields.io/badge/armhf-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[issue]: https://github.com/phdindota/samba-nas-frigate/issues
[repository]: https://github.com/phdindota/samba-nas-frigate
