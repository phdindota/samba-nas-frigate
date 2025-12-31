# Using Frigate with an Internal Drive on Home Assistant OS

This guide explains how to configure the Samba NAS add-on to use an **internal drive** on your Home Assistant OS (HAOS) server for storing Frigate recordings, clips, and snapshots.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step 1: Prepare the Internal Drive](#step-1-prepare-the-internal-drive)
- [Step 2: Configure the Samba NAS Add-on](#step-2-configure-the-samba-nas-add-on)
- [Step 3: Add Network Storage in Home Assistant](#step-3-add-network-storage-in-home-assistant)
- [Step 4: Configure Frigate](#step-4-configure-frigate)
- [Verification and Troubleshooting](#verification-and-troubleshooting)
- [Advanced Configuration](#advanced-configuration)

---

## Overview

This setup allows you to:

- Use an **internal drive** (SSD, HDD, or NVMe) in your HAOS server for Frigate storage
- Automatically mount and share the drive via the Samba NAS add-on
- Access the storage from Frigate running in a Home Assistant add-on or Docker container
- Manage large video files efficiently without impacting your system storage

**Note:** This guide focuses on internal drives. It does **not** cover USB drives or external NAS solutions, though the configuration can be adapted for those use cases.

---

## Prerequisites

Before you begin, ensure you have:

1. **Home Assistant OS (HAOS)** installed and running
   - ⚠️ This configuration is designed specifically for HAOS
   - Other installation types (Container, Core, Supervised) may require different approaches

2. **An internal drive** installed in your HAOS server
   - SSD, HDD, or NVMe drive physically connected to the server
   - The drive should be dedicated to Frigate storage (or at least have a dedicated partition)

3. **Samba NAS add-on** installed from the Home Assistant Add-on Store
   - Version 12.5.0-nas or later recommended

4. **Frigate add-on** or Frigate container ready to be configured

5. **SSH access** to your HAOS server (for drive preparation)
   - You can use the "Advanced SSH & Web Terminal" add-on or the built-in SSH on port 22222

---

## Step 1: Prepare the Internal Drive

You need to format the internal drive with a supported filesystem and give it a recognizable label.

### 1.1 Access the HAOS Host

There are two main ways to access the HAOS host:

**Option A: Using Advanced SSH & Web Terminal Add-on**

1. Install the "Advanced SSH & Web Terminal" add-on from the Add-on Store
2. In the add-on configuration, enable "Protection mode: OFF"
3. Start the add-on and connect via the Web UI or SSH

**Option B: Using the built-in SSH on port 22222**

1. Follow the [HAOS Developer Documentation](https://developers.home-assistant.io/docs/operating-system/debugging/#home-assistant-operating-system) to enable SSH access
2. Connect via SSH: `ssh root@<HAOS_IP> -p 22222`

### 1.2 Identify Your Drive

Once connected to the HAOS host, identify your internal drive:

```bash
# List all block devices
lsblk

# Or use fdisk
fdisk -l
```

Look for your internal drive. It will typically be named `/dev/sda`, `/dev/nvme0n1`, or similar. Make sure you identify the correct drive to avoid data loss!

**Example output:**
```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0 238.5G  0 disk 
└─sda1        8:1    0 238.5G  0 part 
nvme0n1     259:0    0 465.8G  0 disk 
├─nvme0n1p1 259:1    0    32G  0 part 
└─nvme0n1p2 259:2    0 433.8G  0 part /mnt/data
```

In this example, `/dev/sda` is an unused internal SSD that we'll use for Frigate.

### 1.3 Format and Label the Drive

⚠️ **WARNING:** This will erase all data on the drive! Make sure you've selected the correct device.

**Recommended filesystem:** ext4 (best compatibility with Linux/HAOS)

```bash
# Create a partition table (if needed)
# Skip this if your drive already has a partition
parted /dev/sda mklabel gpt
parted /dev/sda mkpart primary ext4 0% 100%

# Format the partition as ext4
mkfs.ext4 /dev/sda1

# Label the partition as 'frigate'
e2label /dev/sda1 frigate

# Verify the label
lsblk -f
```

**Expected output:**
```
NAME   FSTYPE LABEL   UUID                                 MOUNTPOINT
sda                                                         
└─sda1 ext4   frigate 12345678-1234-1234-1234-123456789abc 
```

### 1.4 Alternative: Using an Existing Partition

If you have an existing partition you want to use, you can simply change its label:

```bash
# For ext2/ext3/ext4 filesystems
e2label /dev/sda1 frigate

# For other filesystems:
# vfat/msdos: fatlabel /dev/sda1 frigate
# exfat: exfatlabel /dev/sda1 frigate
# ntfs: ntfslabel /dev/sda1 frigate
# xfs: xfs_admin -L frigate /dev/sda1
```

### 1.5 Supported Filesystems

The Samba NAS add-on supports the following filesystems for `moredisks`:

| Filesystem | ACL Support | Time Machine | Notes |
|------------|-------------|--------------|-------|
| ext2/ext3/ext4 | ✅ Yes | ✅ Yes | **Recommended** - Best for Linux/HAOS |
| btrfs | ✅ Yes | ✅ Yes | Advanced features, snapshots |
| xfs | ✅ Yes | ✅ Yes | High performance for large files |
| squashfs | ✅ Yes | ✅ Yes | Read-only compressed filesystem |
| vfat (FAT32) | ❌ No | ❌ No | Simple, cross-platform |
| msdos (FAT16) | ❌ No | ❌ No | Legacy, limited size |
| f2fs | ❌ No | ❌ No | Flash-optimized |
| exFAT | ❌ No | ❌ No | Experimental, large file support |
| ntfs | ❌ No | ❌ No | Experimental (ntfs3 driver) |
| apfs | ❌ No | ❌ No | Very experimental, read-only, ID only |

**Recommendation:** Use **ext4** for the best balance of compatibility, performance, and reliability with HAOS.

---

## Step 2: Configure the Samba NAS Add-on

### 2.1 Disable Protection Mode

The `automount` and `moredisks` features require Protection Mode to be disabled:

1. Go to **Settings** → **Add-ons** → **Samba NAS**
2. Click on the **Configuration** tab
3. Toggle **Protection mode** to **OFF**
4. Click **Save**

⚠️ **Security Note:** Disabling Protection Mode gives the add-on more system access. Only do this if you trust the add-on and understand the implications.

### 2.2 Apply the Example Configuration

Use the provided example configuration as a starting point:

1. Open the example configuration file: [`sambanas/haos_samba_nas_frigate_internal_example.yaml`](sambanas/haos_samba_nas_frigate_internal_example.yaml)

2. Copy the relevant sections to your Samba NAS add-on configuration:

   **In Settings → Add-ons → Samba NAS → Configuration:**

   ```yaml
   workgroup: WORKGROUP
   username: homeassistant
   password: "YOUR_SECURE_PASSWORD_HERE"
   local_master: true
   allow_hosts:
     - 10.0.0.0/8
     - 172.16.0.0/12
     - 192.168.0.0/16
     - 169.254.0.0/16
     - fe80::/10
     - fc00::/7
   automount: true
   moredisks:
     - frigate
   mountoptions:
     - nosuid
     - relatime
     - noexec
   available_disks_log: true
   medialibrary:
     enable: true
   veto_files:
     - "._*"
     - ".DS_Store"
     - Thumbs.db
     - icon?
     - ".Trashes"
   compatibility_mode: false
   recyle_bin_enabled: false
   wsdd: true
   wsdd2: false
   mqtt_nexgen_entities: false
   autodiscovery: {}
   other_users: []
   acl: []
   interfaces: []
   ```

3. **Important:** Replace `YOUR_SECURE_PASSWORD_HERE` with a strong password

4. Adjust `allow_hosts` if needed to match your network configuration

5. Click **Save**

### 2.3 Start the Add-on

1. Go to the **Info** tab
2. Click **Start**
3. Enable **Start on boot** if you want the share to be available automatically
4. Enable **Watchdog** for automatic restart on failure

### 2.4 Verify the Drive is Mounted

Check the add-on logs to confirm the drive was detected and mounted:

1. Go to the **Log** tab
2. Look for messages like:

   ```
   [INFO] Available disks: frigate
   [INFO] Mounting frigate...
   [INFO] Successfully mounted frigate at /mnt/frigate
   ```

If you see errors, double-check:
- The drive label is exactly `frigate` (case-sensitive)
- Protection Mode is disabled
- The drive is properly formatted and connected

### 2.5 Automatic Symlink Creation

**New Feature:** If you label your drive as `FRIGATE` (uppercase) instead of `frigate` (lowercase), the add-on will automatically:

1. Create a symlink from `/media/frigate` → `/media/FRIGATE` when:
   - `/media/FRIGATE` exists and is a directory
   - `/media/frigate` does not already exist (to prevent data loss)

2. Create the standard Frigate directory structure:
   - `/media/FRIGATE/recordings`
   - `/media/FRIGATE/clips`
   - `/media/FRIGATE/snapshots`

This automatic symlink creation resolves the common case where HAOS mounts a disk labeled `FRIGATE` as `/media/FRIGATE`, but Frigate and various guides reference `/media/frigate`. The add-on logs will show:

```
[INFO] Creating symlink: /media/frigate -> /media/FRIGATE
[INFO] Symlink created successfully
[INFO] Created directory: /media/FRIGATE/recordings
[INFO] Created directory: /media/FRIGATE/clips
[INFO] Created directory: /media/FRIGATE/snapshots
```

**Note:** The add-on will never delete or overwrite an existing `/media/frigate` to prevent accidental data loss.

---

## Step 3: Add Network Storage in Home Assistant

Now that the drive is shared via Samba, add it as network storage in Home Assistant:

### 3.1 Access Storage Settings

1. In Home Assistant, go to **Settings** → **System** → **Storage**
2. Scroll down to **Network storage**
3. Click **Add Network Storage**

### 3.2 Configure the Network Storage

Fill in the form as follows:

| Field | Value | Notes |
|-------|-------|-------|
| **Name** | `frigate_media` | Or any name you prefer |
| **Usage** | Media | Select "Media" for Frigate recordings |
| **Server** | `core` or `<HAOS_IP>` | Use `core` if on the same machine, otherwise the HAOS IP address |
| **Protocol** | CIFS | Samba/SMB protocol |
| **Remote share** | `frigate` | Must match the disk label |
| **Username** | `homeassistant` | As configured in Samba NAS add-on |
| **Password** | `<your password>` | As configured in Samba NAS add-on |

### 3.3 Confirm and Mount

1. Click **Submit**
2. Home Assistant will attempt to mount the share
3. If successful, you'll see the new storage listed under **Network storage**

**Mounted path:** The share will be accessible at `/media/frigate_media` (or whatever name you chose) within Home Assistant containers.

---

## Step 4: Configure Frigate

### 4.1 Verify the Mount in Frigate

First, verify that Frigate can access the mounted storage:

1. Go to the Frigate add-on (or container) configuration
2. In the Frigate add-on **Configuration** tab, ensure the `/media` path is mounted (this should be automatic)
3. Check the Frigate logs to see available storage

### 4.2 Update Frigate Configuration

Edit your Frigate `configuration.yaml` to use the internal drive for recordings:

**Location:** Usually in `/config/frigate.yaml` or accessible via the Frigate add-on configuration editor.

```yaml
# Frigate configuration

# Database settings (optional but recommended for better performance)
database:
  path: /media/frigate_media/frigate.db

# Recording configuration
record:
  enabled: true
  retain:
    days: 7              # Keep recordings for 7 days
    mode: motion         # Record when motion is detected
  events:
    retain:
      default: 30        # Keep event clips for 30 days
      mode: active_objects

# Snapshots configuration
snapshots:
  enabled: true
  retain:
    default: 30          # Keep snapshots for 30 days

# Optional: Explicitly set the media directory
# (Usually not needed if using the default /media/<mount_name>)
# ffmpeg:
#   output_args:
#     record: preset-record-generic-audio-copy

# Camera configuration (example)
cameras:
  front_door:
    ffmpeg:
      inputs:
        - path: rtsp://username:password@camera-ip:554/stream
          roles:
            - detect
            - record
    detect:
      width: 1920
      height: 1080
      fps: 5
    record:
      enabled: true
      retain:
        days: 14
      events:
        retain:
          default: 30
    snapshots:
      enabled: true
```

### 4.3 Key Configuration Points

1. **Database path:** Store the Frigate database on the internal drive for better performance
   ```yaml
   database:
     path: /media/frigate_media/frigate.db
   ```

2. **Retention settings:** Adjust based on your storage capacity
   ```yaml
   record:
     retain:
       days: 7  # Adjust based on your needs and drive size
   ```

3. **Motion vs. continuous recording:**
   - `mode: motion` - Only record when motion is detected (saves space)
   - `mode: all` - Continuous 24/7 recording (requires more storage)

### 4.4 Restart Frigate

1. Save the configuration changes
2. Restart the Frigate add-on or container
3. Check the Frigate logs for any errors

### 4.5 Verify Storage Usage

You can monitor storage usage in several ways:

1. **Frigate UI:** Go to the Frigate web interface → Storage tab
2. **Home Assistant:** Settings → System → Storage → View the network storage
3. **Terminal:** SSH into HAOS and run:
   ```bash
   df -h | grep frigate
   ```

---

## Verification and Troubleshooting

### Verify the Complete Setup

1. **Check Samba share is accessible:**
   - From a Windows PC: `\\<HAOS_IP>\frigate`
   - From macOS: `smb://<HAOS_IP>/frigate`
   - You should be able to browse and write files

2. **Check Home Assistant mount:**
   ```bash
   # From Home Assistant SSH terminal
   ls -la /media/frigate_media
   ```
   You should see the contents of your internal drive

3. **Check Frigate is writing to the drive:**
   - Trigger a recording event (e.g., walk in front of a camera)
   - Check the Frigate UI for recordings
   - Verify files are being created in `/media/frigate_media/recordings/`

### Common Issues and Solutions

#### Issue: "Disk not found" or "Failed to mount"

**Possible causes:**
- Incorrect disk label (case-sensitive)
- Protection Mode is still enabled
- Filesystem not supported
- Drive not properly formatted

**Solutions:**
1. Verify the label: `lsblk -f` should show `LABEL="frigate"`
2. Check Samba NAS add-on logs for specific error messages
3. Enable `available_disks_log: true` to see all detected disks
4. Ensure Protection Mode is OFF in add-on settings

#### Issue: "Permission denied" when Frigate tries to write

**Possible causes:**
- Incorrect mount options
- User permissions mismatch

**Solutions:**
1. Check mount options - avoid `ro` (read-only)
2. Ensure `noexec` doesn't prevent necessary operations (usually fine for data storage)
3. Check the ACL configuration in the Samba NAS add-on
4. Verify the username/password match in both add-ons

#### Issue: "Media folder is empty" in Media Browser

**Possible causes:**
- Home Assistant started before the add-on
- Mount point not properly registered

**Solutions:**
1. Restart Home Assistant: **Settings** → **System** → **Restart**
2. Check that `medialibrary.enable: true` in Samba NAS config
3. Wait a few minutes after add-on starts before restarting HA

#### Issue: Poor performance or slow writes

**Possible causes:**
- Network bottleneck
- Slow internal drive
- Inefficient mount options

**Solutions:**
1. Consider using a faster filesystem (ext4, xfs, btrfs)
2. Use an SSD instead of HDD for better random I/O performance
3. Check network speed if accessing remotely
4. For direct access within HAOS, consider alternative mounting methods

#### Issue: Network storage not showing up in HA

**Possible causes:**
- Samba share not running
- Incorrect share name
- Firewall blocking SMB ports

**Solutions:**
1. Verify Samba NAS add-on is running and started successfully
2. Check the share name matches the disk label exactly
3. Verify allow_hosts includes your HAOS IP
4. Test access from another device on the network

---

## Advanced Configuration

### Using a Specific Partition UUID Instead of Label

If you prefer to use the partition UUID instead of a label:

```yaml
moredisks:
  - "id:12345678-1234-1234-1234-123456789abc"
```

Find the UUID with:
```bash
lsblk -f
# or
blkid /dev/sda1
```

### Creating Multiple Shares from One Drive

You can expose different folders as separate shares using ACL:

```yaml
acl:
  - share: frigate_recordings
    disabled: false
    users:
      - homeassistant
    timemachine: false
    usage:
      - media
  - share: frigate_snapshots
    disabled: false
    users:
      - homeassistant
    ro_users:
      - viewer
    timemachine: false
```

### Read-Only Access for Additional Users

To create a read-only user for viewing recordings:

```yaml
other_users:
  - username: viewer
    password: "viewer_password"

acl:
  - share: frigate
    users:
      - homeassistant  # Full access
    ro_users:
      - viewer         # Read-only access
```

### Using RAID or Multiple Drives

For redundancy, you can use multiple drives in a RAID configuration:

1. Set up software RAID at the HAOS host level (advanced)
2. Label the RAID array `frigate`
3. Configure Samba NAS to mount and share as normal

**Note:** RAID configuration is beyond the scope of this guide and requires advanced Linux knowledge.

### Monitoring Disk Health with SMART

The Samba NAS add-on can monitor drive health using SMART:

```yaml
enable_smart: true  # Default is true
```

Check logs for SMART warnings about drive health.

### Disk Spin-Down for Power Saving

If you want your drive to spin down when idle:

```yaml
hdd_idle_seconds: 600  # Spin down after 10 minutes of inactivity
```

**Note:** This may cause delays when Frigate needs to write suddenly. Consider keeping this disabled for active recording drives.

### Using Multiple Internal Drives

You can mount multiple drives for different purposes:

```yaml
moredisks:
  - frigate_live     # For live recordings
  - frigate_archive  # For long-term storage
  - frigate_snapshots # For snapshot storage
```

Then configure Frigate to use different paths for each purpose.

---

## Best Practices

1. **Use ext4 filesystem:** Best compatibility and performance with HAOS
2. **Dedicated drive:** Use a dedicated drive (or partition) for Frigate to avoid I/O conflicts
3. **SSD for database:** If possible, keep the Frigate database on an SSD for better performance
4. **Regular backups:** Back up your Frigate configuration and important recordings
5. **Monitor storage:** Set up alerts for low disk space
6. **Test before deployment:** Test the setup thoroughly before relying on it for critical recordings
7. **Security:** Use strong passwords and restrict network access appropriately

---

## Additional Resources

- [Samba NAS Add-on Documentation](sambanas/DOCS.md)
- [Frigate Documentation](https://docs.frigate.video/)
- [Home Assistant Storage Documentation](https://www.home-assistant.io/common-tasks/os/#network-storage)
- [HAOS Developer Documentation](https://developers.home-assistant.io/docs/operating-system/)

---

## Support

If you encounter issues:

1. Check the Samba NAS add-on logs for error messages
2. Check the Frigate logs for recording errors
3. Review this documentation and the troubleshooting section
4. Search or post in the Home Assistant Community forums
5. Open an issue on the [GitHub repository](https://github.com/phdindota/samba-nas-frigate/issues)

---

**Last Updated:** December 2024  
**Tested with:** Home Assistant OS 11.x, Samba NAS 12.5.0-nas, Frigate 0.13.x
