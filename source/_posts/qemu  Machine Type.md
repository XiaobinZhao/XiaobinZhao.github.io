---
title: qemu  Machine Type
date: 2021-12-14 09:49:01
tags:
- Machine Type
- qemu
categories:
- 虚拟化
- qemu
---



# qemu  Machine Type

在创建虚拟机时，可以指定qemu参数：Machine Type，来控制虚机采用的CPU芯片架构。

<!-- more -->

可选值一般有：i440fx(pc)和q35.具体可以根据命令查看：

```shell
[root@10-221-126-2 ~]# /usr/libexec/qemu-kvm -M  help 
Supported machines are:
pc                   RHEL 7.6.0 PC (i440FX + PIIX, 1996) (alias of pc-i440fx-rhel7.6.0)
pc-i440fx-rhel7.6.0  RHEL 7.6.0 PC (i440FX + PIIX, 1996) (default)
pc-i440fx-rhel7.5.0  RHEL 7.5.0 PC (i440FX + PIIX, 1996)
pc-i440fx-rhel7.4.0  RHEL 7.4.0 PC (i440FX + PIIX, 1996)
pc-i440fx-rhel7.3.0  RHEL 7.3.0 PC (i440FX + PIIX, 1996)
pc-i440fx-rhel7.2.0  RHEL 7.2.0 PC (i440FX + PIIX, 1996)
pc-i440fx-rhel7.1.0  RHEL 7.1.0 PC (i440FX + PIIX, 1996)
pc-i440fx-rhel7.0.0  RHEL 7.0.0 PC (i440FX + PIIX, 1996)
rhel6.6.0            RHEL 6.6.0 PC
rhel6.5.0            RHEL 6.5.0 PC
rhel6.4.0            RHEL 6.4.0 PC
rhel6.3.0            RHEL 6.3.0 PC
rhel6.2.0            RHEL 6.2.0 PC
rhel6.1.0            RHEL 6.1.0 PC
rhel6.0.0            RHEL 6.0.0 PC
q35                  RHEL-7.6.0 PC (Q35 + ICH9, 2009) (alias of pc-q35-rhel7.6.0)
pc-q35-rhel7.6.0     RHEL-7.6.0 PC (Q35 + ICH9, 2009)
pc-q35-rhel7.5.0     RHEL-7.5.0 PC (Q35 + ICH9, 2009)
pc-q35-rhel7.4.0     RHEL-7.4.0 PC (Q35 + ICH9, 2009)
pc-q35-rhel7.3.0     RHEL-7.3.0 PC (Q35 + ICH9, 2009)
none                 empty machine
```

## i440fx(pc)和q35区别

[Q35-only features](https://wiki.qemu.org/images/4/4e/Q35.pdf)

```
● PCIe “goodies”
– Extended configuration space (MMCFG)
– PCIe native hotplug
– Advanced Error Reporting (AER)
– Alternative Routing-ID Interpretation (ARI)
– Native Power Management
– Function Level Reset (FLR)
– Address Translation Services (ATS) 
● AHCI storage controller
● vIOMMU emulation
● “Secure” Secure Boot
```

[Q35 vs. I440FX](https://remimin.github.io/2019/07/09/qemu_machine_type/)

```
Q35是Intel在2007年9月推出的芯片组，与I440FX/PIIX差别在于：

- Q35 has IOMMU
- Q35 has PCIe
- Q35 has Super I/O chip with LPC interconnect
- Q35 has 12 USB ports
- Q35 SATA vs. PATA

Irq routing

- Q35 PIRQ has 8 pins - PIRQ A-H
- Q35 has two PIC modes – legacy PIC vs I/O APIC
- Q35 runs in I/O APIC mode
- Slots 0-24 are mapped to PIRQ E-H round robin
- PCIe Bus to PIRQ mappings can be programmed
  - Slots 25-31
- Q35 has 8 PCI IRQ vectors available, I440FX/PIIX4 only 2

I440FX/PIIX4 vs. Q35 devices

- AHCI vs. Legacy IDE
- PCI addresses
- Populate slots using flags
- Default slots
```

## zstack应用

以我们熟知的zstack平台为例，创建虚机时的参数一般为：

> CreateVmInstance 
>   zoneUuid=05ea6bfe990e466c90614659125fd6b6 
>   clusterUuid=24082bdb1c8b424bab6b9369f80cd465
>   cpuNum=8
>   defaultL3NetworkUuid=2fe41b60a82948bab4c317475d235ec9
>   hostUuid=3823dad8ffa6408ca2bde58fbd8ff0c3
>   imageUuid=f55dbdcd39f84ee7b0b7e65d3f44b405
>   l3NetworkUuids=2fe41b60a82948bab4c317475d235ec9 
>   memorySize=8589934592 
>   name=zsytest1 
>   systemTags=vmMachineType::q35

其中` systemTags=vmMachineType::q35`参数即可指定qemu-kvm创建的参数中有关Machine Type的指定。

### 默认为i440fx(pc)

一般情况下legacy启动的基于x86_64CPU架构的虚机，不需要指定`systemTags=vmMachineType`参数，machine_type为pc:

```xml
...
  <os>
    <type arch='x86_64' machine='pc-i440fx-rhel7.6.0'>hvm</type>
    <bootmenu enable='yes'/>
    <smbios mode='sysinfo'/>
  </os>
...
```

### q35

设置为q35时有几种情况：

1. CPU架构为`'aarch64', 'mips64el'`,此时boot mode必须为`UEFI`（在项目中遇到的ARM架构的zstack）
2. CPU架构为`'x86_64'`,此时boot mode为`UEFI`(这是实验出来的，尚未找到具体代码)
3. zstack mimi版本需要设置（在项目中遇到的）

### virt

除了pc/q35之外，还有一种machine_type：[virt](https://wiki.qemu.org/Documentation/Platforms/ARM)

```markdown
Generic ARM system emulation with the virt machine
If you don't care about reproducing the idiosyncrasies of a particular bit of hardware, such as small amount of RAM, no PCI or other hard disk, etc., and just want to run Linux, the best option is to use:

-M virt
**virt is a platform which doesn't correspond to any real hardware and is designed for use in virtual machines.**

It supports PCI, virtio, recent CPUs and large amounts of RAM.

See this tutorial for information on getting 32-bit ARM Debian Linux running on the "virt" board.

For 64-bit ARM "virt" is also the best choice, and there's a tutorial for 64-bit ARM Debian Linux setup too.

The "versatilepb" machine has also often been used as a general-purpose Linux target in the past; its disadvantage is that it has a very old CPU and only 256MB of RAM, but it does at least have PCI and SCSI. You can find a description of how to install Debian on it here (the author of that tutorial has also provided some prebuilt images). You're probably better off using "virt" though.
```



## 参考文献

[PCIvsPCIe (qemu.org)](https://wiki.qemu.org/images/f/f6/PCIvsPCIe.pdf)

[PowerPoint Presentation (qemu.org)](https://wiki.qemu.org/images/4/4e/Q35.pdf)

[Qemu虚拟化之Machine Type - 敏的博客 | Min's Blog (remimin.github.io)](https://remimin.github.io/2019/07/09/qemu_machine_type/)



