# uboot移植

[^硬件]: 飞凌mx6ul开发板

## 一、代码源

``` makefile
git clone https://source.codeaurora.org/external/imx/uboot-imx
git branch -a
git checkout remotes/origin/nxp/imx_v2015.04_4.1.15_1.0.0_ga
```

## 二、根据现有板创建用户板

```c
//【1】复制板载文件
cp -R  board/freescale/mx6ul_14x14_evk board/freescale/mx6ul_forlinx
    
//【2】复制头文件
cp include/configs/mx6ul_14x14_evk.h include/configs/mx6ul_forlinx.h
    
//【3】复制配置文件
cp configs/
```

## 三、修改文件名及相关文件

###### 修改文件名

```bash
mv board/freescale/mx6ul_14x14_evk board/freescale/mx6ul_forlinx/mx6ul_14x14_evk.c
    board/freescale/mx6ul_14x14_evk board/freescale/mx6ul_forlinx/mx6ul_forlinx.c

```

###### 修改板级的makefile

```c
obj-y  := mx6ul_forlinx.o
```

###### 修改Kconfig

```c
if TARGET_MX6UL_FORLINX

config SYS_BOARD
	default "mx6ul_forlinx"

config SYS_VENDOR
	default "freescale"

config SYS_SOC
	default "mx6"

config SYS_CONFIG_NAME
	default "mx6ul_forlinx"

endif
```

###### 修改include/configs/mx6ul_forlinx.h

```bash
#ifndef __MX6UL_FORLINX_CONFIG_H
#define __MX6UL_FORLINX_CONFIG_H
```

###### 修改板级MAINTAINERS文件

```c
MX6ULFORLINX BOARD
M:	Fabio Estevam <fabio.estevam@freescale.com>
S:	Maintained
F:	board/freescale/mx6ul_forlinx/
F:	include/configs/mx6ul_forlinx.h
F:	configs/mx6ul_forlinx_defconfig
```

###### 修改修改arch/arm/Kconfig

添加以下内容

```
config TARGET_MX6UL_FORLINX
        bool "Support mx6ul_forlinx"
        select CPU_V7
//source放到后面和其他source语句放一起
source "board/freescale/mx6ul_forlinx/Kconfig"
```

###### 修改deconfig文件

```c
CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/freescale/mx6ul_forlinx/imximage.cfg,MX6UL"
CONFIG_ARM=y
CONFIG_TARGET_MX6UL_FORLINX=y
CONFIG_DM=y
CONFIG_DM_THERMAL=y
```

###### 创建编译脚本文件

```makefile
#!/bin/bash 
export ARCH=arm 
export CROSS_COMPILE=<path to cross compiler prefix> (e.g., /opt/poky/1.4.1/sysroots/i686-pokysdk-linux/usr/bin/cortexa9hf-vfp-neon-poky-linuxgnueabi/arm-poky-linux-gnueabi-
make distclean
make mx6ul_forlinx_deconfig 
make
```

## 四、调试技巧

##### 打开debug的方法

在include/common.h下添加

```c
#define DEBUG	/*该行为添加内容*/
#ifdef DEBUG
#define _DEBUG	1
#else
#define _DEBUG	0
#endif
```

##### 自定义命令的方法

###### 新建指令文件

在common下新建文件cmd_ledoen.c

```
#include <common.h>
#include <command.h>

int do_ledoen(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[])
{
    printf("hello ledoen!\n");

    return 0;
}

U_BOOT_CMD(
    ledoen, CONFIG_SYS_MAXARGS, 1, do_ledoen,
    "display enet2 reg",
    "by ledoen"
);
```

###### 修改Makefile文件

修改common文件夹下的Makefile文件，在最后一行添加内容

```c
obj-y += cli.o
obj-y += cli_readline.o
obj-y += command.o
obj-y += s_record.o
obj-y += xyzModem.o
obj-y += cmd_disk.o


obj-y += cmd_ledoen.o	/*此行为添加内容*/

CFLAGS_env_embedded.o := -Wa,--no-warn -DENV_CRC=$(shell tools/envcrc 2>/dev/null)
```

## 五、调试记录

##### 在receive函数里，寄存器的值

###### EIR

- 值0xe000000
- 第25、26、27位为1，对应TXF、TXB、RXF

###### EIMR

- 值0

###### ECR

- 值0xf0000102
- 第1、8位为1，对应使能位、DBSWP位

###### RCR

- 值0x5ee0124
- 第2、5、8位为1，对应MII_MODE、FCE、RMII_MODE

###### TCR

- 值0x4
- 第2位为1，对应FDEN

###### OPD

- 值0x10020
- 第5、16位为1，对应OPCODE、PAUSE_DUR

###### TFWR

- 值0x102
- 第1、8位为1，对应STRFWD、TFWR

###### RDSR

- 值0x9ef48780

###### TDSR

- 值0x9ef48700

###### MRBR

- 值0x600
- 对应R_BUF_SIZE：1536

##### phy寄存器的值

```
Trying FEC1
fec_mii_setspeed: mii_speed 0000001a
fec_open: fec_open(dev)
fec_mdio_read: phy: 01 reg:01 val:0x786d
fec_mdio_read: phy: 01 reg:01 val:0x786d
fec_mdio_read: phy: 01 reg:04 val:0x81e1
fec_mdio_read: phy: 01 reg:05 val:0xcde1
fec_open:Speed=100


ledoen ECR 4026532098
ledoen RCR 99483940
ledoen MRBR 1536
ledoen TFWR 258
ledoen OPD 65568
ledoen RDSR 2164768768
ledoen TDSR 2164764672
ledoen MSCR 820
ledoen RDAR 16777216



```

## 六、中断的作用

接收中断被触发->通知上层有数据进入->上层调用读函数->读到NULL->退出

这是也ENET_ReadFrame被调用两次的原因，第一次读到数据，第二次读到NULL



和Uboot的管脚定义进行对比