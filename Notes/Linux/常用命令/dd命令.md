[TOC]

dd
覆盖或复制一个文件
dd if=/PATH/FROM/SRC of=/PATH/TO/DEST if源文件,of目标
bs=# block size,复制单元大小
count=# 复制多少个bs

磁盘拷贝:
dd if=/dev/sda of=/dev/sdb

备份MBR:
dd if=/dev/sda of=/tmp/mbr.bak bs=512 count=1

模拟磁盘损坏
dd if=/dev/zero of=/dev/sha bs=256|512 count=1
256:bootloader, 512:bootloader和分区表