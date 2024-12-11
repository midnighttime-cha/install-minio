# ติดตั้ง MinIO Object Storage บน Ubuntu 24.04LTS

## ตรวจสอบ Partition ทั้งหมด
```bash
lsblk
```
```
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   50G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   48G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   24G  0 lvm  /
sdb                         8:16   0  500G  0 disk 
sr0                        11:0    1  2.6G  0 rom
```

## สร้าง Partition ใหม่
```bash
sudo parted /dev/sdb
```
- ตั้งค่าตารางพาร์ติชันเป็น GPT (สำหรับดิสก์ขนาดใหญ่):
```bash
(parted) mklabel gpt
```
- สร้างพาร์ติชันใหม่ ใช้คำสั่ง mkpart เพื่อสร้างพาร์ติชันใหม่:
```bash
(parted) mkpart primary ext4 0% 100%
```
- ตรวจสอบพาร์ติชัน ดูพาร์ติชันที่สร้างแล้ว:
```bash
(parted) print
```
- ออกจากโปรแกรม:
```bash
(parted) quit
```

## ตรวจสอบ Partition
```bash
lsblk
```
```
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   50G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   48G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   24G  0 lvm  /
sdb                         8:16   0  500G  0 disk 
└─sdb1                      8:17   0  500G  0 part 
sr0                        11:0    1  2.6G  0 rom
```

## Mount partition เข้ากับ Directory
```bash
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data
```

## ติดตั้ง MinIO
- Donwload ไฟล์ติดตั้ง
```bash
wget https://dl.min.io/server/minio/release/linux-amd64/archive/minio_20241107005220.0.0_amd64.deb -O minio.deb
```
- ทำการติดตั้ง
```bash
sudo dpkg -i minio.deb
```

## สร้างไฟล์ systemd
```bash
sudo nano /usr/lib/systemd/system/minio.service
```
- คัดลอกคำสั่งต่อไปนี้ไปในไฟล์
```
[Unit]
Description=MinIO
Documentation=https://min.io/docs/minio/linux/index.html
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
WorkingDirectory=/usr/local

User=minio-user
Group=minio-user
ProtectProc=invisible

EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

# MinIO RELEASE.2023-05-04T21-44-30Z adds support for Type=notify (https://www.freedesktop.org/software/systemd/man/systemd.service.html#Type=)
# This may improve systemctl setups where other services use `After=minio.server`
# Uncomment the line to enable the functionality
# Type=notify

# Let systemd restart this service always
Restart=always

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65536

# Specifies the maximum number of threads this process can create
TasksMax=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target

# Built for ${project.name}-${project.version} (${project.name})
```
- จากนั้นรันคำสั่งต่อไปนี้
```bash
sudo groupadd -r minio-user
sudo useradd -M -r -g minio-user minio-user
sudo chown minio-user:minio-user /mnt/data
```

## สร้างไฟล์ Environment ของระบบ
- แก้ไขไฟล์
```bash
sudo nano /etc/default/minio
```
- คัดลอกการตั้งค่าข้างล่างใส่ในไฟล์ `/etc/default/minio`
```bash
# MINIO_ROOT_USER and MINIO_ROOT_PASSWORD sets the root account for the MinIO server.
# This user has unrestricted permissions to perform S3 and administrative API operations on any resource in the deployment.
# Omit to use the default values 'minioadmin:minioadmin'.
# MinIO recommends setting non-default values as a best practice, regardless of environment

MINIO_ROOT_USER=myminioadmin
MINIO_ROOT_PASSWORD=minio-secret-key-change-me

# MINIO_VOLUMES sets the storage volume or path to use for the MinIO server.

MINIO_VOLUMES="/mnt/data"

# MINIO_OPTS sets any additional commandline options to pass to the MinIO server.
# For example, `--console-address :9001` sets the MinIO Console listen port
MINIO_OPTS="--console-address :9001"
```

## Start the MinIO Service
```bash
sudo systemctl start minio.service
```
- ตรวจสอบสถานะโปรแกรม
```bash
sudo systemctl status minio.service
```
- Enable service
```bash
sudo systemctl enable minio.service
```

