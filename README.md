---
PR2715W3欧拉认证注意事项
---





操作系统:`openEuler-22.03-LTS-SP3-everything-x86_64-dvd.iso`

# 测试环境搭建

1. 搭建测试环境时，需要使用外网IP（后续测试也会用到），配置 [openEuler 官方 repo](https://gitee.com/link?target=https%3A%2F%2Frepo.openeuler.org%2F) 中对应版本的 everything 和 update repo源，以下为参考配置。

2. 也可以通过挂载本地repo源或提前下载相关软件包等方法，配置本地repo源即可。

```repo源
[OS]
name=OS
baseurl=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/OS/$basearch/
metalink=https://mirrors.openeuler.org/metalink?repo=$releasever/OS&arch=$basearch
metadata_expire=1h
enabled=1
gpgcheck=1
gpgkey=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/OS/$basearch/RPM-GPG-KEY-openEuler

[everything]
name=everything
baseurl=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/everything/$basearch/
metalink=https://mirrors.openeuler.org/metalink?repo=$releasever/everything&arch=$basearch
metadata_expire=1h
enabled=1
gpgcheck=1
gpgkey=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/everything/$basearch/RPM-GPG-KEY-openEuler

[EPOL]
name=EPOL
baseurl=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/EPOL/main/$basearch/
metalink=https://mirrors.openeuler.org/metalink?repo=$releasever/EPOL/main&arch=$basearch
metadata_expire=1h
enabled=1
gpgcheck=1
gpgkey=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/OS/$basearch/RPM-GPG-KEY-openEuler

[debuginfo]
name=debuginfo
baseurl=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/debuginfo/$basearch/
metalink=https://mirrors.openeuler.org/metalink?repo=$releasever/debuginfo&arch=$basearch
metadata_expire=1h
enabled=1
gpgcheck=1
gpgkey=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/debuginfo/$basearch/RPM-GPG-KEY-openEuler

[source]
name=source
baseurl=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/source/
metalink=https://mirrors.openeuler.org/metalink?repo=$releasever&arch=source
metadata_expire=1h
enabled=1
gpgcheck=1
gpgkey=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/source/RPM-GPG-KEY-openEuler

[update]
name=update
baseurl=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/update/$basearch/
metalink=https://mirrors.openeuler.org/metalink?repo=$releasever/update&arch=$basearch
metadata_expire=1h
enabled=1
gpgcheck=1
gpgkey=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/OS/$basearch/RPM-GPG-KEY-openEuler

[update-source]
name=update-source
baseurl=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/update/source/
metalink=https://mirrors.openeuler.org/metalink?repo=$releasever/update&arch=source
metadata_expire=1h
enabled=1
gpgcheck=1
gpgkey=http://repo.openeuler.org/openEuler-22.03-LTS-SP3/source/RPM-GPG-KEY-openEuler
```

# 测试工具安装

目前通过`dnf install oec-hardware`安装的测试工具问题太多，无法完成测试，可自行编译安装对测试中遇到的问题进行验证，官方操作步骤如下，正式测试结果需要通过dnf install oec-hardware安装测试工具

```
克隆仓库源码；
git clone https://gitee.com/openeuler/oec-hardware.git

进入对应的目录，编译安装；
cd oec-hardware
make && make install

打包验证，此处以1.0.0版本为例进行打包，具体打包的版本请以spec文件里的版本为准。
dnf install -y rpm-build 
cd oec-hardware
tar jcvf oec-hardware-1.1.5.tar.bz2 *
mkdir -p /root/rpmbuild/SOURCES
cp oec-hardware-1.1.5.tar.bz2 /root/rpmbuild/SOURCES/
rpmbuild -ba oec-hardware.spec
```

通过手动编译安装在首次运行时会提示缺少相关依赖，根据提示安装即可正常使用。

```
pip install concurrent-log-handler
```





# CPU测试

1. 首先使用`cpupower frequency-info`命令查询CPU频率，若结果如下所示

   ```
   analyzing CPU 0:
     driver: intel_pstate
     CPUs which run at the same hardware frequency: 0
     CPUs which need to have their frequency coordinated by software: 0
     maximum transition latency:  Cannot determine or is not supported.
     hardware limits: 800 MHz - 3.40 GHz
     available cpufreq governors: performance powersave
     current policy: frequency should be within 800 MHz and 3.40 GHz.
                     The governor "performance" may decide which speed to use
                     within this range.
     current CPU frequency: Unable to call hardware
     current CPU frequency: 3.20 GHz (asserted by call to kernel)
     boost state support:
       Supported: yes
       Active: yes

测试就会报错。如果是intel_pstate，可以按照下面的步骤，在grub中增加intel_pstate=disable

- vi /etc/default/grub 在GRUB_CMDLINE_LINUX 所在行最后添加intel_pstate=disable参数。

- 执行grub2-mkconfig -o /boot/efi/EFI/openeuler/grub.cfg (legacy模式下执行grub2-mkconfig -o /boot/grub2/grub.cfg)。

- cat /proc/cmdline 再检查下修改的cmd，然后重启生效。

- cpupower frequency-info 可以查看目前支持5种模式。

  ```
  analyzing CPU 0:
    driver: acpi-cpufreq
    CPUs which run at the same hardware frequency: 0
    CPUs which need to have their frequency coordinated by software: 0
    maximum transition latency: 10.0 us
    hardware limits: 800 MHz - 2.60 GHz
    available frequency steps:  2.60 GHz, 2.60 GHz, 2.50 GHz, 2.30 GHz, 2.20 GHz, 2.10 GHz, 2.00 GHz, 1.80 GHz, 1.70 GHz, 1.60 GHz, 1.40 GHz, 1.30 GHz, 1.20 GHz, 1.10 GHz, 900 MHz, 800 MHz
    available cpufreq governors: conservative ondemand userspace powersave performance schedutil
    current policy: frequency should be within 800 MHz and 2.60 GHz.
                    The governor "performance" may decide which speed to use
                    within this range.
    current CPU frequency: 2.60 GHz (asserted by call to hardware)
    boost state support:
      Supported: yes
      Active: yes
  ```

  2. 如果在测试过程中遇到Test userspace failed.错误，请检查测试工具是否通过dnf install oec-hardware安装，如果是，目前测试工具的代码存在问题，可在测试结果说明，可卸载后自行编译安装工具进行验证。
  3. 如遇到`Max average time of all CPUs userspace load test: 2.17
     Min average time of all CPUs userspace load test: 2.18
     The speedup(0.99) is not between 1.00 and 5.50
     Test userspace failed.`Max average time 小于Min average time的情况，需对CUP进行调优操作，具体方法请教会的同事，我没测过cpu不清楚相关操作。

# Swap分区

1. 安装操作系统时，Swap分区大小需要大于4G，不能等于4G，否则内存测试会报错。

2. 若安装操作系统时，Swap分区大小未调整到4G以上，可采取如下措施

   > 创建一个磁盘分区sda1 `fdisk /dev/sda`

   > 执行`mkswap /dev/sda1`格式化分区

   > 执行`swapon /dev/sda1`挂载swap分区

   > `free -h`查看当前swap可用空间

# disk和raid测试

1. 执行disk和raid测试时，要保证至少有一个空闲的（非系统盘）硬盘和raid，否则会导致硬盘和raid测试失败
2. raid卡测试项需要用户在配置文件 test_config.yaml 中指定相关测试信息
3. disk测试配置信息默认为all，不用修改。
4. disk测试时，需要取消测试盘的挂载，负责会导致测试盘占用，跳过测试。

# USB测试

1. 测试时需要根据提示新插入、拔出一个usb设备（不能拔插机器上在测试前就存在的usb设备）

# ethernet测试

1. 目前如果使用 `dnf` 安装客户端 oec-hardware。	`dnf install oec-hardware	`。在进行Ethernet测试时需要将network.py文件中的代码

``` 
def test_icmp(self):
        """
        Test ICMP
        :return:
        """
        count = 500
        cmd = "ping -q -c %d -i 0 %s | grep 'packet loss' | awk '{print $6}'" % (
            count, self.server_ip)
        for _ in range(self.retries):
            result = self.command.run_cmd(cmd)
            if result[0].strip() == "0%":
                self.logger.info("Test icmp succeed.")
                return True
        self.logger.error("Test icmp failed.")
        return False
```

中的`cmd = "ping -q -c %d -i 0 %s | grep 'packet loss' | awk '{print $6}'" % (count, self.server_ip)`改为`cmd = "ping -q -c %d -i 0.001 %s | grep 'packet loss' | awk '{print $6}'" % (count, self.server_ip)`。否则测试会报错。（修改代码后须在测试结果中进行说明）

2. 测试前需修改配置文件 test_config.yaml 中指定相关测试信息，我的配置如下所示，仅供参考

   ```
   fc:
     fc1:
       device: '0000:03:00.0'
       disk: all
     fc2:
       device: '0000:03:00.1' 
       disk: sda
   raid:
     raid1:
       device: '0000:cb:00.0'
       disk: all
     raid2:
       device: '0000:cb:00.0'
       disk: sdb
   disk: all
   ethernet:
     # IP has been manually configured, get server IP.
     eth1:
       device: ens28f0
       if_rdma: N
       client_ip: 192.168.4.27
       server_ip: 192.168.4.103
     # Configure the IP obtained here.
     eth2:
       device: ens28f1
       if_rdma: N
       client_ip: 192.168.4.107
       server_ip: 192.168.4.103
     # The program automatically generates an IP address for IP configuration
     eth3:
       device: enp125s0f2
       if_rdma: y
       client_ip:
       server_ip:
   infiniband:
     # IP has been manually configured, get server IP.
     ib1:
       device: ibp1s0
       client_ip:
       server_ip: 2.2.2.4
     # Configure the IP obtained here.
     ib2:
       device: ibp1s0
       client_ip: 2.2.2.3
       server_ip: 2.2.2.4:8090
     # The program automatically generates an IP address for IP configuration
     ib3:
       device: ibp1s0
       client_ip:
       server_ip:
   kabiwhitelist:
     ko1:
       ko_name: ''
     ko2:
       ko_name: ''
     rpm:
       rpm_name: ''
   ```

# 其他注意事项

1. 安装完操作系统后请勿更新网卡、raid卡等相关驱动，否则会报`[ERROR] Out-of-tree modules: igb`、`[WARNING] Unsigned modules: igb`等错误。

