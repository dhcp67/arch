# ArchLinux 安装配置流程



一、ArchLinux 系统安装

1. 1. 引导界面，选择Arch Linux install medium

   2. 连接wifi:`iwd`

      * `device list ` **查询机器的网卡设备**
      * `station 网卡名称 scan`**查询附近可用 WIFI 网络 **
      * `station 网卡名称 get-networks` **显示扫描结果**
      * `station 网卡名称 connect`  **连接无线网络**

   3. 修改镜像源

      * `vim /etc/pacman.d/mirrorlist ` **最前面几行改成国内源地址**

        ``Server = https://mirrors.aliyun.com/archlinux/$repo/os/$arch
        Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
        Server = https://mirrors.163.com/archlinux/$repo/os/$arch``

   4. 更新系统时间

      * `timedatectl set-ntp true`  **时间同步**
      * `timedatectl list-timezones`  **列出所有时区**
      * `timedatectl set-timezone time-zone( Asia/Shanghai)`  **设置时区**

   5. 配置磁盘分区

      `lsblk`命令查看磁盘状态

      | 挂载点        | 分区      | 分区类型            | 建议大小                      |
      | ------------- | --------- | ------------------- | ----------------------------- |
      | /mnt          | /dev/sda3 | Linux 根分区（/）   | 20GB-50GB，具体视硬盘大小决定 |
      | /mnt/boot/EFI | /dev/sda1 | EFI 系统分区        | 300MB                         |
      | [SWAP]        | /dev/sda2 | Linux swap 交换空间 | 4GB-16GB                      |
      | /mnt/home     | /dev/sda4 | 用户分区目录        | 剩余空间                      |

      * 建立 GPT 分区表

        `fdisk /dev/sda` **对/dev/sda进行分区**

        `g`**建立 GPT 分区表**

      * 建立EFI、swap、/、home分区

        `n` **建立分区**

        `p` **查看分区列表**

        `t` **改变分区类型，`L`查看可选列表，swap为linux swap**

        `w` **保存**

      * 格式化分区

        `lsblk`  **查看分区，注意分区大小和盘符**

        `mkswap /dev/sda2`  **格式化 swap 分区**

        ``swapon /dev/sda2``  **启用 swap 分区**

        `mkfs.ext4 /dev/sda3` **格式化/分区为 ext4 格式**

        `mkfs.ext4 /dev/sda4` **格式化/home 分区为 ext4 格式**

   6. 挂载分区

      * `mount /dev/sda3 /mnt` **将/dev/sda3 分区挂载到/目录**
      * `mkdir /mnt/boot` **建立/boot目录**
      * `mkdir /mnt/boot/EFI` **建立/boot/EFI 目录**
      * `mkdir /mnt/home` **建立/home 目录**
      * `mount /dev/sda1 /mnt/boot/EFI` **将/dev/sda1 分区挂载到/boot/EFI 目录**
      * `mount /dev/sda4 /mnt/home` **将/dev/sda4 分区挂载到/home 目录**

   7. 开始安装系统

      * `pacstrap -i /mnt base base-devel linux linux-firmware` **安装必须的 base 软件包和 Linux 内核及固件**

   8. 配置基础系统

      * 配置 fstab

        `genfstab -U /mnt >> /mnt/etc/fstab` **生成/etc/fstab 文件，`cat /mnt/etc/fstab` 检查是否正确。**

      * Chroot 到新系统

        `arch-chroot /mnt /bin/bash` **切换到系统**

        `pacman -S vim dhcpcd` **安装vim 编辑器和dhcpcd**

        `systemctl enable dhcpcd` **开机自动启动 dhcpcd 服务**

        `pacman -S network-manager-applet` 安装网络服务 

        `systemctl enable NetworkManager` k开机启动

      * 设置时区

        `ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime` **设置时区上海**

        `hwclock --systohc --utc` **运a行 hwclock 以生成/etc/adjtime**

      * 设置本地语言

        `vim /etc/locale.gen`选择中文，执行`locale-gen`

   9. 引导系统

      * `pacman -S dosfstools grub efibootmgr` **GRUB 进行 UEFI 引导**
      * `grub-install --target=x86_64-efi --efi-directory=/boot/EFI --recheck` **安装引导工具**
      * `grub-mkconfig -o /boot/grub/grub.cfg` **创建配置文件**
      * `sudo pacman -S os-prober` **安装os-prober探测windows分区，安装后执行上一条命令** 
      * `sudo pacman -S ntfs-3g`   **如果还没有探测到，安装ntfs-3g后再创建配置文件**

   10. 用户管理

       * `passwd` **设置 root 用户密码**

       * 添加用户

         `useradd -m -g users -s /bin/bash username` **username 改成需要添加的用户名**

         `passwd username` **给用户设置密码**

       * 把用户加入到 sudo 组

         `vim /etc/sudoers` 在 root ALL=(ALL) ALL 下面增加一行：username ALL=(ALL) ALL，保存

   11. 安装iwd

       * `sudo pacman -S iwd` **安装iwd**
       * `systemctl enable iwd` **设置开机自启**

   12. `reboot`重启并连接网络

   13. 桌面环境配置

       * 显卡驱动

         执行 `lspci | grep VGA`确定显卡型号，`pacman -S 驱动包` 安装驱动

         | 显卡              | 驱动               |
         | ----------------- | ------------------ |
         | intel-gpu         | xf86-video-intel   |
         | amdgpu            | xf86-video-amdgpu  |
         | Geforce7±         | xf86-video-nouveau |
         | VMware 虚拟机显卡 | f86-video-vmware   |
         | ati               | xf86-viWdeo-ati    |
         | 通用              | xf86-video-vesa    |

         **Nvidia 显卡驱动： GeForce 400 及以上版本安装 nvidia 或者 nvidia-lts GeForce 8000/9000、ION 以及 100-300（NV5x, NV8x, NV9x and NVAx）等 2006-2010 的显卡型号，安装 nvidia-340xx (AUR) 或者 nvidia-340xx-lts (AUR) 。 --> pacman -S cude 、pacman -S nvidia-settings**

       * 内核驱动
       
         ucode 微码固件：intel CPU 微码固件：`pacman -S intel-ucode` 
       
         amd CPU 微码固件：`pacman -S amd-ucode`
       
   14. 安装 KDE 桌面环境

       * 安装KDE和xorg

         `pacman -S xorg plasma`

         `pacman -S konsole dolphin kate plasma-nm google-chrome ark ntfs-3g samba` **安装必备的一些软件**

         `systemctl enable sddm` **启用 sddm 显示管理器**

       * 安装字体

         `pacman -S ttf-dejavu wqy-microhei wqy-zenhei wqy-bitmapfont ttf-arphic-uming noto-fonts-cjk`

       * 笔记本驱动

         `pacman -S xf86-input-synaptics` **触控板驱动**

         `sudo cp /usr/share/X11/xorg.conf.d/70-synaptics.conf /etc/X11/xorg.conf.d/&& vim /etc/X11/xorg.conf.d/70-synaptics.conf`  并且修改为下面内容

         ```rust
         Section "InputClass"
                 Identifier "touchpad"
                 Driver "synaptics"
                 MatchIsTouchpad "on"
                         Option "TapButton1" "1"
                         Option "TapButton2" "3"
                         Option "TapButton3" "0"
                         Option "VertEdgeScroll" "on"
                         Option "VertTwoFingerScroll" "on"
                         Option "HorizEdgeScroll" "on"
                         Option "HorizTwoFingerScroll" "on"
                         Option "VertScrollDelta" "-112"
                         Option "HorizScrollDelta" "-114"
                         Option "MaxTapTime" "125"
         EndSection
         ```
         
         `pacman -S fprintd libfprint` **指纹识别驱动**
         
       
   15. 安装按键控制音量

       * ` sudo pacman -S alsa-utils`  **安装alsa工具包**
       * `alsamixer` **解除静音**
       * `xfce4-pulseaudio-plugin` **按键音量**
       
   16. 命令补全

       * `pacman -S bash-completion` **安装bash-completion**
       * ```bash
         if [ -f /etc/bash_completion ]; then
         . /etc/bash_completion
         fi
         ```

       * 在.bashrc加入以上内容
