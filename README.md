# ติดตั้ง MinIO Object Storage บน Ubuntu 24.04LTS

## ตรวจสอบ Partition ทั้งหมด
```bash
lsblk
```
`
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   50G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   48G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   24G  0 lvm  /
sdb                         8:16   0  500G  0 disk 
└─sdb1                      8:17   0  500G  0 part 
sr0                        11:0    1  2.6G  0 rom
`
