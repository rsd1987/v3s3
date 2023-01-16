
# Buildroot Package for Allwinner V3s/S3/S3L

> Support for S3/S3L coming soon (no access to hardware for testing as of now)

This repo contains a _buildroot external tree_ for the Allwinner V3s/S3/S3L SoCs. Using buildroot's external tree mechanism allows all project-specific files to be separated from the buildroot internals and easy upgrades to future buildroot releases.

The main goal is to support all available features/peripherals on these SoCs with the latest mainline software, with minimum patches/modifications. Unfortunately, some binary blobs are required for certain features (specifically, video encoding; see more below).

## General Features

- Fast 0.3 seconds boot time (from `Starting kernel...` to `/sbin/init`)
- Automatic start of `udhcpc` and SSH support via `dropbear`
- `initramfs` support, as a simple way to perform full-disk upgrades over the air. This is disabled by default; for more information, see _insert link here_
- `u-boot` env support: boot env variables can be accessed in userspace via the `fw_getenv` and `fw_setenv` commands

## Tested SoC Hardware Features

- Connectivity: SPI, I2C, UART, LRADC, SDIO, USB, Ethernet
- DVP (parallel) camera interface
- H.264 BP/MP/HP video encoding, up to 720p @ 60fps or 1080p @ 30fps
- Hardware cryptographic accelerator (accessible in userspace via /dev/crypto; see [cryptodev-linux](http://cryptodev-linux.org/))

In addition, available features shown in the [Sunxi Mainline Status Matrix](https://linux-sunxi.org/Linux_mainlining_effort#Status_Matrix) in the S3/V3s column should be supported as well, but I did not test them personally.

## Usage Instructions

```
# Change directory to any appropriate location
cd /some/working/directory

# Download the v3s3 external tree
git clone https://github.com/Unturned3/v3s3

# Get a buildroot release (any recent version should work; tested with 2022.05.1)
wget buildroot.org/downloads/buildroot-x.y.z.tar.gz
tar -xf buildroot-x.y.z.tar.gz

cd buildroot-x.y.z
make BR2_EXTERNAL=../v3s3 licheepi_zero_defconfig

# Make any customizations if desired
make nconfig
make linux-nconfig
make uboot-nconfig
make busybox-menuconfig
...

# Download needed source packages
make source

# Build
make all

# Burn output image onto an SD card
sudo dd if=output/images/sdcard.img of=/dev/sdX bs=4M
```

## H264 Encoding Demo

Enable the "CedarVE H.264 encoding demo" option in the `make nconfig` customizations step (located in the `External options` sub-menu), and proceed with the build process.

Demo program source code is available at [h264enc_demo](https://github.com/Unturned3/h264enc_demo)


//copy from whycan  base on 

# 切换到某个合适的目录
cd /some/working/directory

# 下载 buildroot external tree
git clone https://github.com/Unturned3/v3s3

# 下载并解压任意一个 buildroot 版本（最近的版本应该都能用；我测试的是2022.05.1)
wget buildroot.org/downloads/buildroot-x.y.z.tar.gz
tar -xf buildroot-x.y.z.tar.gz

# 切换至 buildroot 目录，并初始化配置
cd buildroot-x.y.z
make BR2_EXTERNAL=../v3s3 licheepi_zero_defconfig

# 打开 buildroot 配置目录
make nconfig

# 勾选 h.264 硬编码 demo 选项
External options --->
	[*] CedarVE H.264 encoding demo

# 下载所需的软件包
make source

# 编译
make all

# 烧录 SD 卡 (将 /dev/sdX 替换成合适的路径）
sudo dd if=output/images/sdcard.img of=/dev/sdX bs=4M

使用方法
本人用的硬件是荔枝派Zero，OV5640 DVP (parallel) 摄像头。荔枝派Zero 第一次上电可能会出现 OV5640 i2c ID 错误的情况，目前暂不知原因，不过重启（软重启，i.e. reboot, 不是插拔电源）一次便可解决。只要 /dev/video0, /dev/media0 存在就没太大问题。media-ctl -p 这个指令也可以用于查看 OV5640 是否注册成功。

用串口、或者 ssh 登录后，/root 目录下会有一个 h264enc_demo 的二进制文件：

./h264enc_demo [width] [height] [FPS] [n_frames]
支持 640x480, 1280x720, 1920x1080。n_frames 为采集的帧数，如果不写的话默认450帧。

所有分辨率均支持 30 FPS；640x480 模式下额外支持 60FPS （这是 OV5640 驱动的限制）

h264enc_demo 会将编码流输出至 /mnt/out.h264，如果录制的 n_frames 较大，可考虑在 /mnt 上挂载一个大 SD 卡分区来存储数据。

copy_video.sh 可通过 scp 指令将 /mnt/out.h264 拷贝至某台远程电脑（需要你自己设置合适的 scp destination）

在装有 ffmpeg 的电脑上，可通过 ffmpeg -framerate 30 -i out.h264 -c copy out.mp4 指令将裸 h264 流转换为 MP4


