<p align="center">
  <a href="https://apps.apple.com/us/app/ty-nvr/id6742030555" target="_blank">
    <img src="https://cdn.iphone-news.org/wp-content/uploads/2014/08/app_store_badge.png" alt="Download on the App Store" >
  </a>
  
  <a href="https://github.com/eisongao/TY-NVR/releases/download/7.5.2/app-release.apk" target="_blank">
    <img src="https://hf-files-oregon.s3.amazonaws.com/hdpconcord_kb_attachments/2020/09-18/9501d8c2-babe-4f54-8df8-92692dfbc657/google-play-badge.png" alt="Download on the Google Play Store" >
  </a>
</p>


###  APP 已改名Frigate NVR 

##下面是旧版资料了，只是参考

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

## 🔧 设置历史回放 / Setup Historical Playback

The default Frigate `/api/recordings` endpoint only returns recordings from the last hour. To retrieve older recordings, add a custom `/histories` endpoint.

### 调整 / Modify `frigate/api/media.py`

Add the following code to `frigate/api/media.py` to enable the `/histories` endpoint:

```python
from datetime import datetime, timedelta
from fastapi import APIRouter, Query
from fastapi.responses import JSONResponse
from frigate.models import Recordings

router = APIRouter()

@router.get("/{camera_name}/histories")
def histories(
    camera_name: str,
    date: str = Query(default=None, description="Date in YYYY-MM-DD format, overrides after/before"),
    after: float = Query(default=None, description="Start timestamp (Unix epoch)"),
    before: float = Query(default=None, description="End timestamp (Unix epoch)"),
    require_motion: bool = Query(default=False, description="Filter recordings with motion"),
    require_objects: bool = Query(default=False, description="Filter recordings with detected objects"),
):
    """Retrieve recordings for a camera within a specified time range (non-paginated)."""

    # Validate camera_name
    if not camera_name:
        return JSONResponse(status_code=400, content={"error": "Camera name is required"})

    # Date takes precedence over after/before
    if date:
        try:
            dt = datetime.strptime(date, "%Y-%m-%d")
            after = dt.timestamp()
            before = (dt + timedelta(days=1)).timestamp()
        except ValueError:
            return JSONResponse(status_code=400, content={"error": "Invalid date format, use YYYY-MM-DD"})

    # Default to past 1 hour if no time range provided
    if after is None or before is None:
        now = datetime.now()
        after = (now - timedelta(hours=1)).timestamp() if after is None else after
        before = now.timestamp() if before is None else before

    # Validate timestamps
    if after >= before:
        return JSONResponse(status_code=400, content={"error": "After timestamp must be earlier than before timestamp"})

    # Build query conditions
    where_clause = [
        Recordings.camera == camera_name,
        Recordings.end_time >= after,
        Recordings.start_time <= before,
    ]
    if require_motion:
        where_clause.append(Recordings.motion > 0)
    if require_objects:
        where_clause.append(Recordings.objects > 0)

    try:
        recordings = (
            Recordings.select(
                Recordings.id,
                Recordings.start_time,
                Recordings.end_time,
                Recordings.segment_size,
                Recordings.motion,
                Recordings.objects,
                Recordings.duration,
                Recordings.path,
            )
            .where(*where_clause)
            .order_by(Recordings.start_time)
            .dicts()
        )
        return JSONResponse(content=list(recordings))
    except Exception as e:
        return JSONResponse(status_code=500, content={"error": f"Failed to retrieve recordings: {str(e)}"})
```

### 应用更改 / Apply Changes
1. Open `frigate/api/media.py` in your Frigate installation.
2. Add or replace the `/histories` endpoint with the code above.
3. Restart the Frigate service:
   ```bash
   docker restart <your_frigate_container_name>
   ```

### 测试端点 / Test the Endpoint
```bash
curl "http://<frigate_host>:5000/api/<camera_name>/histories?date=2025-06-05"
```

---

## 🔧 Firebase 云消息传递 (FCM) 设置 / Firebase Cloud Messaging (FCM) Setup

To enable push notifications for motion detection, integrate Firebase Cloud Messaging (FCM) with both the TY-NVR Flutter app and the Frigate backend.

