# ESP32 Wi-Fi Penetration Tool
## Main组件

这是主组件(也称为伪组件)。它包含攻击实现本身和攻击包装器框架。

以下攻击实现的理论基础在[/doc/ATTACK_THEORY.md](../doc/ATTACKS_THEORY.md)中。

### 广播取消认证
发送取消认证帧的一种方法是绕过阻止发送这些帧的Wi-Fi协议栈库。为此使用了[WSL绕过器](../components/wsl_bypasser)组件。关于绕过工作原理的更多细节,请参见WSL绕过器组件的README。

取消认证帧使用广播目标MAC地址(ff:ff:ff:ff:ff:ff)、源MAC地址和目标AP的BSSID构建。

#### 优点
- 不需要有活动通信。即使设备没有主动通信,它们也会接收到这个帧。

#### 缺点
- 如[Aircrack-ng文档](https://www.aircrack-ng.org/doku.php?id=deauthentication#why_does_deauthentication_not_work)所述,某些设备会忽略广播取消认证帧。
    > 某些客户端会忽略广播取消认证。在这种情况下,你需要向特定客户端发送定向的取消认证。

### 伪造AP
另一个选择是启动伪造的复制AP。这种方式不会触及Wi-Fi协议栈库,仅使用ESP-IDF API。考虑到可以为AP接口设置任何有效的MAC地址,我们可以通过使用`esp_wifi_set_mac`设置与真实AP相同的MAC来创建复制AP。
我们从AP扫描器获知所有必要的值,并将它们保存在`wifi_ap_record_t`结构中。从那里我们可以将信息传递给`wifi_config_t`并通过`esp_wifi_set_config`配置AP。一旦这个AP启动,当它从任何STA接收到2类或3类帧时,它将响应取消认证帧。这种行为直接在802.11标准中定义。STA无法验证帧是来自真实AP还是伪造AP,并且会以防御方式从网络中取消认证自己。
这在下面的序列图中演示：

![伪造AP序列图](../doc/drawio/rogueap-seq.drawio.svg)

#### 优点
- 取消认证帧是定向发送给向AP发送帧的STA。

#### 缺点
- 这种方法需要有活动通信正在进行。
- 它可能会完全混淆STA,使其无法再次认证,或者可能尝试与伪造AP而不是真实AP进行认证。(可以通过打开和关闭复制AP来解决,给STA一些重新连接的时间)

### PMKID捕获
要从AP捕获PMKID,我们只需要初始化连接并从AP获得第一个握手消息。如果PMKID可用,AP会将其作为第一个握手消息的一部分发送,所以我们不需要知道凭据。

### 拒绝服务
这重用了上述的取消认证方法,只是跳过了握手捕获。它还允许组合所有取消认证方法,这使其对各种设备的不同行为更具鲁棒性。

## 参考
提供Doxygen API参考文档