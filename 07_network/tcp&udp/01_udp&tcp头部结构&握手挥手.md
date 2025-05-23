### 一、TCP和UDP头部字节定义
#### 1. **TCP头部结构**（20-60字节）
- **源端口/目的端口**（各2字节）：标识发送方和接收方的应用进程。
- **序列号**（4字节）：标识数据流中每个字节的顺序，用于解决乱序问题。
- **确认号**（4字节）：表示期望接收的下一个字节的序列号。
- **数据偏移**（4位）：表示TCP头部的长度（单位：4字节），最小值为5（对应20字节）。
- **控制位**（6位）：包括`URG`（紧急数据）、`ACK`（确认有效）、`PSH`（尽快提交数据）、`RST`（重置连接）、`SYN`（建立连接）、`FIN`（终止连接）。
- **窗口大小**（2字节）：用于流量控制，表示接收方可接受的数据量。
- **校验和**（2字节）：用于检测头部和数据部分的错误。
- **紧急指针**（2字节）：与`URG`配合使用，指示紧急数据的结束位置。

#### 2. **UDP头部结构**（8字节）
- **源端口/目的端口**（各2字节）：标识发送方和接收方的应用进程。
- **长度**（2字节）：表示整个UDP数据报（头部+数据）的总长度，最大为65535字节。
- **校验和**（2字节）：可选字段，用于检测数据错误。

---

### 二、三次握手与四次挥手
#### 1. **三次握手**（建立连接）
1. **客户端 → 服务端**：发送`SYN`报文（序列号`seq=x`），状态变为`SYN_SENT`。
2. **服务端 → 客户端**：回复`SYN+ACK`（`seq=y`, `ack=x+1`），状态变为`SYN_RCVD`。
3. **客户端 → 服务端**：发送`ACK`（`ack=y+1`），双方进入`ESTABLISHED`状态，连接建立。

#### 2. **四次挥手**（关闭连接）
1. **主动关闭方 → 被动关闭方**：发送`FIN`报文（`seq=u`），状态变为`FIN_WAIT_1`。
2. **被动关闭方 → 主动关闭方**：回复`ACK`（`ack=u+1`），状态变为`CLOSE_WAIT`；主动关闭方进入`FIN_WAIT_2`。
3. **被动关闭方 → 主动关闭方**：发送`FIN`报文（`seq=w`），状态变为`LAST_ACK`。
4. **主动关闭方 → 被动关闭方**：回复`ACK`（`ack=w+1`），进入`TIME_WAIT`状态，等待2MSL后关闭；被动关闭方收到后关闭连接。

---

### 三、TIME_WAIT与CLOSE_WAIT状态解析
#### 1. **TIME_WAIT状态**
- **产生原因**：主动关闭方在发送最后一个`ACK`后进入`TIME_WAIT`，需等待2MSL（约60秒）以确保：
  - 若`ACK`丢失，被动关闭方重传的`FIN`能被正确处理。
  - 防止旧连接的报文干扰新连接。
- **影响**：占用端口资源，可能导致短时间无法重用端口。

#### 2. **CLOSE_WAIT状态**
- **产生原因**：被动关闭方收到`FIN`后未正确调用`close()`关闭连接，常见于：
  - 应用程序未释放资源（如未关闭Socket）。
  - 代码逻辑异常导致未执行关闭操作。
- **解决方式**：
  - 确保代码中显式关闭连接（例如使用`try-with-resources`或`finally`块）。
  - 使用工具（如`netstat`）监控并修复泄漏的连接。

---

### 四、Keepalive机制
#### 1. **作用**
检测长时间无数据交互的连接是否存活，避免因网络中断或对方崩溃导致资源浪费。

#### 2. **参数配置**
- **`tcp_keepalive_time`**（默认7200秒）：连接空闲多久后发送探测包。
- **`tcp_keepalive_intvl`**（默认75秒）：探测包发送间隔。
- **`tcp_keepalive_probes`**（默认9次）：连续探测失败次数上限，超过则断开连接。

#### 3. **应用场景**
- 长连接场景（如数据库连接池），防止中间设备（防火墙）超时断开。
- 实时通信（如视频会议），快速感知对方异常。

---

### 五、总结
- **TCP/UDP差异**：TCP通过复杂头部和流程保证可靠性，UDP轻量但不可靠。
- **状态处理**：`TIME_WAIT`是正常关闭流程的一部分，`CLOSE_WAIT`需代码优化避免。
- **Keepalive**：需根据场景调整参数，平衡资源消耗与连接可靠性。