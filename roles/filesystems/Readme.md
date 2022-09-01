# Filesystem

## Mounting drives

__DRIVES NEED TO BE FORMATTED FIRST__

## Quick guide

Commands in my case as an example:

- format the cache drives

```bash
sudo mkfs.ext4 /dev/ubuntu-vg/cache1
sudo mkfs.ext4 /dev/sde
```

- result:

```bash
NAME                      FSTYPE      FSVER    LABEL  UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sde                       ext4        1.0             579f95b4-0f0b-4b01-ad78-d5c27c41851d
nvme0n1
├─nvme0n1p1               vfat        FAT32           083C-24C1                                   1G     0% /boot/efi
├─nvme0n1p2               ext4        1.0             99e874c6-de94-4d13-8f83-47a1d5449618      1.5G    13% /boot
└─nvme0n1p3               LVM2_member LVM2 001        fhJ879-saOg-kZPF-2xxc-KoLZ-g5Qh-M27U9j
  ├─ubuntu--vg-ubuntu--lv ext4        1.0             c5987131-7825-4d3f-9b71-5544cc735bf0     82.7G    10% /
  └─ubuntu--vg-cache1
```

- format the parity and data drives

```bash
sudo mkfs.xfs /dev/sda
sudo mkfs.xfs /dev/sdb
sudo mkfs.xfs /dev/sdc
sudo mkfs.xfs /dev/sdd
```

- result:

```bash
NAME                      FSTYPE      FSVER    LABEL  UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda                       xfs                         2dc82efa-25c9-4e3b-b8d7-af42151db314
sdb                       xfs                         063c8aaf-448f-40f9-99c2-f329c2110fae
sdc                       xfs                         c4c84c0c-12f0-42ae-ad73-955b56a9ad5c
sdd                       xfs                         9c884ae5-0f8e-4000-b17a-36bce1472d6c
sde                       ext4        1.0             579f95b4-0f0b-4b01-ad78-d5c27c41851d
nvme0n1
├─nvme0n1p1               vfat        FAT32           083C-24C1                                   1G     0% /boot/efi
├─nvme0n1p2               ext4        1.0             99e874c6-de94-4d13-8f83-47a1d5449618      1.5G    13% /boot
└─nvme0n1p3               LVM2_member LVM2 001        fhJ879-saOg-kZPF-2xxc-KoLZ-g5Qh-M27U9j
  ├─ubuntu--vg-ubuntu--lv ext4        1.0             c5987131-7825-4d3f-9b71-5544cc735bf0     82.7G    10% /
  └─ubuntu--vg-cache1
```