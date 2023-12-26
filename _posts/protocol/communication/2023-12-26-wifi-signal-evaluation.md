---
title: 无线网卡性能测试
date: 2023-12-26
sidebar: 'auto'
categories:
  - Protocol
tags:
  - WiFI
  - Test
author: ghostxbh
location: blog
summary: 介绍无线网卡性能测试的方法, 主要针对AP模式下的信号强度、信号稳定性、时延迟等进行测试。
---

# 无线网卡说明

介绍无线网卡性能测试的方法, 主要针对AP模式下的信号强度、信号稳定性、时延迟等进行测试。

# 一、信号强度测试

## 1、平板端下载 Cellular-Z 软件

保证控制柜天线无明显遮挡物, 打开控制柜自发的AP信号, 打开**Cellular-Z**软件, 可查看控制柜自发AP的信号**强度**,**频率**,**信道**等信息.

![](http://file.uzykj.com/wifi_signal_evaluation/1.png)	

## 2、测试建议

可分别测试距离控制柜1m,3m,5m,10的信号强度情况

注：信号强度是最高建链速率的保障, 建议信号强度在 -45dbm ～ -30dbm 之间.



# 二、信号稳定性测试

## 1、PC端下载 PingPlotter 软件

[**PingPlotter官网**](http://www.pingplotter.com/)

下载并安装好 PingPlotter.

## 2、PingPlotter 界面介绍

![](http://file.uzykj.com/wifi_signal_evaluation/2.png)	

![](http://file.uzykj.com/wifi_signal_evaluation/3.png)	

注：红色柱状体表示丢包, 黑色折线代表时延的变化.

## 3、测试过程

保证控制柜天线无明显遮挡物, 打开控制柜自发的AP信号, 打开**PingPlotter**软件, 输入控制柜自发AP的网址, 点击开始追踪

![](http://file.uzykj.com/wifi_signal_evaluation/4.png)	

待一定时长后, 可选择停止追踪

## 4、导出测试结果

可选择导出PingPlotter格式的测试结果, 或者导出png图片.

![](http://file.uzykj.com/wifi_signal_evaluation/5.png)	

如下图所示:

![](http://file.uzykj.com/wifi_signal_evaluation/6.png)	

![](http://file.uzykj.com/wifi_signal_evaluation/7.png)	



# 三.使用EW-7811UAC无线网卡开启5G自发ap

## 1.准备EW-7811UAC无线网卡,将其插入到控制柜上  

![](http://file.uzykj.com/wifi_signal_evaluation/8.png)	

此时在控制柜终端输入 `lspci`,若出现如下图所示的内容,表示硬件已识别到了无线网卡

![](http://file.uzykj.com/wifi_signal_evaluation/9.png)	

## 2.安装无线网卡驱动

- 安装依赖包
```
sudo apt update
sudo apt install build-essential dkms
```
- 安装无线网卡驱动
- 拷贝 `7822UAC-7811UAC.zip` 到目标设备上，解压驱动文件，并切换到解压后的目录
```
unzip 7822UAC-7811UAC.zip
cd 7811
```
- 使用 DKSM（支持动态内核模块）安装驱动
```
sudo chmod 755 *.sh
sudo ./dkms-install.sh
```
- 安装后重启设备
```
reboot
```

## 3.修改配置文件

- 终端输入 `ifconfig` 查看是否识别到了无线网卡

![](http://file.uzykj.com/wifi_signal_evaluation/10.png)	

以上图为例，这里的无线网卡名称是 `wlx08beac2cb5f7`

- 修改 /etc/create_ap.conf 配置文件

![](http://file.uzykj.com/wifi_signal_evaluation/11.png)

- 重新启动 create_ap
```
systemctl restart create_ap
systemctl enable create_ap
```

## 4.验证是否是5G

可下载 `Cellular-Z` 软件查看开启的 ap 是否是 5G  

![](http://file.uzykj.com/wifi_signal_evaluation/12.png)

# 四.关于无线网信道修改

首先确认当前无线网卡支持的是2.4G或5G,修改 /etc/create_ap.conf 配置文件中的 `CHANNEL` 参数  

- 2.4G建议信道: 6,8,9,11
- 5G建议信道: 149,165

# 五.示例:无线网卡自发AP测试

## 1、说明

- 通过站在不同的地方观察某个AP信号（dBm值），从而测试不同地方的信号覆盖，dBm值越接近0，信号越强；dBm值越远离0，信号越弱
- 调整了路由器/AP的位置、高度、遮挡，也会相应的影响信号强度
- 正常信号强度，-40 dbm ~ -85 dbm之间，小于 -90 dbm 就很差了，几乎没法连接

## 2、测试案例
- 测试内容：无线网卡测试：（EB-LINK Intel 7265）VS （RTL8822CE）VS（EW-7811UAC）

- 测试场景说明: IS控制柜，测试期间关闭arcs软件，使用大的天线，无遮挡  

- (信道拥堵测试: 如同时开10个控制柜并且自发AP为同一信道,此时再测试信号稳定性和强度)

- 测试流程: 
1.在控制柜上安装好无线模块,确认已开启自发ap  
2.本地电脑使用cmd终端确认可ping通控制柜  
3.信号强度测试:使用`Cellular-Z`app软件在固定距离范围内移动测试信号的强度  
4.信号稳定性测试:使用`PingPlotter`电脑软件在固定距离和固定位置测试不同网卡的信号稳定性  

以本次测试为例, 网卡类型：  
- **EB-LINK Intel 7265**: 2.4G，信道8
- **RTL8822CE**: 2.4G，信道8
- **EDIMAX EW-7822UAC**: 5G，信道165


### 测试结果

- **EB-LINK Intel 7265**(inactivated-7265)

![](http://file.uzykj.com/wifi_signal_evaluation/wifi_7265.jpg)	

①距离：**1m**  
信号强度：-58db  
②距离：**5m**  
信号强度：-60db~-70db  
- 10min信号测试1：平均时延：46.1ms，最小时延：1ms，最大时延：1354ms，丢包率：4.6%  

![](http://file.uzykj.com/wifi_signal_evaluation/7265_5m_1.png)	

- 10min信号测试2：平均时延：17.6ms，最小时延：1ms，最大时延：227ms，丢包率：6.2%  

![](http://file.uzykj.com/wifi_signal_evaluation/7265_5m_2.png)	

- 10min信号测试3：平均时延：11.5ms，最小时延：0.9ms，最大时延：248ms，丢包率：7.1%  

![](http://file.uzykj.com/wifi_signal_evaluation/7265_5m_3.png)	

③距离：**10m**  
信号强度：-69db~-76db  
- 10min信号测试1：平均时延：14.2ms，最小时延：0.9ms，最大时延：356ms，丢包率：7.5%  

![](http://file.uzykj.com/wifi_signal_evaluation/7265_10m_1.png)	

- 10min信号测试2：平均时延：15ms，最小时延：1.1ms，最大时延：306ms，丢包率：7.1%  

![](http://file.uzykj.com/wifi_signal_evaluation/7265_10m_2.png)	

- 10min信号测试3：平均时延：17.2ms，最小时延：1ms，最大时延：582ms，丢包率：7.9%  

![](http://file.uzykj.com/wifi_signal_evaluation/7265_10m_3.png)	

- **RTL8822CE**(inactivated-8822)

![](http://file.uzykj.com/wifi_signal_evaluation/wifi_8822.jpg)	

①距离：**1m**  
信号强度：-49db  
②距离：**5m**  
信号强度：-53db~-63db  
- 10min信号测试1：平均时延：8.4ms，最小时延：1.1ms，最大时延：286ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/8822_5m_1.png)

- 10min信号测试2：平均时延：16.6ms，最小时延：1.1ms，最大时延：2403ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/8822_5m_2.png)

- 10min信号测试3：平均时延：6ms，最小时延：1.1ms，最大时延：242ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/8822_5m_3.png)

③距离：**10m**  
信号强度：-65db~-80db  
- 10min信号测试1：平均时延：32ms，最小时延：1.1ms，最大时延：854ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/8822_10m_1.png)

- 10min信号测试2：平均时延：7.6ms，最小时延：1.2ms，最大时延：224ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/8822_10m_2.png)

- 10min信号测试3：平均时延：11.4ms，最小时延：1.3ms，最大时延：318ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/8822_10m_3.png)

- **EW-7811UAC**(inactivated-IS)

![](http://file.uzykj.com/wifi_signal_evaluation/wifi_7811.jpg)	

①距离：**1m**  
信号强度：-47db  
②距离：**5m**  
信号强度：-58db~-66db  
- 10min信号测试1：平均时延：4ms，最小时延：2ms，最大时延：203ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/7811_5m_1.png)

- 10min信号测试2：平均时延：4.8ms，最小时延：1.8ms，最大时延：313ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/7811_5m_2.png)

- 10min信号测试3：平均时延：2.3ms，最小时延：1.8ms，最大时延：5.4ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/7811_5m_3.png)

③距离：**10m**  
信号强度：-65db~-70db  
- 10min信号测试1：平均时延：2.3ms，最小时延：1.6ms，最大时延：5.6ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/7811_10m_1.png)

- 10min信号测试2：平均时延：2.3ms，最小时延：1.6ms，最大时延：27ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/7811_10m_2.png)

- 10min信号测试3：平均时延：2.3ms，最小时延：1.8ms，最大时延：11ms，丢包率：0%  

![](http://file.uzykj.com/wifi_signal_evaluation/7811_10m_3.png)

- 测试汇总


| **无线网卡类型**    | EB-LINK Intel 7265 | RTL8822CE | EDIMAX EW-7822UAC |
|:-------------:|:------------------:|:---------:|:-----------------:|
| 1m信号强度(db)    | -58                | -49       | -47               |
| 5m信号强度(db)    | -60~-70            | -53~-63   | -58~-66           |
| 5m_1平均时延(ms)  | 46.1               | 8.4       | 4                 |
| 5m_1最小时延(ms)  | 1                  | 1.1       | 2                 |
| 5m_1最大时延(ms)  | 1354               | 286       | 203               |
| 5m_1丢包率(%)    | 4.6                | 0         | 0                 |
| 5m_2平均时延(ms)  | 17.6               | 16.6      | 4.8               |
| 5m_2最小时延(ms)  | 1                  | 1.1       | 1.8               |
| 5m_2最大时延(ms)  | 227                | 2403      | 313               |
| 5m_2丢包率(%)    | 6.2                | 0         | 0                 |
| 5m_3平均时延(ms)  | 11.5               | 6         | 2.3               |
| 5m_3最小时延(ms)  | 0.9                | 1.1       | 1.8               |
| 5m_3最大时延(ms)  | 248                | 242       | 5.4               |
| 5m_3丢包率(%)    | 7.1                | 0         | 0                 |
| 10m信号强度(db)   | -69~-76            | -65~-80   | -65~-70           |
| 10m_1平均时延(ms) | 14.2               | 32        | 2.3               |
| 10m_1最小时延(ms) | 0.9                | 1.1       | 1.6               |
| 10m_1最大时延(ms) | 356                | 854       | 5.6               |
| 10m_1丢包率(%)   | 7.5                | 0         | 0                 |
| 10m_2平均时延(ms) | 15                 | 7.6       | 2.3               |
| 10m_2最小时延(ms) | 1.1                | 1.2       | 1.6               |
| 10m_2最大时延(ms) | 306                | 224       | 27                |
| 10m_2丢包率(%)   | 7.1                | 0         | 0                 |
| 10m_3平均时延(ms) | 17.2               | 11.4      | 2.3               |
| 10m_3最小时延(ms) | 1                  | 1.3       | 1.8               |
| 10m_3最大时延(ms) | 582                | 318       | 11                |
| 10m_3丢包率(%)   | 7.9                | 0         | 0                 |
- 测试总结 

`EDIMAX EW-7822UAC 无线网卡` 性能略高于 `RTL8822CE 无线网卡`，两者性能均优于 `EB-LINK Intel 7265 无线网卡`。


---
收录时间: 2023-12-26

<Vssue :title="$title" />
