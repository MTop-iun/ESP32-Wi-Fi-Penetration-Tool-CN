# ESP32 Wi-Fi渗透测试工具

这个项目为ESP32平台提供了一个用于实现各种Wi-Fi攻击的通用工具。它提供了Wi-Fi攻击中常用的一些通用功能,简化了新攻击的实现。它还包括Wi-Fi攻击本身,如从握手过程中捕获PMKID,或通过启动伪造的复制AP或直接发送取消认证帧等不同方法来捕获握手包等。

显然破解不是这个项目的一部分,因为ESP32不足以有效地破解哈希值。其余的工作都可以在这个小巧、廉价、低功耗的SOC上完成。

<p align="center">
    <img src="doc/images/logo.png" alt="Logo">
</p>

## 功能特点
- **PMKID捕获**
- **WPA/WPA2握手包捕获**和解析  
- 使用多种方法进行**取消认证攻击**
- **拒绝服务攻击**
- 将捕获的流量格式化为**PCAP格式**
- 将捕获的握手包解析为可被Hashcat破解的**HCCAPX文件**
- 被动握手包嗅探
- 易于扩展的新攻击实现框架
- 管理接入点(AP),方便使用智能手机进行即时配置
- 更多......

## 使用方法
1. [构建](#Build)并[烧录](#Flash)项目到ESP32(DevKit或模块)
1. 给ESP32供电
1. 管理AP会在启动后自动启动
1. 连接到此AP\
默认设置:
*SSID:* `ManagementAP` 和 *密码:* `mgmtadmin`
1. 在浏览器中打开`192.168.4.1`,您将看到如下所示的Web客户端界面用于配置和控制工具:

    ![Web客户端界面](doc/images/ui-config.png)

## 构建
本项目目前使用ESP-IDF 4.1版本开发(提交号`5ef1b390026270503634ac3ec9f1ec2e364e23b2`)。在更新版本上可能无法正常工作。

项目可以使用常规的ESP-IDF方式构建:

```shell
idf.py build
```
本项目不支持使用`make`的传统构建方式。


## 烧录
如果您已经配置好ESP-IDF,最简单的方法是使用`idf.py flash`。

如果您不想配置整个ESP-IDF环境,可以使用[`build/`](build/)目录中包含的预构建二进制文件,并使用[`esptool.py`](https://github.com/espressif/esptool)进行烧录(需要Python环境)。

示例命令(按照[esptool仓库](https://github.com/espressif/esptool)中的说明操作):

```
esptool.py -p /dev/ttyS5 -b 115200 --after hard_reset write_flash --flash_mode dio --flash_freq 40m --flash_size detect 0x8000 build/partition_table/partition-table.bin 0x1000 build/bootloader/bootloader.bin 0x10000 build/esp32-wifi-penetration-tool.bin
```

在Windows系统上,您可以使用官方的[Flash下载工具](https://www.espressif.com/en/support/download/other-tools)。



## 文档
### Wi-Fi攻击
本项目中的攻击实现在[主组件README](main/)中有描述。这些攻击背后的理论可以在[doc/ATTACKS_THEORY.md](doc/ATTACKS_THEORY.md)中找到。

### API参考
本项目使用Doxygen符号来记录组件API和实现。项目中包含了Doxyfile文件,如果您想生成API参考文档,只需在根目录下运行`doxygen`命令即可。它将在`doc/api/html`目录下生成HTML格式的API参考文档。

### 组件
本项目由多个可在其他项目中重用的组件组成。每个组件都有其自己的README文件进行详细说明。以下是各组件的简要描述：

- [**Main**](main) 组件是本项目的入口点。所有必要的初始化步骤都在这里完成。管理AP启动后,控制权移交给网络服务器。
- [**Wifi Controller**](components/wifi_controller) 组件封装了所有Wi-Fi相关操作。用于启动AP、作为STA连接、扫描附近AP等。
- [**Webserver**](components/webserver) 组件提供Web界面来配置攻击。它要求AP已启动,且未启用SSL加密等额外安全功能。
- [**Wi-Fi Stack Libraries Bypasser**](components/wsl_bypasser) 组件绕过Wi-Fi协议栈库的限制,以发送某些类型的任意802.11帧。
- [**Frame Analyzer**](components/frame_analyzer) 组件处理捕获的帧并为其他组件提供解析功能。
- [**PCAP Serializer**](components/pcap_serializer) 组件将捕获的帧序列化为PCAP二进制格式,并提供给其他组件(主要用于webserver/UI)。
- [**HCCAPX Serializer**](components/hccapx_serializer) 组件将捕获的帧序列化为HCCAPX二进制格式,并提供给其他组件(主要用于webserver/UI)。

### 功耗
根据实验测量,ESP32在执行攻击时的电流消耗约为100mA。

## 贡献
欢迎贡献代码。请随意重构当前的代码库。在注释新函数和文件时请遵循Doxygen标记规范。本项目主要用于教育和演示目的,因此欢迎详细的文档说明。

## 免责声明
本项目展示了Wi-Fi网络及其底层802.11标准的漏洞,以及如何利用ESP32平台对这些漏洞进行攻击。请仅在获得授权的网络上负责任地使用。