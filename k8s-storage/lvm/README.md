# **LVM (Logical Volume Manager)**

## **1. What is LVM?**

LVM is a storage management layer between **physical storage devices** and the **file system**.
It provides flexibility and abstraction, making it possible to:

* **Resize volumes** on the fly without unmounting
* **Combine multiple disks** into one logical pool
* **Take snapshots** of volumes for backups
* **Move data** between disks without downtime
* **Create RAID-like configurations** using LVM
* **Migrate data** between storage devices transparently

Overall, instead of your file system being locked to a single, fixed-size partition, LVM lets you manage storage dynamically.

---

## **2. LVM Key Terms**

| Term                     | Description                                                                                |
| ------------------------ | ------------------------------------------------------------------------------------------ |
| **PV (Physical Volume)** | Actual storage devices (disks or partitions) that LVM can manage.                          |
| **VG (Volume Group)**    | A pool of storage created by combining one or more PVs.                                    |
| **LV (Logical Volume)**  | "Virtual partitions" carved from a VG — these are what you format and mount.               |
| **PE (Physical Extent)** | Smallest allocatable unit in a VG (default: 4 MB). Think of it as the "block size" of LVM. |
| **LE (Logical Extent)**  | Logical unit that maps to one or more PEs in an LV.                                       |
| **COW (Copy-on-Write)**  | Mechanism used by snapshots to track changes efficiently.                                  |

---

## **3. LVM Architecture**

```
[Physical Disk / Partition] → PV → VG → LV → File System

/dev/sdb  → Physical Volume (PV)
/dev/sdc  → Physical Volume (PV)
     ↓
Volume Group "vg_data"
     ↓
Logical Volumes "lv_music", "lv_backup"
     ↓
Format & mount → /mnt/music, /mnt/backup
```

---

## **4. Lab Setup — Checking Available Disks**

```bash
lsblk

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   40G  0 disk 
└─sda1   8:1    0   40G  0 part /
sdb      8:16   0    5G  0 disk 
sdc      8:32   0    5G  0 disk 
sdd      8:48   0    5G  0 disk 
```

* **sda** — Root disk (OS is installed here)
* **sdb, sdc, sdd** — Empty disks to be used for LVM

---

## **5. Creating Physical Volumes (PV)**

```bash
sudo pvcreate /dev/sdb /dev/sdc /dev/sdd
sudo pvs

PV         VG Fmt  Attr PSize PFree
/dev/sdb      lvm2 ---  5.00g 5.00g
/dev/sdc      lvm2 ---  5.00g 5.00g
/dev/sdd      lvm2 ---  5.00g 5.00g
```

**Note:** A PV can be a whole disk (e.g., `/dev/sdb`) or a partition (e.g., `/dev/sdb1`).

---

## **6. Creating a Volume Group (VG)**

```bash
$ sudo vgcreate vg_lab /dev/sdb /dev/sdc /dev/sdd
$ sudo vgs

VG     #PV #LV #SN Attr   VSize   VFree  
vg_lab   3   0   0 wz--n- <14.99g <14.99g
```

* `VSize` — Total size of the VG (sum of PVs)
* `VFree` — Free space available to create LVs

---

## **7. Creating a Logical Volume (LV)**

```bash
sudo lvcreate -L 10G -n lv_data vg_lab
sudo lvs

LV      VG     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
lv_data vg_lab -wi-a----- 10.00g
```

** Verify the LV Status**
```bash

$ sudo lvscan

ACTIVE            '/dev/vg_lab/lv_data' [10.00 GiB] inherit

$ sudo lvdisplay vg_lab/lv_data

  --- Logical volume ---
  LV Path                /dev/vg_lab/lv_data
  LV Name                lv_data
  VG Name                vg_lab
  LV UUID                wG93ln-sdBT-hmkq-Ud0y-FU09-2f8r-vIb7fr
  LV Write Access        read/write
  LV Creation host, time lvm-lab, 2025-08-29 06:15:14 +0000
  LV Status              available
  # open                 0
  LV Size                10.00 GiB
  Current LE             2560
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
   

```


---

## **8. Formatting the LV with a File System**

Check the disktype of the LV and put a filesystem on it.
```bash
$ sudo lvscan
  ACTIVE            '/dev/vg_lab/lv_data' [10.00 GiB] inherit

$ sudo disktype /dev/vg_lab/lv_data

--- /dev/vg_lab/lv_data
Block device, size 10 GiB (10737418240 bytes)
Blank disk/medium

```


Create a filesystem on the LV.
```bash
sudo mkfs.ext4 /dev/vg_lab/lv_data
```

This creates an `ext4` filesystem on the LV.

```bash
sudo disktype /dev/vg_lab/lv_data

--- /dev/vg_lab/lv_data
Block device, size 10 GiB (10737418240 bytes)
Ext4 file system
  UUID 69A4CB59-424D-4972-844A-E9F0D01EC802 (DCE, v4)
  Volume size 10 GiB (10737418240 bytes, 2621440 blocks of 4 KiB)

```

---

## **9. Mounting the Logical Volume**

```bash
sudo mkdir /mnt/data
sudo mount /dev/vg_lab/lv_data /mnt/data
df -h

Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/vg_lab-lv_data  9.8G   24K  9.3G   1% /mnt/data
```
---


## **10. Adding/Removing Storage**

### Extend Volume Group (Add more disks)
```bash
# Add a new disk to existing VG
sudo pvcreate /dev/sde
sudo vgextend vg_lab /dev/sde
sudo vgs  # Check increased VG size
```

