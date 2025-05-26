# TY-NVR 📷

TY-NVR 是一个使用 [Flutter](https://flutter.dev) 构建的轻量级 NVR 应用，专为 [Frigate](https://frigate.video) 实时视频分析系统打造。它支持多种视频流协议，提供本地与远程访问、事件查看、推送通知等功能。

> 🎯 目标平台：Android / iOS / Web（部分支持）  
> 🔒 自主部署，隐私友好，无第三方广告

---

## ✨ 功能特性 Features

- 🔌 **集成 Frigate**：通过 MQTT / REST API 与 Frigate 实时对接
- 🖥️ **支持视频播放**：支持 HLS / RTSP / MJPEG 等多种流格式
- 🔔 **移动侦测通知**：通过 Firebase 或 WebPush 实现事件提醒
- 🎥 **事件回放**：查看 Frigate 检测到的历史事件与录像
- 🧭 **自定义摄像头布局**：支持网格展示与单页切换
- 🛠️ **跨平台支持**：一套代码运行于 iOS / Android

---

## 📦 技术栈 Tech Stack

- Flutter + Dart
- [`video_player`](https://pub.dev/packages/video_player), [`flutter_vlc_player`](https://pub.dev/packages/flutter_vlc_player)
- [`mqtt_client`](https://pub.dev/packages/mqtt_client)
- [`dio`](https://pub.dev/packages/dio) + [`provider`](https://pub.dev/packages/provider)
- Frigate REST API / MQTT 消息集成
- 本地缓存与事件缩略图存储

---

## 🔧 配置 Configuration

在 `lib/config.dart` 或 `.env` 文件中配置你的 Frigate 服务器信息：

```dart
const frigateHost = "http://192.168.1.x:5000";
const mqttBroker = "mqtt://192.168.1.x:1883";
