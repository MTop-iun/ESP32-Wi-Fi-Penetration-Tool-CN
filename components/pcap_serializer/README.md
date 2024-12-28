# ESP32 Wi-Fi Penetration Tool
## PCAP序列化器组件

该组件将提供的帧格式化为PCAP二进制格式。

它基于[Wireshark的LibPCAP文件格式参考](https://gitlab.com/wireshark/wireshark/-/wikis/Development/LibpcapFileFormat)。
它简单地将新帧附加到结构化缓冲区中,并可以按需获取。

## 使用方法
1. 首先通过调用`pcap_serializer_init()`初始化新的PCAP文件缓冲区。
1. 然后使用`pcap_serializer_append_frame()`将更多帧附加到文件中。
1. 要获取缓冲区,调用`pcap_serializer_get_buffer()`和`pcap_serializer_get_size()`。

## 参考
提供Doxygen API参考文档