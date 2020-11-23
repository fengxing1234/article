---
title: android.os.Build详解
date: 2020-09-22 08:56:27
tags:
---

**`android.os.Build`**类主要用于获取一些设备初始化信息和系统属性，该类的主要信息都是通过一些 `static` 修饰的属性和方法获得。

## 属性

| 类型     | 名称                  | 描述                                                         |
| :------- | :-------------------- | :----------------------------------------------------------- |
| String   | ID                    | 变更列表编号或“M4-rc20”之类的标签                            |
| String   | DISPLAY               | 向用户显示的构建ID字符串                                     |
| String   | PRODUCT               | 产品名称                                                     |
| String   | DEVICE                | 外观设计名称                                                 |
| String   | BOARD                 | 设备主板信息                                                 |
| String   | CPU_ABI               | CPU指令集(API 21被标记为过时，建议使用*SUPPORTED_ABIS*)      |
| String   | CPU_ABI2              | CPU指令集2(API 21被标记为过时，建议使用*SUPPORTED_ABIS*)     |
| String   | MANUFACTURER          | 产品/硬件制造商                                              |
| String   | BRAND                 | 设备系统定制商                                               |
| String   | MODEL                 | 最终用户可见的设备版本号                                     |
| String   | BOOTLOADER            | 系统启动程序版本号                                           |
| String   | RADIO                 | 无线电固件版本号(API 15被标记为过时，建议使用*getRadioVersion()*方法) |
| String   | HARDWARE              | 硬件名称                                                     |
| boolean  | IS_EMULATOR           | 是否为模拟器                                                 |
| String   | SERIAL                | 硬件序列号(API 26被标记为过时，建议使用*getSerial()*方法)    |
| String[] | SUPPORTED_ABIS        | 设备支持的CPU指令集                                          |
| String[] | SUPPORTED_32_BIT_ABIS | 设备支持的32位CPU指令集                                      |
| String[] | SUPPORTED_64_BIT_ABIS | 设备支持的64位CPU指令集                                      |
| String   | TYPE                  | 构建类型                                                     |
| String   | TAGS                  | 用逗号分隔的描述构建的标签                                   |
| String   | FINGERPRINT           | 构建的唯一识别码                                             |
| long     | TIME                  | 以毫秒为单位的构建生成时间                                   |
| String   | USER                  | *仅对内部工程构建有意义*                                     |
| String   | HOST                  | *仅对内部工程构建有意义*                                     |

## 方法

| 类型    | 名称                         | 描述                                 |
| :------ | :--------------------------- | :----------------------------------- |
| List    | getFingerprintedPartitions() | 获取有关已定义单独指纹的分区构建信息 |
| String  | getRadioVersion()            | 获取无线电固件版本号                 |
| String  | getSerial()                  | 获取硬件序列号                       |
| boolean | is64BitAbi()                 | 判断是否为64位系统                   |


