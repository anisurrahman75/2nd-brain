# **LVM (Logical Volume Manager)**

## **1. What is LVM?**

LVM is a storage management layer between **physical storage devices** and the **file system**.
It provides flexibility and abstraction, making it possible to:

* **Resize volumes** on the fly without unmounting
* **Combine multiple disks** into one logical pool
* **Take snapshots** of volumes for backups
* **Move data** between disks without downtime

Overall, instead of your file system being locked to a single, fixed-size partition, LVM lets you manage storage dynamically.

---

## **2. LVM Key Terms**

| Term                     | Description                                                                                |
| ------------------------ | ------------------------------------------------------------------------------------------ |
| **PV (Physical Volume)** | Actual storage devices (disks or partitions) that LVM can manage.                          |
| **VG (Volume Group)**    | A pool of storage created by combining one or more PVs.                                    |
| **LV (Logical Volume)**  | "Virtual partitions" carved from a VG — these are what you format and mount.               |
| **PE (Physical Extent)** | Smallest allocatable unit in a VG (default: 4 MB). Think of it as the "block size" of LVM. |

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
sudo vgcreate vg_lab /dev/sdb /dev/sdc /dev/sdd
sudo vgs

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

---

## **8. Formatting the LV with a File System**

```bash
sudo mkfs.ext4 /dev/vg_lab/lv_data
```

This creates an `ext4` filesystem on the LV.

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


## **11. Common LVM Operations**

### Resize a Logical Volume

```bash
# Increase by 5GB
sudo lvresize -L +5G /dev/vg_lab/lv_data
sudo resize2fs /dev/vg_lab/lv_data
```

### Reduce a Logical Volume (Dangerous: backup first)

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

---

## **12. LVM Workflow Summary**

1. **Create PVs** → `pvcreate`
2. **Create VG** → `vgcreate`
3. **Create LV** → `lvcreate`
4. **Format** → `mkfs`
5. **Mount** → `mount`
6. (Optional) Resize, snapshot, or migrate volumes.

---

