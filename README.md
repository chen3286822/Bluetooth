# Bluetooth · Android 蓝牙通信示例

一个使用 Java 编写的 Android 经典蓝牙通信示例，演示设备搜索、建立连接和双向文字聊天，保留了早期 Android 工程的实现方式。

项目使用 **RFCOMM**，不是 BLE / GATT 示例。适合阅读蓝牙设备发现、Socket 连接和数据收发的基本流程。

## 功能

- 请求开启蓝牙，列出已配对设备。
- 搜索附近的蓝牙设备，显示设备名称和地址。
- 一台设备启动服务端，另一台选择设备并发起连接。
- 连接后双向发送文字，显示消息和连接提示。
- 手动断开连接。

## 实现方式

设备页通过 `BluetoothAdapter` 搜索设备，通过广播接收搜索结果。聊天页使用：

- `listenUsingRfcommWithServiceRecord()`：创建服务端并等待连接。
- `createRfcommSocketToServiceRecord()`：建立客户端连接。
- `BluetoothSocket` 的输入、输出流：收发数据。
- 后台线程和 `Handler`：处理连接、读取数据及更新界面。

两端使用相同的服务 UUID：`00001101-0000-1000-8000-00805F9B34FB`。

## 工程结构

```text
src/gjz/bluetooth/
├── Bluetooth.java          # 入口与“设备列表 / 对话列表”切换
├── deviceActivity.java     # 设备搜索、选择及连接入口
├── chatActivity.java       # 服务端、客户端和消息收发
├── ChatListAdapter.java    # 设备列表适配器
├── deviceListAdapter.java  # 对话列表适配器
└── AnimationTabHost.java   # 标签页切换动画
res/                        # 布局、文字和图片
AndroidManifest.xml         # 应用入口、权限和最低 SDK 配置
project.properties          # 旧版 Android 工程配置
```

## 构建环境

这是旧版 Android 工程，不包含 Gradle 配置或 Gradle Wrapper，不能直接按现代 Android Studio 项目的方式构建。

仓库现有配置：

- 包名：`gjz.bluetooth`
- 最低 SDK：API 9，见 `AndroidManifest.xml`
- 工程构建目标：`android-19`，见 `project.properties`
- `default.properties` 仍保留更早的 `android-5` 配置，恢复工程时需要注意两者并不一致。

如使用旧版 Eclipse + ADT 环境，可尝试以现有 Android 源码创建或导入工程，指定本仓库目录，并检查构建目标为 API 19。仓库没有记录完整的 JDK、ADT 和构建工具版本，这不是已验证的环境复现步骤。

如需在现代 Android Studio 中继续开发，需要先迁移构建配置，再处理新版 Android 的权限和 API 兼容问题。

## 两台设备如何通信

以下流程根据现有界面和代码整理，需要在合适的实机环境中验证：

1. 在两台支持经典蓝牙的 Android 设备上安装应用，开启蓝牙。可以先在系统蓝牙设置中完成配对。
2. 在设备 A 的“设备列表”中点击“启动服务端”，进入对话页等待连接。
3. 在设备 B 的“设备列表”中选择设备 A；如果没有列出，先让 A 在系统设置中处于可被发现状态，再点击“开始搜索”。
4. 选中设备 A 后，在弹窗中点击“连接”。如系统要求配对，按系统提示确认。
5. 等待连接成功提示，再输入文字并点击“发送”。两端都可以发送消息。
6. 使用完毕点击“断开”；重新连接前先断开旧连接。

应用本身没有请求进入可被发现状态的界面。搜索不到设备时，应先检查对方的系统蓝牙设置。列表中的“没有设备已经配对”等提示不是可连接的设备。

## 使用限制

- 这是历史示例，尚未针对现代 Android 版本完成适配。
- 只演示单个对端的文字通信，不包含 BLE、文件传输、多人聊天或聊天记录持久化。
- 数据直接写入字节流，没有定义消息长度或分隔符；一次读取不一定对应一条完整消息。用于实际项目时，需要补充消息格式和拆分、拼接处理，并明确字符编码。
- 清单中除蓝牙权限外，还保留了 `READ_CONTACTS` 声明；当前 Java 源码未见读取联系人的逻辑。

本说明依据仓库现有源码整理，未重新进行构建或双机蓝牙测试。
