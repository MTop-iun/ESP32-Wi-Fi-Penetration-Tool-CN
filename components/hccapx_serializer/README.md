# ESP32 Wi-Fi Penetration Tool
## HCCAPX序列化器组件

该组件将提供的帧格式化为HCCAPX([hashcat](https://hashcat.net/hashcat/))二进制格式。

它基于[Hashcat的HCCAPX文件格式参考](https://hashcat.net/wiki/doku.php?id=hccapx)。
它解析作为WPA握手一部分的EAPOL-Key数据包(使用[帧分析器组件](../frame_analyzer)),并构建HCCAPX格式的文件,该文件可以直接提供给hashcat来破解PSK(预共享密钥,通常被称为*网络密码*)。

## 使用方法
1. 首先通过调用`hccapx_serializer_init`提供目标AP的SSID来初始化序列化器
2. 通过调用`hccapx_serializer_add_frame()`添加更多握手帧
3. 调用`hccapx_serializer_get()`获取存储HCCAPX二进制数据的缓冲区指针

## 参考
提供Doxygen API参考文档