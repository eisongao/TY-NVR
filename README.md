# 📷 TY-NVR

**TY-NVR** 是一个使用 [Flutter](https://flutter.dev) 构建的轻量级 NVR 应用，专为 [Frigate](https://frigate.video) 实时视频分析系统打造。  
支持多种视频流协议，提供本地与远程访问、事件查看、推送通知等功能。

> 🎯 **目标平台**：Android / iOS / Web（部分支持）  
> 🔒 **自主部署**，隐私友好，无第三方广告

---

## ✨ 功能特性 / Features

- 🔌 **集成 Frigate**：通过 MQTT / REST API 与 Frigate 实时对接  
- 🖥️ **支持视频播放**：支持 HLS / RTSP / MJPEG 等多种流格式  
- 🔔 **移动侦测通知**：通过 Firebase 或 WebPush 实现事件提醒  
- 🎥 **事件回放**：查看 Frigate 检测到的历史事件与录像  
- 🧭 **自定义摄像头布局**：支持网格展示与单页切换  
- 🛠️ **跨平台支持**：一套代码运行于 iOS / Android

---

## 📦 技术栈 / Tech Stack

- `Flutter` + `Dart`  
- 视频播放：[`video_player`](https://pub.dev/packages/video_player)、[`flutter_vlc_player`](https://pub.dev/packages/flutter_vlc_player)  
- 通信：[`mqtt_client`](https://pub.dev/packages/mqtt_client)、[`dio`](https://pub.dev/packages/dio)  
- 状态管理：[`provider`](https://pub.dev/packages/provider)  
- 与 Frigate 交互：REST API / MQTT 消息集成  
- 本地缓存、事件缩略图存储支持

---

## 🔧 FCM 配置 / Firebase Cloud Messaging (FCM) Configuration

为启用移动侦测推送，请将 Firebase Cloud Messaging 集成进 Flutter 应用和 Frigate 服务端。

### 1️⃣ 创建 Firebase 项目

1. 登录 [Firebase 控制台](https://console.firebase.google.com/)  
2. 点击「添加项目」，命名为 `TY-NVR`，按向导完成  
3. 启用 Cloud Messaging（默认启用）  
4. 在“项目设置”中，添加 Android / iOS 应用  
   > 应用包名为：`com.app.frigateNvr`

---

### 2️⃣ 下载服务账号密钥并导入 Frigate

1. 打开 Firebase 控制台 → `项目设置` → `服务账号`  
2. 点击 **生成新的私钥**，下载 `.json` 文件（例如：`ty-nvr-fcm-service-account.json`）  
3. 在 Frigate 服务器中配置：

#### 安装 sqlite3（如尚未安装）：

```bash
# Debian / Ubuntu
sudo apt-get update
sudo apt-get install -y sqlite3

# CentOS / RHEL
sudo yum install -y sqlite
```

4. 复制并备份现有的 `frigate.db`（根据自己存储位置替换）：  
   ```bash
   cp /home/frigate/config/frigate.db /home/frigate/config/frigate.db.bak
   sqlite3 /home/frigate/config/frigate.db
   ```
   
5.在 sqlite3 交互终端中，创建 fcmconfig 表：
在终端输入 
```bash
sqlite3 /home/frigate/config/frigate.db
sqlite>
```
```bash
CREATE TABLE IF NOT EXISTS fcmconfig (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  project_id VARCHAR(128) NOT NULL UNIQUE,
  private_key_id VARCHAR(256) NOT NULL,
  private_key TEXT NOT NULL,
  client_email VARCHAR(256) NOT NULL,
  client_id VARCHAR(128) NOT NULL,
  auth_uri VARCHAR(256) NOT NULL,
  token_uri VARCHAR(256) NOT NULL,
  auth_provider_x509_cert_url VARCHAR(256) NOT NULL,
  client_x509_cert_url VARCHAR(256) NOT NULL,
  created_at DATETIME NOT NULL,
  updated_at DATETIME NOT NULL
);
```
检测是否成功
```bash
sqlite> .tables
```
退出
Ctrl+D

## 应用首页示例截图 / Example Home Screen

以下效果示例

![TY-NVR Home Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/home.png?raw=true)
![TY-NVR Live Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/live.png?raw=true)
![TY-NVR Playback Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/playback.png?raw=true)
![TY-NVR Multi Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/Multi-screen%20display.png?raw=true)
![TY-NVR Events Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/events.png?raw=true)
![TY-NVR Files Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/files.png?raw=true)
![TY-NVR Settings Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/setting.png?raw=true)