### 1. 创建 Firebase 项目 / Create a Firebase Project
1. Go to [Firebase Console](https://console.firebase.google.com/).
2. Click **Add Project**, name it `TY-NVR`, and complete the setup.
3. Enable **Cloud Messaging** (enabled by default).
4. Add an Android/iOS app with the package name `com.app.frigateNvr`.

### 2. 下载服务帐户密钥 / Download Service Account Key
1. In Firebase Console, navigate to **Project Settings** → **Service Accounts**.
2. Click **Generate new private key** and download the `.json` file (e.g., `ty-nvr-fcm-service-account.json`).

### 3. 在 Frigate 中安装 `firebase-admin` /  Install `firebase-admin` in Frigate
To persist the `firebase-admin` installation, modify the Frigate Docker image:

#### 更新 `requirements-wheels.txt` / Update `requirements-wheels.txt`
1. Locate `requirements-wheels.txt` in your Frigate directory.
2. Add:
   ```
   firebase-admin==6.5.0
   ```
   > **Note**: Check for the latest version of `firebase-admin` if `6.5.0` is outdated:
   > ```bash
   > pip install firebase-admin --upgrade
   > ```

3. Rebuild the Docker image:
   ```bash
   cd /path/to/frigate
   docker build -t frigate-custom .
   ```

4. Update `docker-compose.yml`:
   ```yaml
   services:
     frigate:
       image: frigate-custom
       ...
   ```

5. Restart Frigate:
   ```bash
   docker-compose up -d
   ```

### 4. 将 FCM 凭证存储在数据库中 / Store FCM Credentials in Database
Store the Firebase service account key in the Frigate database for persistent access.

#### 安装 SQLite / Install SQLite
```bash
# Debian/Ubuntu
sudo apt-get update
sudo apt-get install -y sqlite3

# CentOS/RHEL
sudo yum install -y sqlite
```

#### 创建 `fcmconfig` 表 / Create `fcmconfig` Table
1. Back up the Frigate database:
   ```bash
   cp /home/frigate/config/frigate.db /home/frigate/config/frigate.db.bak
   ```

2. Open SQLite:
   ```bash
   sqlite3 /home/frigate/config/frigate.db
   ```

3. Create the table:
   ```sql
   CREATE TABLE IF NOT EXISTS fcmconfig (
       id INTEGER PRIMARY KEY AUTOINCREMENT,
       project_id VARCHAR(128) NOT NULL UNIQUE,
       project_name VARCHAR(128) NOT NULL UNIQUE DEFAULT 'tynvr-fcm',
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

4. Insert service account data (replace placeholders with values from `ty-nvr-fcm-service-account.json`):
   ```sql
   INSERT INTO fcmconfig (
       project_id,
       project_name,
       private_key_id,
       private_key,
       client_email,
       client_id,
       auth_uri,
       token_uri,
       auth_provider_x509_cert_url,
       client_x509_cert_url,
       created_at,
       updated_at
   ) VALUES (
       'your-project-id',
       'tynvr-fcm',
       'your-private-key-id',
       '-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n',
       'your-client-email@your-project.iam.gserviceaccount.com',
       'your-client-id',
       'https://accounts.google.com/o/oauth2/auth',
       'https://oauth2.googleapis.com/token',
       'https://www.googleapis.com/oauth2/v1/certs',
       'https://your-client-x509-cert-url',
       '2025-06-06 14:03:00',
       '2025-06-06 14:03:00'
   );
   ```

5. Verify and exit:
   ```sql
   .tables
   SELECT * FROM fcmconfig;
   .exit
   ```

### 5. Add FCM Model to Frigate
Modify `frigate/models.py` to include the `FCMConfig` model.

```python
from peewee import Model, IntegerField, CharField, TextField, DateTimeField
import datetime

class FCMConfig(Model):
    id = IntegerField(primary_key=True)
    project_id = CharField(max_length=128, unique=True, null=False)
    project_name = CharField(max_length=128, unique=True, null=False, default='tynvr-fcm')
    private_key_id = CharField(max_length=256, null=False)
    private_key = TextField(null=False)
    client_email = CharField(max_length=256, null=False)
    client_id = CharField(max_length=128, null=False)
    auth_uri = CharField(max_length=256, null=False)
    token_uri = CharField(max_length=256, null=False)
    auth_provider_x509_cert_url = CharField(max_length=256, null=False)
    client_x509_cert_url = CharField(max_length=256, null=False)
    created_at = DateTimeField(default=datetime.datetime.now, null=False)
    updated_at = DateTimeField(default=datetime.datetime.now, null=False)
```

### 应用更改 / Apply Changes
1. Open `frigate/comms/webpush.py`.
2. Replace the Firebase initialization logic with the code above.
3. Ensure the `CONFIG_DIR` path matches your Frigate configuration directory (e.g., `/config`).
4. Restart Frigate:
   ```bash
   docker restart <your_frigate_container_name>
   ```

```bash
service_account_path = os.path.join(CONFIG_DIR, "serviceAccountKey.json")
        logger.debug(f"Checking service account path: {service_account_path}")
        if not os.path.exists(service_account_path):
            logger.error(f"Service account key file not found at {service_account_path}")
        else:
            try:
                try:
                    firebase_admin.get_app()
                    logger.debug("Firebase Admin SDK already initialized")
                except ValueError:
                    cred = credentials.Certificate(service_account_path)
                    firebase_admin.initialize_app(cred)
                    logger.info("Firebase Admin SDK initialized successfully")
            except Exception as e:
                logger.error(f"Failed to initialize Firebase Admin SDK: {str(e)}")
```

```bash
# Initialize Firebase Admin SDK
        try:
            # 这里用 project_name="tynvr-fcm"
            fcm_config = FCMConfig.get(FCMConfig.project_name == "tynvr-fcm")
        except DoesNotExist:
            logger.error("没有在数据库中找到 FCMConfig，无法初始化 Firebase Admin SDK")
        else:
            # 把 Peewee model 里的各字段拼成一个 dict
            service_account_dict = {
                "type": "service_account",  # 如果你表里没存这个字段，可以硬编码
                "project_id": fcm_config.project_id,
                "private_key_id": fcm_config.private_key_id,
                "private_key": fcm_config.private_key,
                "client_email": fcm_config.client_email,
                "client_id": fcm_config.client_id,
                "auth_uri": fcm_config.auth_uri,
                "token_uri": fcm_config.token_uri,
                "auth_provider_x509_cert_url": fcm_config.auth_provider_x509_cert_url,
                "client_x509_cert_url": fcm_config.client_x509_cert_url,
                # 如果你表里还存了 "universe_domain" 之类，也可以一并加上
            }
            try:
                # 如果已经初始化过，就不会再初始化
                firebase_admin.get_app()
                logger.debug("Firebase Admin SDK already initialized")
            except ValueError:
                # 直接把 dict 传给 credentials.Certificate
                try:
                    cred = credentials.Certificate(service_account_dict)
                    firebase_admin.initialize_app(cred)
                    logger.info("Firebase Admin SDK initialized successfully (从数据库读取配置)")
                except Exception as e:
                    logger.error(f"Firebase Admin SDK 初始化失败：{e}")
```
---

## 📸 示例截图 / Example Screenshots

Below are screenshots showcasing TY-NVR’s interface:

- **Home Screen**:  
  ![TY-NVR Home Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/home.png?raw=true)
- **Live View**:  
  ![TY-NVR Live Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/live.png?raw=true)
- **Playback**:  
  ![TY-NVR Playback Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/playback.png?raw=true)
- **Multi-Camera Display**:  
  ![TY-NVR Multi Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/Multi-screen%20display.png?raw=true)
- **Events**:  
  ![TY-NVR Events Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/events.png?raw=true)
- **Files**:  
  ![TY-NVR Files Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/files.png?raw=true)
- **Settings**:  
  ![TY-NVR Settings Screen](https://github.com/eisongao/TY-NVR/blob/main/screenshot/setting.png?raw=true)

---

## 🚀 后续步骤 / Next Steps

1. **Test FCM Notifications**:
   - Trigger a motion event in Frigate and verify that a push notification is received on the TY-NVR app.
2. **Verify Historical Playback**:
   - Use the `/histories` endpoint to fetch recordings older than 1 hour and ensure they display in the TY-NVR app.
3. **Monitor Logs**:
   - Check Frigate logs for any Firebase initialization errors:
     ```bash
     docker logs <your_frigate_container_name>
     ```

For further assistance, refer to the [Frigate documentation](https://docs.frigate.video) or [Flutter documentation](https://flutter.dev/docs).