### Remove Physical Volume from VG
```bash
# Move data away from PV first
sudo pvmove /dev/sdd
sudo vgreduce vg_lab /dev/sdd
sudo pvremove /dev/sdd
```

---

## **11. Common LVM Operations**

### Resize a Logical Volume

**Extending (Safe):**
```bash
# Increase by 5GB
sudo lvextend -L +5G /dev/vg_lab/lv_data
sudo resize2fs /dev/vg_lab/lv_data  # For ext4/ext3
# Or for XFS: sudo xfs_growfs /mnt/data
```

### Reduce a Logical Volume (Dangerous: backup first)

⚠️ **Warning:** Always backup data before reducing!

```bash
sudo umount /mnt/data
sudo e2fsck -f /dev/vg_lab/lv_data
sudo resize2fs /dev/vg_lab/lv_data 8G
sudo lvreduce -L 8G /dev/vg_lab/lv_data
sudo mount /dev/vg_lab/lv_data /mnt/data
```

### Take a Snapshot

```bash
sudo lvcreate -L 1G -s -n lv_data_snap /dev/vg_lab/lv_data
```

## **12. LVM Snapshots Deep Dive**

### What is an LVM Snapshot?

An LVM snapshot is a "copy-on-write" backup of a logical volume at a specific point in time. It lets you preserve the state of your data, useful for backups or testing.

**Key Points:**
- Snapshots are **not full copies** - they only store differences
- Uses **Copy-on-Write (COW)** mechanism
- Original data remains unchanged until modified
- When original data changes, old data is copied to snapshot space

### How Snapshots Work (COW Mechanism)

```
Original LV: [A][B][C][D]  →  Snapshot created
                              ↓
Original LV: [A][X][C][D]  →  Block B modified
                              ↓  
Snapshot:    [B]           →  Old B stored in snapshot space
```

### How to Take a Snapshot

1. **Create a snapshot LV**  
   The following command creates a 1GB snapshot of `lv_data`:
   ```bash
   sudo lvcreate -L 1G -s -n lv_data_snap /dev/vg_lab/lv_data
   ```
   - `-L 1G`: Size of snapshot (space for COW changes)
   - `-s`: Indicates snapshot
   - `-n lv_data_snap`: Name of snapshot LV
   - `/dev/vg_lab/lv_data`: Source LV

2. **Verify the snapshot**
   ```bash
   sudo lvs
   sudo lvdisplay /dev/vg_lab/lv_data_snap
   ```

3. **Mount the snapshot (optional)**
   ```bash
   sudo mkdir /mnt/data_snap
   sudo mount -o ro /dev/vg_lab/lv_data_snap /mnt/data_snap
   ls /mnt/data_snap
   ```

### Monitoring Snapshot Usage

```bash
# Check snapshot usage
sudo lvs -o +snap_percent
sudo lvdisplay /dev/vg_lab/lv_data_snap | grep "Allocated to snapshot"
```

### How to Restore from a Snapshot

**Method 1: Merge (Destructive - replaces original)**
```bash
sudo umount /mnt/data
sudo lvconvert --merge /dev/vg_lab/lv_data_snap
sudo mount /dev/vg_lab/lv_data /mnt/data
```

**Method 2: Copy data from snapshot (Non-destructive)**
```bash
sudo mount -o ro /dev/vg_lab/lv_data_snap /mnt/data_snap
cp -a /mnt/data_snap/important_file /mnt/data/
```

### Remove a Snapshot

```bash
sudo umount /mnt/data_snap  # If mounted
sudo lvremove /dev/vg_lab/lv_data_snap
```

### Snapshot Best Practices

- **Size appropriately:** Allocate 15-20% of original LV size for active systems
- **Monitor usage:** Snapshots become invalid when full
- **Don't keep long-term:** Snapshots impact performance
- **Use for:** Backups, testing, rollback points

### ⚠️ Important Snapshot Limitations

1. **Snapshot overflow:** If changes exceed snapshot size, snapshot becomes invalid
2. **Performance impact:** COW operations slow down writes to original LV
3. **Not incremental:** Each snapshot is independent
4. **Space overhead:** Metadata overhead even for small changes

### Notes

- **COW Process:** When original data changes, LVM copies the original block to snapshot space before writing new data
- **Snapshot size** should accommodate expected changes, not the full LV size
- **Read-only snapshots** are recommended for backups to prevent accidental modifications
- **Invalid snapshots** cannot be restored - monitor with `lvs -o +snap_percent`

## **13. LVM Stripes and RAID**

### Creating Striped Logical Volume (RAID 0)
```bash
# Stripe across 2 PVs for better performance
sudo lvcreate -L 4G -i2 -I64 -n lv_stripe vg_lab
```

### Creating Mirrored Logical Volume (RAID 1)
```bash
# Mirror across 2 PVs for redundancy
sudo lvcreate -L 4G -m1 -n lv_mirror vg_lab
```

## **14. Troubleshooting Commands**

```bash
# Check LVM configuration
sudo vgck vg_lab
sudo lvmdump

# Activate/Deactivate LVs
sudo lvchange -ay /dev/vg_lab/lv_data    # Activate
sudo lvchange -an /dev/vg_lab/lv_data    # Deactivate

# Scan for LVM volumes
sudo pvscan
sudo vgscan
sudo lvscan

# Display detailed information
sudo pvdisplay
sudo vgdisplay
sudo lvdisplay
```

---

https://www.youtube.com/watch?v=WPO3KiJtG1A

---
