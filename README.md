````markdown
# SerialComm 串口通信模块

一个用于串口通信的 Go 模块，支持：

- CRC16 校验
- 长度前缀协议
- Base64 + JSON 解包
- 自定义回调函数处理
- 可与 EdgeX Foundry 等工业框架集成

---

## 📦 安装

```bash
go get github.com/yourname/serialcomm
````

---

## 🧩 目录结构

```
serialcomm/
├── receiver.go     // 串口接收逻辑
├── sender.go       // 串口发送逻辑
├── serialcomm.go   // 接口和配置结构体定义
├── types.go        // 消息与负载结构体定义
└── utils.go        // CRC、反馈等工具函数
```

---

## 🚀 快速开始

### 1️⃣ 创建接收器（Receiver）

```go
package main

import (
    "log"
    "time"

    "github.com/yourname/serialcomm/serialcomm"
)

func main() {
    receiver, err := serialcomm.NewSerialReceiver(&serialcomm.SerialConfig{
        PortName:     "/dev/ttyUSB0",        // 或 COM3, COM7 等
        BaudRate:     115200,
        ReadTimeout:  500 * time.Millisecond,
        MaxLength:    10000,
        ReadCallback: handleMessage,
    })
    if err != nil {
        log.Fatalf("创建串口接收器失败: %v", err)
    }

    err = receiver.Start()
    if err != nil {
        log.Fatalf("启动接收器失败: %v", err)
    }

    // 阻塞主线程
    select {}
}

func handleMessage(msg *serialcomm.Message, payload *serialcomm.Payload) {
    log.Printf("收到消息: %+v", msg)
    log.Printf("Payload: %+v", payload)
}
```

---

### 2️⃣ 发送数据（Sender 示例）

```go
sender, err := serialcomm.NewSerialSender("/dev/ttyUSB0", 115200)
if err != nil {
    log.Fatal(err)
}
defer sender.Close()

data := []byte(`{"hello":"world"}`)
err = sender.Send(data)
if err != nil {
    log.Fatal(err)
}
```

---

## 📚 数据结构说明

### 🔹 Message（外层封装）

```go
type Message struct {
    APIVersion    string `json:"apiVersion"`
    ReceivedTopic string `json:"receivedTopic"`
    CorrelationID string `json:"correlationID"`
    RequestID     string `json:"requestID"`
    ErrorCode     int    `json:"errorCode"`
    Payload       string `json:"payload"`     // base64 encoded JSON
    ContentType   string `json:"contentType"` // e.g. "application/json"
}
```

### 🔹 Payload（Base64 解码后）

```go
type Payload struct {
    APIVersion string `json:"apiVersion"`
    RequestID  string `json:"requestID"`
    Event      Event  `json:"event"`
}
```

---

## 🧪 单元测试

模块内部使用 `testify` 进行 mock 和测试，测试覆盖：

* CRC 校验正确性
* 协议长度与超时逻辑
* 回调是否正常触发

运行测试：

```bash
go test ./serialcomm/...
```

---

## 🔐 协议格式（Wire Format）

| 字段            | 长度   | 描述                   |
| ------------- | ---- | -------------------- |
| 长度（BigEndian） | 4 字节 | 数据包 JSON 长度（不含 CRC）  |
| 数据 JSON       | N 字节 | Message 对象 JSON（被解包） |
| CRC 校验码       | 2 字节 | MODBUS CRC16 检验码     |

---

## 🧩 依赖

* [github.com/tarm/serial](https://github.com/tarm/serial) — 串口库
* [github.com/sigurn/crc16](https://github.com/sigurn/crc16) — CRC16 校验

---

## 🧑‍💻 作者

维护者：[clint](https://github.com/clint456)
仓库地址：[github.com/yourname/serialcomm](https://github.com/clint456/serialcomm)

---

## 📄 License

MIT License
