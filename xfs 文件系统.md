# xfs 文件系统

优化安装方案：
```bash
sudo mkfs.xfs -f -i size=512 -l size=128m,lazy-count=1 -d agcount=4 /dev/sdd3
```

使用 `xfsdump`、`xfsrestore` 备份和还原：[xfs 文件系统的备份与恢复](Linux系统备份及恢复.md#xfs 文件系统的备份与恢复)

