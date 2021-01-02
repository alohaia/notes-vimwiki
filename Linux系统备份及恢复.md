# Linux 系统备份及恢复

通用备份方案
--------------------------

### 备份

```bash
# 备份文件储存分区的挂载
sudo mount /dev/sdb3 /mnt
sudo tar --use-compress-program=pigz -cvpf /mnt/system-backup.tgz --exclude=/proc --exclude=/lost+found --exclude=/mnt --exclude=/sys --exclude=/run/media /
# 额外排除 /home
sudo tar --use-compress-program=pigz -cvpf /mnt/system-backup.tgz --exclude=/proc --exclude=/lost+found --exclude=/mnt --exclude=/sys --exclude=/run/media --exclude=/home /
```

### 恢复

**进入 LiveCD**
```bash
# 联网
wifi-menu
# 配置镜像
vim /etc/pacman.d/mirrorlist
```
```
## Country : China
Server = https://mirrors.sjtug.sjtu.edu.cn/manjarostable/$repo/$arch
```
```bash
pacman -S pigz
# 挂载分区（仅作为示例）
## 查看分区
lsblk
## 挂载原系统的 / /boot/efi
mount /dev/nvme0n1p3 /mnt
mount /dev/nvme0n1p1 /mnt/boot/efi
tar --use-compress-program=pigz -xvpf
mkdir -p /mnt/proc /mnt/mnt /mnt/sys /mnt/lost+found
```
```
NAME        FSTYPE FSVER LABEL     UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda
├─sda1      vfat   FAT32 SYSTEM    D0AC-6FE7
├─sda2
├─sda3      ntfs                   546A452C6A450BE4
├─sda4      ntfs         Windows   5E46AF1046AEE7CB
└─sda5      ntfs         Softwares FC1C5CAB1C5C62A6
sdb
├─sdb1      vfat   FAT32           DB46-756D
├─sdb2      swap   1               a74254d7-4922-41e9-b453-15bda42ebb9a
├─sdb3      ntfs         Documents 27597CC223DDCCF5                        346G    44% /mnt
├─sdb4      xfs                    7d276a67-2f37-4ff2-b142-dc18f9c90275
└─sdb5      xfs                    b65a2c0b-7c3b-4bf4-aa73-f32caf75962b
nvme0n1
├─nvme0n1p1 vfat   FAT32           C8C6-7FC3                             510.8M     0% /boot/efi
├─nvme0n1p2 swap   1               7f647658-0103-491f-b571-497ae85e19c8                [SWAP]
├─nvme0n1p3 xfs                    a9eb33b6-6a34-4bbb-b5d4-7c500225b76c   88.8G    11% /
├─nvme0n1p4 xfs                    7fd8a9e3-8e5e-4d7f-99c3-a6a1498bd192  159.7G    20% /home
└─nvme0n1p5 ntfs         Shared    CC00E4A900E49C28                       12.1G    93% /home/aloha/Shared
```

xfs 文件系统的备份与恢复
--------------------------

<font color='red'>需要软件</font>：`xfsdump`

**注意**：
- 只能备份和恢复 XFS 文件系统
- 仅支持已挂载文件系统的备份
- 需要 root 权限
- 备份数据只能用 xfsrestore 恢复

### 备份操作

```bash
# 注意：不能以 / 结尾
# 会要求输入两个“标签”：备份会话标签、媒体标签，可随意
xfsdump -f <backup_file|device> <filesystem> [-s <file|dir>]
# 提前在命令中指定标签
xfsdump -f <backup_file|device> -L <session label> -M <media label> <filesystem> [-s <file|dir>]
# -e 排除被设置了“no dump”标签（通过 chattr 命令设置）的文件
xfsdump -e -f <backup_file> -L <session label> -M <media label> <filesystem> [-s <file|dir>]
# -z 限制文件大小 (kb)，大于指定大小的文件不进行备份
xfsdump -z -f <backup_file|device> -L <session label> -M <media label> <filesystem> [-s <file|dir>]
```

```bash
# 对单一文件设置“no dump”标签
chattr +d <file>
# 对目录递归设置“no dump”标签
chattr -R +d <dir>
```

- `-f` 指定输出文件或着设备
- `<filesystem>` 为要备份的`xfs`文件系统的挂载目录
- `-s` 指定相对`<filesystem>`（挂载目录）的相对路径

### 还原操作

```bash
xfsrestore -f <backup_file|device> <dest>
```

- `<dest>` 指定将备份还原到的位置（挂载目录），即为之前的`<filesystem>`（同一个文件系统，当时载在目录可能不同）
- `-s` 选项用于还原备份时添加了`-s`选项的备份，并指定还原位置（相对`<dest>`（挂载目录）的相对路径）


### 增量备份与还原

#### 备份

> 注意无论是完全备份还是增量备份，都必须存储在单独的文件中。
> `<session label>` 也应当指定新的名称。

> `-l` 选项指定备份等级（1-9，level 为 0 即为完全备份）。若指定备份等级为`3`，则只有在备份等级小于`3`（`1`、`2`）
> 的备份后发生了改变的文件才会在此次备份中被包含。

```bash
# 先进行一次完全备份
xfsdump -f <backup_file|device> -L <session label> -M <media label> <filesystem> [-s <file|dir>]
# -l 指定为第 1 次增量备份
xfsdump -l 1 -f <backup_file|device> -L <session label> -M <media label> <filesystem> [-s <file|dir>]
# -l 指定为第 2 次增量备份
xfsdump -l 2 -f <backup_file|device> -L <session label> -M <media label> <filesystem> [-s <file|dir>]
```

#### 还原

```bash
# 先恢复完全备份
xfsrestore -f <back_full> <dest>
# 按照备份顺序恢复增量备份
xfsrestore -f <back_inc1> <dest>
xfsrestore -f <back_inc2> <dest>
```

### 其他操作

#### 查看备份信息

```bash
xfsdump -I
```

#### 删除所有备份

1. 【可选】删除所有备份
2. 删除 `/var/lib/xfsdump/inventory` 下以文件系统 uuid 命名的文件。

